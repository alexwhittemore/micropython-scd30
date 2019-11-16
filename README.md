# Sensirion SCD30 CO² Sensor I2C driver for ~~MicroPython~~ CircuitPython

## NOTE: This version of the library is currently CircuitPython compatible, not yet both-compatible.

It shouldn't be too hard to make this library platform-agnostic, as the changes to make it CircuitPython-compatible were pretty minimal. I haven't tried using Blinka, which might have been the "right" way to deploy this in my project anyway. I'm not sure how Blinka's supposed to work in situations like this. Maybe it's relevant. 

Sensirion SCD30 is a CO², Humidity and Temperature sensor on a module. This is
a I2C driver written in Python 3 for MicroPython.

## Getting Started

### Prerequisites

* Sensirion SCD30 Sensor Module
* MicroPython board with I2C interface

### Wiring

Wire the I2C bus to the I2C bus on your MicroPython board. This is an example
using the Pyboard D:

| Pyboard       | SCD30         |
| ------------- |---------------|
| X15 (3V3)     | VDD           |
| X14 (GND)     | GND           |
| X9            | TX/SCL        |
| X10           | RX/SDA        |

### Usage

This example reads the measurements in a continous loop:

``` python
import time
from machine import I2C, Pin
from scd30 import SCD30

i2cbus = I2C(1)
scd30 = SCD30(i2c, 0x61)

while True:
    # Wait for sensor data to be ready to read (by default every 2 seconds)
    while scd30.get_status_ready() != 1:
        time.sleep_ms(200)
    scd30.read_measurement()
```

CircuitPython Example:

``` python
import time
import board
from scd30 import SCD30

i2c = board.I2C()
scd30 = SCD30(i2c, 0x61)

while 1:
    while scd30.get_status_ready() != 1:
            time.sleep(0.200)
    print("getting measurement")
    try:
        co2, temp, relh = scd30.read_measurement()
    except CRCException:
        pass
    print(co2)
    print(temp)
    print(relh)
```

Note that the CO² sensor needs some time to stabilize. Therefor the sensor
should be kept powered to achieve a reasonable measurement interval (e.g. <5
minutes). To save power the sensors measurement inverval can be tweaked. See
also the [Low Power Mode for SCD30](https://docs-emea.rs-online.com/webdocs/16c9/0900766b816c9dc7.pdf)
application note.

### Calibration

The CO² sensor has two modes of calibration: FRC (Forced Recalibration) or ASC
(Automatic Self-Calibration). This only describes the former.

Essentially the sensor is already calibrated at factory. However, when setting a
new measurement interval recalibration might be necessary. The process is to
bring the sensor into a controlled environment (e.g. outside) and set the known
value at that environment (e.g. 400ppm). From what I understand ASC does
essentially the same, just assumes that the lowest values over a certain periode
are "outside values"...

Also note that the temperature sensor suffers from heating effects on the PCB.
When the sensor operates in 2 second interval the heating is about 3°C. I
usually run the sensor at 30 seconds interval and observed a heating of 2°C. The
offset is subtracted from the measured temperature! To set a new offset, take
the old offset into account!

## Built With

* [MicroPython](http://micropython.org/)
* [SCD30 Sensor Module](https://www.sensirion.com/en/environmental-sensors/carbon-dioxide-sensors-co2/)

## License

This project is licensed under the MIT License - see the
[LICENSE](LICENSE) file for details

