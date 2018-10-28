alarm
==================

fona 808 - [schematic](https://cdn-learn.adafruit.com/assets/assets/000/026/594/original/adafruit_products_schem.png?1437161178)
teensy 3.2 - [pinout](https://www.pjrc.com/teensy/teensy32_front_pinout.png)
MMA8451 - [datasheet](https://cdn-shop.adafruit.com/datasheets/MMA8451Q-1.pdf) [adafruit](https://learn.adafruit.com/adafruit-mma8451-accelerometer-breakout/pinouts)

### mcu
I intially bought an adafruit m0 express for this project but `rtczero` in its deepest sleep setup only goes down to 5mA, which wouldn't cover the battery needed. I went for a teensy instead, which goes way below 1mA thanks to [duff2013 great Snooze library](https://github.com/duff2013/Snooze). The whole board drains under 500uA in idle mode.

the code is in `main.cpp` run simply with `platformio run --target upload`

### accelerometer
I hacked up the original Adafruit MMA8451 lib to provide interrupt capacity ([code here](lib/Adafruit MMA8451 Interrupt)). Can't put it in low power, as it's not precise enough, but it doesn't drain much current (160uA iirc from the docs)

### setup
So it's pretty much based around the adafruit fona808 arduino shield. I built a custom "motherboard" to that shield off a teensy 3.2, which is powered directly on the lipo battery.
The motherboard also hosts an adafruit MMA8451 breakout board. To save the battery, the power is toggled on and off with a relay driven by the teensy and a couple of bs170 mos. I chose not to use the poweroff instructions from adafruit for power consumtion reasons.

### server code
emits a GET request to `URL_TO_FETCH` with the following parameter:
```
/e?report=0-1-2-3 (0 daily ping, 1 movement, 2 emergency, 3 emergency gsm)
     &bat=battery_percentage
     &sig=rssi_gsm
     &pos=lat,long,date
     &gps=
```

the server controls the mode of the device via the http status code. `200` leaves the board in normal reporting, `418` makes the board report every 25 secs, and `421` makes the board report every 25 secs with GSM stats. A boilerplate code for the server is in `server.go`.