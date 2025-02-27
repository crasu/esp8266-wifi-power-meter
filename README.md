# ESP8266 Wifi Power Meter (Ferraris)

## Description

<p align="center">
<img src="https://github.com/lrswss/esp8266-wifi-power-meter/blob/master/ferraris_meter.png" alt="Wemos D1 mini with TCRT5000 IR sensor mounted on ferraris electricity meter">

<img src="https://github.com/lrswss/esp8266-wifi-power-meter/blob/master/webui.png" alt="UI of Embedded Webserver">
</p>

Impulses generated by an infrared sensor detecting the revolutions of a ferraris
disk based electricity meter are published with MQTT and are also displayed on a simple
embedded web page. As visual feedback the blue LED on the Wemos D1 mini board will
blink if the red marker has been detected. Meter values are frequently saved
to EEPROM to migate counter resets due to possible intermediate power failures.

## Shopping list

* [Wemos D1 mini](https://arduino-projekte.info/wemos-d1-mini/) development board
* [TCRT5000](https://www.google.com/search?q=TCRT5000) IR sensor board
* 5V (250mA) Power supply with micro USB plug

## Using the TCRT5000's analog output (A0) instead of D0

Since the low/high impulses generated by the widely used TCRT5000 sensor on D0 are
not very reliable (e.g. multiple counts for single pass of the "red marker" on
the ferraris disk) its analog output (A0) is used to automatically determine a
suitable threshold value to detect the marker. This way you don't need to fiddle
with the small potentiometer on the sensor board to find the "right" trigger
threshold for your specific sensor mounting setup.

## Compiling the firmware for the Wemos D1 mini

Set the `WIFI_` and `MQTT_` defines at the beginning of the sketch. The value for
`TURNS_PER_KWH` must match the value printed somewhere on your electricity meter
Uncomment `LANGUAGE_EN` for an english web UI and serial debug messages.

[Add support for ESP8266 based boards](https://github.com/esp8266/Arduino) to
your Arduino IDE using the boards manager.

Before trying to compile and flashing the sketch to the `Wemos D1 R1` board make
sure that the following libraries have been installed using the
[Arduino IDE library manager](https://www.arduino.cc/en/Guide/Libraries):

* [EPS8266Wifi](https://github.com/esp8266/Arduino/tree/master/libraries/ESP8266WiFi)
* [ESP8266WebServer](https://github.com/esp8266/Arduino/tree/master/libraries/ESP8266WebServer)
* [ESP8266HTTPUpdateServer](https://github.com/esp8266/Arduino/tree/master/libraries/ESP8266HTTPUpdateServer)
* [ArduinoJson](https://arduinojson.org/)
* [PubSubClient](https://github.com/knolleary/pubsubclient/releases)
* [EEPROM_Rotate](https://github.com/xoseperez/eeprom_rotate)

## Hardware Schematics

<p align="center">
<img src="https://github.com/lrswss/esp8266-wifi-power-meter/blob/master/schematics.png" alt="Schematics für Wifi Power Meter">
</p>

Connect the 5V pin of the TCRT5000 sensor to 3.3V pin (not 5V!) of the Wemos
board and its ground pin to GND. The analog output of the TCRT5000 sensor (A0)
must be connected to the corresponding analog input (A0) of the Wemos. The Wemos
can be powered with a small USB power supply (5V, around 250mA).

## Initial operation

After powering up the Wemos D1 mini it should connect to your Wifi network and
receive an IP via DHCP (check your router or the serial debugging messages).
Point your favourite browser to the IP. If you see the embebbed web page you
can proceed mounting the sensor and the Wemos on your ferraris meter.

## Mounting the sensor on the ferraris meter

For mounting the TCRT5000 on your ferraris meter a little 3D printed case
comes in handy. Thingiverse offers quite a few options like
[this one](https://www.thingiverse.com/thing:2668168)

Since this sketch also supports OTA-Updates of the firmware, you only need to flash
your ESP8266 once before mounting in your meter cabinet.

## Setting the threshold to detect the red marker

Make sure that during the initial calibration phase of `READINGS_TOTAL_SEC`
(default 90 secs.) triggered by `Calculate Threshold` the ferraris disk is actually
spinning fast enough. The calculation of the threshold should be based on at least
two full revolutions with the red marker passing the IR sensor twice. Turning on
your oven should help to speed up things. ;-)

After the initial and hopefully successful calibration cycle you only have to
save the calculated threshold value with `Save Threshold` to switch the Wifi
power meter to normal operation. You can later tune the threshold value with
`http://<IP>/setThreshold?value=XX`.

The initial offset to your current electricity meter reading needs to be set once
with the following command `http://<IP>/setCounter?value=XXXXX.X` after setting the
threshold value. Then you're ready to go.

## RESTful support

The Wifi power meter offers support for RESTful HTTP requests. Current readings are
available as JSON under `http://<IP>/readings`. See `restful.html` as an example.

## Debug readings with InfluxDB and Grafana

For debugging purposes you can continuously send the analog readings (every `READINGS_INTERVAL_MS`)
to an [InfluxDB](https://www.influxdata.com/products/influxdb-overview/) with UDP.
Contiously visualizing the readings with [Grafana](https://grafana.com/oss/grafana/)
might help placing the TCRT5000 on your electricity meter to get better IR readings.

## Contributing

Pull requests are welcome! For major changes, please open an issue first to discuss
what you would like to change.

## License

Copyright (c) 2019-2021 Lars Wessels
This software was published under the MIT license.
Please check the [license file](LICENSE).
