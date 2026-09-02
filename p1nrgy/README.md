# Home Assistant Add-on: p1nrgy

p1nrgy is a Home Assistant add-on that reads energy consumption data from DSMR-compatible smart meters and publishes the data to an MQTT broker. It's aim is to be blazing fast and lightweight, ideal for running on resource-constrained devices like Raspberry Pi.

p1nrgy is written in Go, so it compiles to a single binary and not using an interpreter, making it very/more efficient in terms of CPU and memory usage.
The idea is also to keep it as simple as it gets, reader the DSMR data, parsing it, and publishing it to MQTT. It's then up to the user in Home Assistant to consume the MQTT data.

## Supported Architectures

![Supports aarch64 Architecture][aarch64-shield]
![Supports amd64 Architecture][amd64-shield]

## Installation

In order to install this add-on, you need to add this repository to your Home Assistant instance. You can do this by clicking the badge below:

[![Open your Home Assistant instance and show the add add-on repository dialog with a specific repository URL pre-filled.](https://my.home-assistant.io/badges/supervisor_add_addon_repository.svg)](https://my.home-assistant.io/redirect/supervisor_add_addon_repository/?repository_url=https%3A%2F%2Fgithub.com%2Fhans-2022%2Fp1ngry-addon)

I was looking for a lightweight appraoch to capture P1 smart meter telegrams. Liked p1nrgy. This fork was made for following reasons: 

I wanted to link to my EMQX implementation and from there work with the telegrams

and while doing that I noticed that the "erwindouna" P1ngry logged each datagram. So with chatgpt help I built in a switch to toggle that through the configuration. When updating it will ask for a restart

## Configuration

By default, p1nrgy provides a basic configuration that should work for most DSMR smart meters using a serial connection. However, you may need to adjust the settings based on your specific setup. Upon your first installation, be mindful that you modify the configuration. Otherwise the add-on will start with default settings and most likely in mock mode. All options are required.
It also requires the MQTT service to be set up and running in Home Assistant and use the setting from there.

p1nrgy requires the following configuration options:

| Option                  | Description                                                                       | Default            |
| ----------------------- | --------------------------------------------------------------------------------- | ------------------ |
| `Enable DSMR`           | Enable reading from a DSMR smart meter via a serial connection.                   | `true`             |
| `DSMR mock`             | Enable mock mode for DSMR data (useful for testing without a real smart meter).   | `serial`           |
| `DSMR device`           | A list of serial devices to try for reading DSMR data.                            | `/dev/ttyUSB0`     |
| `DSMR serial baud`      | The baudrate (see `DSMR serial settings` table for more information).             | `115200`           |
| `DSMR serial data bits` | The parity and data bits (see `DSMR serial settings` table for more information). | `8`                |
| `DSMR serial parity`    | The parity (see `DSMR serial settings` table for more information).               | `N`                |
| `  HD mqtt added      ` |                                                                                   |                    |
| `mqtt host`             | IP adress of raspberry pi used (example 192.168.5.32).                            | ``                 |
| `mqtt port`             | Port MQTT listens to to.                                                          | `1883`             |
| `mqtt username`         | Username to log into EMQX.                                                        | ``                 |
| `mqtt password`         | Password to log into EMQX.                                                        | ``                 |
| `mqtt client id`        | Client ID for EMQX connection.                                                    | `p1nrgy_hass_addon`|
| `mqtt mqtt topic`       | mqtt topic for telegram data.                                                     | `p1nrgy/data`      |
| `  HD debug log added ` |                                                                                   |                    |
| `debug logging`         |  Toggle false / true for telegrams in supervisor log.                             | `false`            |

### DSMR Serial Settings

| DSMR Version     | Baudrate | Parity / Data Bits                         |
| ---------------- | -------- | ------------------------------------------ |
| DSMR 2.2         | 9600     | 7E1 (7 data bits, Even parity, 1 stop bit) |
| DSMR 3.0         | 9600     | 7E1                                        |
| DSMR 4.0         | 115200   | 8N1 (8 data bits, No parity, 1 stop bit)   |
| DSMR 4.2         | 115200   | 8N1                                        |
| DSMR 5.0 / 5.0.2 | 115200   | 8N1                                        |

## Usage

Once the add-on is installed and configured, start it from the Home Assistant Supervisor panel. The add-on will connect to the DSMR smart meter and begin publishing energy consumption data to the configured MQTT broker. p1nrgy will pubish on the topic: `p1nrgy/data`. From there, you can go as you please.

in my configuration.yaml I include a file with mqtt sensors 
```yaml
mqtt:
  sensor: !include mqtt_sensors.yaml
```
In that include file `mqtt_sensors.yaml` I have following example of a sensor that read the appropriate part of the telegram data

```yaml
- name: "p1nrgy Current Power Generation"
  unique_id: p1nrgy_current_power_generation
  state_topic: "p1nrgy/data"
  value_template: "{{ value_json.CurrentPowerGeneration }}"
  unit_of_measurement: "kW"
  device_class: power
  state_class: measurement
```
## Future plans

Once the basics are stable, the plan is to make an integration that can automatically set up the MQTT sensors in Home Assistant based on the data published by p1nrgy.

### Troubleshooting

Upon starting the add-on, you can check the logs to see if it successfully connects to the DSMR smart meter and MQTT broker. The logging will provide details if one of them failed.

[aarch64-shield]: https://img.shields.io/badge/aarch64-yes-green.svg
[amd64-shield]: https://img.shields.io/badge/amd64-yes-green.svg
[armhf-shield]: https://img.shields.io/badge/armhf-yes-green.svg
[armv7-shield]: https://img.shields.io/badge/armv7-yes-green.svg
[i386-shield]: https://img.shields.io/badge/i386-yes-green.svg
