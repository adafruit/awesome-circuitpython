# CircuitPython Cheatsheet #

## Digital I/O ##

    import board
    from digitalio import DigitalInOut, Direction, Pull

    led = DigitalInOut(board.D13)
    led.direction = Direction.OUTPUT

    switch = DigitalInOut(board.D5)
    switch.direction = Direction.INPUT
    switch.pull = Pull.UP   # Pull.Down is available on some MCUs

    while True:
        led.value = not switch.value
        time.sleep(0.01)

## Analog Input ##

    import time
    import board
    from analogio import AnalogIn

    analog_in = AnalogIn(board.A1)

    def get_voltage(pin):
        return (pin.value * 3.3) / 65536

    while True:
        print((get_voltage(analog_in),))
        time.sleep(0.1)

Analog input values are always 16 bit (i.e. in range(0, 65535)), regardless of the converter's resolution. The get_voltage function converts the analog reading into a voltage, assuming the default 3.3v reference voltage.

## Analog Output ##

    import board
    from analogio import AnalogOut

    analog_out = AnalogOut(board.A0)

    while True:
        # Count up from 0 to 65535
        for i in range(0, 65536):
            analog_out.value = i

Analog output values are always 16 bit (i.e. in range(0, 65535)). Depending on the underlying hardware those values will get scaled to match the resolution of the converter.
The example will generate a stairstepped signal, the number of steps depends on the resolution of the converter. E.g. the 10-bit converter in the SAMD21 will create 1024 steps, while the 12-bit converter on the SAMD51 will create 4096 steps.

## PWM ##

You can use a PWM in one of two ways.

1) With fixed frequency PWM with variable duty cycle. This is useful for controllign the brightness of a LED or the speed of a motor.

    import time
    import board
    import pulseio

    led = pulseio.PWMOut(board.D13, frequency=5000, duty_cycle=0)

    while True:
        for i in range(100):
            # PWM LED up and down
            if i < 50:
                led.duty_cycle = int(i * 2 * 65535 / 100)  # Up
            else:
                led.duty_cycle = 65535 - int((i - 50) * 2 * 65535 / 100)  # Down
            time.sleep(0.01)

2) With variable frequency as well. This is handy for producing tones. The duty cycle effects the sound (as opposed to the note).

    import time
    import board
    import pulseio

    piezo = pulseio.PWMOut(board.A1, duty_cycle=0, frequency=440, variable_frequency=True)

    while True:
        for f in (262, 294, 330, 349, 392, 440, 494, 523):
            piezo.frequency = f
            piezo.duty_cycle = 65536 // 2  # On 50%
            time.sleep(0.25)  # On for 1/4 second
            piezo.duty_cycle = 0  # Off
            time.sleep(0.05)  # Pause between notes
        time.sleep(0.5)

## Servo ##

    import time
    import board
    import pulseio
    from adafruit_motor import servo

    # create a PWMOut object on Pin A2.
    pwm = pulseio.PWMOut(board.A2, duty_cycle=2 ** 15, frequency=50)

    # Create a servo object, my_servo.
    my_servo = servo.Servo(pwm)

    while True:
        for angle in range(0, 180, 5):  # 0 - 180 degrees, 5 degrees at a time.
            my_servo.angle = angle
            time.sleep(0.05)
        for angle in range(180, 0, -5): # 180 - 0 degrees, 5 degrees at a time.
            my_servo.angle = angle
            time.sleep(0.05)


## Cap Touch ##

    import time

    import board
    import touchio

    touch_pad = board.A1  # For Circuit Playground Express

    touch = touchio.TouchIn(touch_pad)

    while True:
        if touch.value:
            print("Touched!")
        time.sleep(0.05)


## NeoPixels ##

    import time
    import board
    import neopixel

    RED = (255, 0, 0)
    GREEN = (0, 255, 0)
    BLUE = (0, 0, 255)

    pixel_pin = board.A1
    num_pixels = 8

    pixels = neopixel.NeoPixel(pixel_pin, num_pixels, brightness=0.3, auto_write=False)

    pixels.fill(RED)
    pixels.show()

    #The usual slicing operations can be used
    pixels[1:6:2] = GREEN
    pixels[7] = BLUE
    pixels.show()


## DotStar ##

    import time
    import adafruit_dotstar
    import board

    RED = (255, 0, 0)
    num_pixels = 30

    # DotStars use 2 pins instead of 1 that NeoPixels take
    pixels = adafruit_dotstar.DotStar(board.A1, board.A2, num_pixels, brightness=0.1, auto_write=False)
    pixels.fill(0) # all off
    pixels[::2] = [RED] * (num_pixels // 2) # every other pixel red
    pixels.show()

## UART Serial ##

    import board
    import busio
    import digitalio

    uart = busio.UART(board.TX, board.RX, baudrate=9600)

    while True:
        data = uart.read(32)  # read up to 32 bytes

        if data is not None:
            # convert bytearray to string
            data_string = ''.join([chr(b) for b in data])
            print(data_string, end="")
            uart.write(data_string)

## I2C ##

    import time
    import adafruit_tsl2561
    import board
    import busio

    i2c = busio.I2C(board.SCL, board.SDA)

    # Create library object on our I2C port
    tsl2561 = adafruit_tsl2561.TSL2561(i2c)

    # Use the object to print the sensor readings
    while True:
        print("Lux:", tsl2561.lux)
        time.sleep(1.0)

## SPI ##

    import board
    import busio
    import digitalio
    import adafruit_bme280

    spi = busio.SPI(board.SCK, MOSI=board.MOSI, MISO=board.MISO)
    cs = digitalio.DigitalInOut(board.D5)
    bme280 = adafruit_bme280.Adafruit_BME280_SPI(spi, cs)
    print("\nTemperature: %0.1f C" % bme280.temperature)
    print("Humidity: %0.1f %%" % bme280.humidity)
    print("Pressure: %0.1f hPa" % bme280.pressure)


## HID Keyboard ##

    from adafruit_circuitplayground.express import cpx
    from adafruit_hid.keyboard import Keyboard
    from adafruit_hid.keycode import Keycode

    kbd = Keyboard()

    while True:
        if cpx.button_a:
            kbd.send(Keycode.SHIFT, Keycode.A)  # Type capital 'A'
            while cpx.button_a: # Wait for button to be released
                pass

        if cpx.button_b:
            kbd.send(Keycode.CONTROL, Keycode.X)  # control-X key
            while cpx.button_b: # Wait for button to be released
                pass

## HID Mouse ##

    from adafruit_circuitplayground.express import cpx
    from adafruit_hid.mouse import Mouse

    m = Mouse()
    cpx.adjust_touch_threshold(200)

    while True:
        if cpx.touch_A4:
            m.move(-1, 0, 0)
        if cpx.touch_A3:
            m.move(1, 0, 0)
        if cpx.touch_A7:
            m.move(0, -1, 0)
        if cpx.touch_A1:
            m.move(0, 1, 0)

        if cpx.button_a:
            m.press(Mouse.LEFT_BUTTON)
            while cpx.button_a:    # Wait for button A to be released
                pass
            m.release(Mouse.LEFT_BUTTON)

        if cpx.button_b:
            m.press(Mouse.RIGHT_BUTTON)
            while cpx.button_b:    # Wait for button B to be released
                pass
            m.release(Mouse.RIGHT_BUTTON)
