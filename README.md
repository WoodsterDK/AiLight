# AiLight

AiLight is a custom firmware for the inexpensive Ai-Thinker RGBW WiFi RGB light bulbs that has the ESP8266 MCU at its core. Xose Pérez has written an excellent [article](http://tinkerman.cat/ailight-hackable-rgbw-light-bulb/) on his blog how you can upload your own firmware to this light.

![AiLight](https://www.sachatelgenhof.nl/user/pages/02.blog/ailight/screen_combo_m.png)

## Features

AiLight allows you to:

- switch the light on or off
- set the level of the 4 colour channels (Red, Green, Blue and White)
- set the brightness level
- set the light at a particular colour temperature
- let the light flash (with a given colour and brightness)
- keep the last known settings (colour, brightness, etc.) after a power cycle
- enable Gamma Correction to make the LED colours appear closer to what our eyes perceive

This can all be done in [Home Assistant](https://home-assistant.io) (using the MQTT built-in integration via JSON) or the built in (mobile friendly) UI. The UI interface also gives you the ability to configure the light remotely. You can easily change your WiFi settings or the configuration of your MQTT broker.

### Other

- MQTT Last Will and Testament enabled
- Support for Over The Air (OTA) firmware updates
- Preserve light settings and configuration after power cycle or restart
- Perform remote restart using the built-in HTML UI.
- Reset to factory defaults using the built-in HTML UI (* 'factory' here means the default settings of the AiLight firmware upon compile time)


**Roadmap**

- MQTT Discovery
- Effects
- Transitions
- ~~Remember light state after power cycle~~
- ~~MQTT Last Will And Testament~~
- ~~Gamma Correction~~

Making this firmware was largely inspired by the [MY9291](https://github.com/xoseperez/my9291) LED driver and the [Espurna](https://bitbucket.org/xoseperez/espurna) firmware of [Xose Pérez](https://github.com/xoseperez).

## Requirements

To build and upload this firmware you need to have [PlatformIO](http://platformio.org/) and [NodeJS](https://nodejs.org/en/) installed. Please check their respective websites for installation instructions.

## Installation/Configuration

1. Clone this AiLight repository and change your current directory to where AiLight is cloned.
2. Make a copy of the `config.example.h` file in the 'src' folder named `config.h`
3. Depending on your environment, set at least the following values in the `config.h` file:

  - WIFI_SSID `// Your WiFi SSID`
  - WIFI_PSK `// Your WiFi PSK/Password`
  - MQTT_SERVER `// The hostname/IP address of your MQTT broker`
  - MQTT_USER `// The username to connect to your MQTT broker`
  - MQTT_PASSWORD `// The password to connect to your MQTT broker`

  Leaving the WiFi settings blank, will make the AiLight start in AP mode (Access Point). From there you can access the   settings screen and enter your WiFi settings.

  Other configuration variables can be left as is, however feel free to adjust as you see fit.

4. Make a copy of the `platformio.example.ini` file name `platformio.ini`. This PlatformIO configuration doesn't require any changes, however it is recommended to change the OTA port number and OTA password when using your Ai-Thinker RGBW Light in production. (In that case don't forget to update the respective variables in your `config.h` file too).

5. In a terminal session (or in PlatformIO), run the command `npm install` (or `yarn install`). This is a required step that will install necessary NodeJS packages for building the UI interface. Make sure you do this in the directory where you cloned AiLight (e.g 'AiLight'), otherwise the NodeJS packages will not get installed.

6. Click on the "PlatformIO: Build" icon (or issue a "platformio run" command from the PlatformIO terminal).

That's all it takes, you're ready to go!

If no compilation errors popped up, you can start flashing the firmware to your Ai-Thinker RGBW Light using an FTDI (or alike) programmer. This is a required step of course, since your Ai-Thinker RGBW Light still has the factory firmware.

If the upload of he AiLight firmware was successful, it is recommended to restart your Ai-Thinker RGBW Light. This can be done by reconnecting the power of your FTDI programmer.

While connected to your FTDI programmer, check the output on your Serial Monitor. You should see some messages appear that will tell you details of the firmware, the light, hostname and the assigned IP address.

![AiLight](https://www.sachatelgenhof.nl/user/pages/02.blog/ailight/terminal_030.png)

The same information can be seen in the HTML UI (About screen). By default the Web interface can be accessed via 'http://AiLight-######.local' where '#####' is the unique ID of your Ai-Thinker RGBW Light. If you have changed your hostname, then of course the URL is different also.

From this point, you can upload newer versions of the firmware via OTA. Update the `upload_port` variable in your platformio.ini file with the 'hostname' value from the Serial Monitor / HTML UI. Now you can start using OTA to upload any updates of the firmware over the air.


### Home Assistant / MQTT
Now that the AiLight firmware is uploaded, the next step is making your Ai-Thinker RGBW Light present in [Home Assistant](https://home-assistant.io). To do that, add the following to your configuration.yaml file.

MQTT Broker:

    mqtt:
      broker: <YOUR_MQTT_SERVER_ADDRESS>
      port: 1883
      client_id: home-assistant
      username: <YOUR_MQTT_USERNAME>
      password: <YOUR_MQTT_PASSWORD>
Note: if your MQTT broker is set for anonymous access, you can leave the username and password variables out. Also, change the port number and client_id variables if your situation differs.

Light:

    light:
      platform: mqtt_json
      name: 'AiLight Office'
      state_topic: '<YOUR_MQTT_STATE_TOPIC>' (e.g. 'AiLight-CC8410')
      command_topic: '<YOUR_MQTT_COMMAND_TOPIC>' (e.g. 'AiLight-CC8410/set')
      color_temp: true
      brightness: true
      rgb: true
      white_value: true
      effect: false
      optimistic: false

Lastly, add your Ai-Thinker RGBW Light in a Home Assistant view. For example like this:

    group:
      default_view:
      lights:
        name: Lights
        entities:
          - light.ailight_office

Restart Home Assistant to load the new configuration. If everything goes according to plan, you should see the light appear in your Home Assistant.

![Home Assistant](https://www.sachatelgenhof.nl/user/pages/02.blog/ailight/hass.png)

### Environments

The PlatformIO configuration contains 4 environments for building and flashing your Ai-Thinker RGBW Light:

- 'env-dev': development/debug environment for building and uploading the firmware via an FTDI (or alike) programmer
- 'env-prod': production environment (optimized for size) for building and uploading the firmware via an FTDI (or alike) programmer

In addition, for each of these 2 environments, an OTA environment is available for uploading the firmware over OTA.

## Dependencies

- [MY9291](https://github.com/xoseperez/my9291) LED driver.
- AsyncMQTTClient library
- ArduinoJson library
- ESPAsyncTCP library
- ESPAsyncWebServer library

## Contributing

Contributions are encouraged and welcome; I am always happy to get feedback or pull requests on Github :)

## Credits and License

The Ai-Thinker RGBW Light Firmware is open-sourced software licensed under the [MIT license](http://opensource.org/licenses/MIT). For the full copyright and license information, please see the <license> file that was distributed with this source code.</license>
