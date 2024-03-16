# Read out electricity meter Iskra MT681 (Stromzähler)

## Motivation

After installing 2 photovoltaic modules on my balcony (check out the project documentation at <https://entorb.net/wickie/Balkonkraftwerk>), my electricity provider unfortunately replaced my old electricity meter (that rotated backwards when PV production exceeded my consumption).

As the new meter has an optic interface, I decided to try to read it out to have at least some benefit of the replacement...

A great documentation (in German) on how to read out different electricity meters <https://ottelo.jimdofree.com/stromz%C3%A4hler-auslesen-tasmota/> was my starting point.

## Hardware

My electricity meter is an Iskra eHZ-MT681-D4A52-K0p

For the readout I bought an all-inclusive product <https://www.wispr-shop.de/produkt/wifi-ir-schreib-lesekopf-diy-set/> for 40€ at eBay. It bundles

* Hichi IR sensor
* micro controller ESP01S (Wi-Fi and MicroUSB power connector)
* pre-installed software Tasmota V13.3

so no Arduino programming and no soldering ;-)

## Setup

Following the manual of the sensor I performed these steps

* connect laptop to sensor's Wi-Fi
* enter my Wi-Fi credentials
* switch back to my Wi-Fi
* access sensor webinterface
* setup the electricity meter via console -> script (see below)
* activating the script
* -> data appears on the main page of the UI
* set Logging->TelePeriod to 15 for readout every 15s
* (optionally) configure access to my MQTT server

### Tasmota script for Iskra MT681-D4A52-K0p

* copied from <https://tasmota.github.io/docs/Smart-Meter-Interface/#iskra-ehz-mt681-d4a52-k0p>
* skipping/commenting out the last line of the static Meter_id

```text
>D
>B
=>sensor53 r
>M 1
+1,3,s,0,9600,MT681
1,77070100010800ff@1000,Verbrauch,kWh,Total_in,4
1,77070100100700ff@1,Leistung,W,Power_cur,0
1,77070100020800ff@1000,Erzeugung,kWh,Total_out,4
; 1,77070100000009ff@#,Service ID,,Meter_id,0|
```

## Data visualization

### In device webinterface

Here is described, how to upload a Tasmota firmware and script, that provides data visualization on the webinterface of the device <https://ottelo.jimdofree.com/stromz%C3%A4hler-auslesen-tasmota/>. I did not try that.

### In Grafana

Instead, I use some existing pieces of software on a Raspberry Pi 3b "server"

* activate MQTT protocol in the sensor webinterface to send the data
* Python script to receive the data via MQTT
* InfluxDB to store the data
* Grafana to visualize the data

### In HomeAssistant

Alternative setup could be to use HomeAssistant to receive the data via MQTT, with is also described in the documentation <https://ottelo.jimdofree.com/stromz%C3%A4hler-auslesen-tasmota/>.

## Power Saving

After setting up the device and connecting to MQTT broker, I decided to turn off the webinterface, to hopefully reduce the power consumption. (here tasmota_MT681 is the name of my device)

```sh
mosquitto_pub -u mqtt_user -P mqtt_passwd -t "cmnd/tasmota_MT681/webserver" -m "0"
```

reduce sending frequency to 300s (currently I use 15s)

```sh
mosquitto_pub -u mqtt_user -P mqtt_passwd -t "cmnd/tasmota_MT681/TelePeriod" -m "300"
```

## Tasmota commands per MQTT

see <https://tasmota.github.io/docs/Commands/>

sending an empty message body, results in the device responding with its current value

```sh
mosquitto_pub -u mqtt_user -P mqtt_passwd -t "cmnd/tasmota_MT681/webserver" -m ""
```
