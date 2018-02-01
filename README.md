## ESPShaker
*ESP8266 interactive serial command processor via Arduino core.* [![Build Status](https://travis-ci.org/Hieromon/ESPShaker.svg?branch=master)](https://travis-ci.org/Hieromon/ESPShaker)   
**Version 1.1 released. Available client for MQTT to publish/subscribe messages, also EEPROM reads and writes.**

It is implemented several APIs provided by ESP8266 arduino core so that they can be executed with interactive commands. You can control esp8266 interactively with the command via serial interface. With ESPShaker you can investigate the behavior of ESP8266, without having to write program code every time.  
Since ESPShaker does not use the [AT SDK](http://espressif.com/en/support/download/at) provided by Espressif Systems, it can examine the pure behavior by [ESP8266 arduino core](https://github.com/esp8266/Arduino).  

![screen_shot](https://user-images.githubusercontent.com/12591771/34244131-567b705e-e667-11e7-90a8-d751e3b63b5d.png)

### Features

* Implemented a commonly used basic WiFi API to the command
* Supports WIFI_AP, WIFI_STA, WIFI_AP_STA mode
* Supports *SmartConfig* with *ESP-TOUCH*
* GPIO port I/O
* SPIFFS file system handling
* Supports EEPROM read and write
* Built in Web server with http GET/PUT handling and DNS server, mDNS service
* It can build simple web pages interactively for http response
* MQTT message publish/subscribe providigation(Depends on [PubSubClient library](https://pubsubclient.knolleary.net/))
* Batch execution for a series of commands

### Works on

It has tested on NodeMCU v1.0 module (ESP-12E). Other ESP8266 modules will also work, but its sketch size exceeds 370KB. It may not work on prior modules such as ESP-01.  
Required [Arduino IDE](http://www.arduino.cc/en/main/software) is current upstream at **the 1.8 level or later**, and also [ESP8266 Arduino core 2.3.0](https://github.com/esp8266/Arduino).

### Installation

Download ESPShaker file as a zip from this repository, and extract the resulting folder into your Arduino sketch folder and build it with Arduino IDE. In addition to this file, two external libraries are required as [PseudoPWM](https://github.com/Hieromon/PseudoPWM) for LED blinking and [PageBuilder](https://github.com/Hieromon/PageBuilder) for html page dynamic creation. Also [PubSubClient library](https://github.com/knolleary/pubsubclient/releases/latest) is required for subscribing and publishing messages according to the MQTT protocol.  
Then open the serial monitor and reset the ESP8266 module. If ESPShaker is working properly the current WiFi status will be displayed on the serial monitor. The serial monitor's baudrate is 115200 bps.

### Usage

What you need to recognize is that the API sequence is important for proper communication with ESP8266. In other words, you can explore the error caused by the wrong sequence of the owned code. It will also be useful for analysis of unexpected return codes.  
Usage is simple. Connect ESP8266 module to PC and start the serial monitor. Then enter the following command. A specific API is called and its response is displayed on the serial monitor. ESPShaker also works with other serial terminal utilities. Local echo setting is needed.

#### General commands sequence

- AP
  ```
  mode ap
  apconfig 192.168.4.1 192.168.4.1 255.255.225.0
  softap myssid mypassword
  ...here, you will connect the access point named myssid from your wi-fi client device.
  http on /hello Hello, world!
  start web
  ...here, you can access the web server in ESPShaker at http://192.168.4.1/hello
  start mdns mywebserver http tcp
  ...here, you can access the web server in ESPShaker at http://mywebserver.local/hello
  ```

- STA
  ```
  mode sta
  begin anyssid password
  -> some connection status response
  http get http://some.where.url/
  -> some response from server
  ```
- AP_STA
  ```
  mode apsta
  begin anyssid password
  -> WiFi.Status(WL_CONNECTED)
  apconfig 192.168.4.1 192.168.4.1 255.255.225.0
  softap myssid mypassword
  http on /hello Hello, world!
  start web
  start mdns mywebserver http tcp
  ... 
  ```

#### Commands

##### Syntax

```
command [operand [...]]
```
Enter `command` and `operand` separated by blanks. Depending on the command, it may need to specify more than one operand. The command terminated by `LF` code (\n). Therefore, it is necessary to terminate the serial monitor transmission with LF by `ENTER` key. 

##### Command summary

| Command | Description | Used Arduino core's API |
| --- | --- | --- |
| apconfig | Set IP addresses of AP mode | WiFi.softAPConfig |
| autoconnect | Set auto connect validity | WiFi.setAutoConnect |
| begin | Begin WiFi communication | WiFi.begin |
| delay | Delay milliseconds | delay |
| discon | Disconnect WiFi | WiFi.disconnect |
| eeprom | read and write eeprom data | EEPROM.read, EEPROM.write |
| event | Notify WiFi event occurrence | WiFi.onEvent |
| fs | SPIFFS file system handling | SPIFFS |
| gpio | GPIO access | deigitalRead, digitalWrite |
| http | Issues HTTP GET method or Register web page | HTTPClient.begin and get, ESP8266WebServer.addHandler |
| mode | Set Wi-Fi working mode | WiFi.mode |
| mqtt | MQTT message publish/subscribe | Depends on PubSubClient library |
| persistent | Set whether to store WiFi working mode in flash memory | WiFi.persistent |
| reset | Reset esp8266 module | ESP.reset |
| scan | Scan all available APs | WiFi.scanNetworks |
| show | Display the current IPs the module has | WiFi.localIP and Others |
| sleep | Set sleep type or enter a deep sleep | WiFi.setSleepMode or ESP.deepSleep |
| smartconfig | Configure new SSID with [ESP-TOUCH](http://espressif.com/en/products/software/esp-touch/resources) | WiFi.beginSmartConfig |
| softap | Start SoftAP operation or stops it | WiFi.softAP |
| start | Start Web server or DNS server | ESP8266WebServer.begin or DNSServer.start |
| station | Display number of stations current connected SoftAP | WiFi.softAPgetStationNum |
| status | Display current connection status of station | WiFi.status |
| stop | Stop server | ESP8266WebServer.close or DNSServer.stop |
| wps | Begin WPS configuration | WiFi.beginWPSConfiguration |

The **WiFi** object is an exported instance of **ESP8266WiFiClass** described in **ESP8266WiFi.h**. The **HTTPClient** is defined in **ESP8266HTTPClient.h**. **ESP8266WebServer** is defined in **ESP8266WebServer.h** and **DNSServer** class is declared in **DNSServer.h**.  
Enter `help` or `?` will display commands list.

#### Command specifications

1. **apconfig**  
   Configure IPs of AP mode operation.
   ```
   apconfig [AP_IP] [GW_IP] [NETMASK]
   ```
   `AP_IP` : AP's local IP address. If not specified, **192.168.4.1** is assumed as the default.  
   `GW_IP` : Gateway IP address. If not specified, **192.168.4.1** is assumed as the default.  
   `NETMASK` : Subnet mask. If not specified, **255.255.255.0** is assumed as the default.

2. **autoconnect**  
   Set the ESP8266 Station to connect to the AP (whose ID is cached) automatically or not when powered on.
   ```
   autoconnect on | off
   ```
   `on` : Auto-connect on.  
   `off` : Auto-connect off.

3. **begin**  
   Connect to the AP and starts working the WiFi station. Specified `SSID` and `PASSPHRASE` would be saved to the flash in ESP8266 module.  
   ```
   begin [SSID [PASSPHRASE]] [#wait]
   ```
   `SSID` : SSID that should be connected.  
   `PASSPHRASE` : Passphrase.  
   `#wait` : Wait for WiFi connection status changed.  
   If `SSID` or `PASSPHRASE` omitted, the previous value in the flash will be used.  
   For `#wait` option is specified, it waits for the next command processing until WiFi connection status transition. This option is useful when storing a series of commands in a file and executing as batch process. For batch processing, refer to [Batch process](#Batch-process).

4. **delay**  
   Pauses the program for the amount of time in milliseconds.
   ```
   delay MILLISECONDS
   ```
   `MILLISECONDS` : The amount time to delay in milliseconds uint.  
   **'delay'** command is almost for batch processing.

5. **discon**  
   Disconnects Wi-Fi.
   ```
   discon [ap]
   ```
   `ap` : Disconnect station from ESP8266 in SoftAP mode by **WiFi.softAPdisconnect**. If this operand is not specified, it disconnects ESP8266 from the access point by **WiFi.disconnect**.

6. **eeprom**  
   Read and  write to EEPROM.
   ```
   eeprom addr [ADDRESS]
   eeprom clear BYTE LENGTH
   eeprom write DATA
   eeprom read LENGTH
   ```
   + `eeprom addr [ADDRESS]` : Display current EEPROM address, If **ADDRESS** is specified then the current address changed to it.
   + `eeprom clear BYTE LENGTH` : Fill the area of **LENGTH** size from the current address with **BYTE**.
   + `eeprom write DATA` : Writes **DATA** to the current address of EEPROM.
   + `eeprom read LENGTH` : Display data with **LENGTH** size from the current address of EEPROM.  

   The each parameter as **ADDRESS**, **LENGTH**, **BYTE** and **DATA** are hexadecimal. **BYTE** is 2-digits hexadecimal is actually. **DATA** can be specified with continuous long hexadecimal digit like as `68656c6c6f2c20776f726c64`. The **clear**, **write** and **read** commands automatically increment the current EEPROM address. **ADDRESS** ranges from 0 to FFF are valid and if the address reached at end of EEPROM, it rounds to 0.  

7. **event**  
   Detects events with **WiFi.onEvent** method and displays events.
   ```
   event on | off
   ```
   `on` : Displays that an event has occurred.  
   `off` : No happens at events, but the callback on **onEvent** remains yet.  
   When an event occurs, its message would be displayed as the below.  

   > \> softap esp8266ap 12345678  
   > \> event on  
   > [event]:7 (WIFI_EVENT_SOFTAPMODE_PROBEREQRECVED)  
   > [event]:7 (WIFI_EVENT_SOFTAPMODE_PROBEREQRECVED)  
   > ... WiFi client was connected.  
   > [event]:5 (WIFI_EVENT_SOFTAPMODE_STACONNECTED)  

8. **fs**  
   Handles SPIFFS file system.
   ```
   fs start
   fs dir | info
   fs format
   fs file PATH
   fs remove PATH
   fs rename PATH NEW_PATH
   ```
   + `fs start` : Mounts SPIFFS file system and starts FS API. Just launching ESPShaker does not mount SPIFFS. It is necessary to mount it with **fs start** command before starting FS operation.  
   + `fs dir | info` command displays the information of directory or file system.  
     + `dir` : Display directory list.  
     + `info` : Display the file system information.  
   + `fs format` : Formats the file system.  
   + `fs file PATH` : Reads from file specified `PATH` and displays read data.  
   + `fs remove PATH` : Remove file specified `PATH`.  
   + `fs rename PATH NEW_PATH` :  Rename file specified `PATH` to `NEW_PATH`.
   
9. **gpio**  
   Gets or sets GPIO port value. 
   ```
   gpio get PORT_NUM
   gpio set PORT_NUM low | high
   ```
   <img  align="right" alt="esp12_pins" width="320" src="https://arduino-esp8266.readthedocs.io/en/latest/_images/esp12.png">**'gpio'** command operates on the specified GPIO. It is available 0 (**GPIO0**) to 16 (**GPIO16**) but there are limitations. The following is a quote from [ESP8266 Arduino Core's documentation](https://arduino-esp8266.readthedocs.io/en/latest/reference.html#digital-io):  
   > The diagram below shows pin mapping for the popular ESP-12 module. Digital pins 6—11 are not shown on this diagram because they are used to connect flash memory chip on most modules. Trying to use these pins as IOs will likely cause the program to crash.  

   So 6 to 11 can not be specified for safety reasons.  
   + `get PORT_NUM` : Reads the status of GPIO specified `PORT_NUM`. The port value is read digitally and is either `LOW` or `HIGH`. At that time the **pinMode** is not changed.  
   + `set PORT_NUM low | high` : Sets the specified `PORT_NUM` to `low` or `high`.

10. **http**  
   Issues the GET method to specified Web site or define web page as plain texts.
   ```
   http get URL
   http on URI PAGE_CONTENT
   ```
   + **'http get'** command issues a GET method to the URL and displays the response of the content including the header from server.  
   `URL` : Specify url to GET like `http://www.example.com/bar.html`.  
   + **'http on'** command dynamically creates a simple web page consisting only of text with the following operands.  
   `URI` : Specify uri of the simple web page to be created.  
   `PAGE_CONTENT` : A page content text. However, it can include simple HTML tags. It is also possible to specify content files on SPIFFS. Specify by appending the prefix `file:` to the file name, it will automatically read that file.  
   + e.g.  
     + `http get http://www.google.com/` will display the response from the Google server. At this time, ESP8266 module must be connected to Internet via the other AP with STA mode or AP_STA mode.  
     + `http on /hello Hello, world!` creates 'Hello, world!' page in the embedded Web server of ESPShaker. When a client such as a smartphone accesses the uri that is running the ESPShaker embedded web server, its web page prepared by **http on** is available.  

     <br>![httpon](https://user-images.githubusercontent.com/12591771/34244185-847f135c-e667-11e7-9da6-aba2740f683b.png)  
     <br>In the above, switch to SoftAP mode and register **/hello** page contents with **'http on'** command. Then start and access the embedded Web server with **'start web'** command. A response from /hello looks like as below. If not registered page is accessed a response of 404 will appear.  
     <br>![r_sc_helloworld](https://user-images.githubusercontent.com/12591771/34244216-a2fa4cde-e667-11e7-94b4-48ec62f3212c.png) &nbsp;&nbsp; ![r_sc_404](https://user-images.githubusercontent.com/12591771/34244243-b5f98cd2-e667-11e7-9655-524afbf4085e.png)  

11. **mode**  
   Set Wi-Fi working mode to Station mode (**WIFI_STA**), SoftAP (**WIFI_AP**) or Station + SoftAP (**WIFI_APSTA**), and save it in flash. Immediately after resetting, the default mode is SoftAP mode.
   ```
   mode ap | sta | apsta | off
   ```
   `ap` :  Set WIFI_AP mode.  
   `sta`  : Set WIFI_STA mode.  
   `apsta` : Set WIFI_APSTA mode.  
   `off` : Shutdown Wi-Fi working.  

12. **mqtt**  
    Publish/subscribe messages between ESP8266 and a server using the MQTT protocol. The actual execution of this command is based on the PubSubClient library. The **mqtt** command does **not** definitely the native support for MQTT over **Websockets**.
    ```
    mqtt server SERVER [PORT]
    mqtt con CLIENT_ID USER PASSWORD
    mqtt pub TOPIC PAYLOAD [#r]
    mqtt sub TOPIC [QoS] | stop
    mqtt close
    ```
    - `mqtt server SERVER [PORT]` : Sets the MQ Telemetry Transport message broker address as the server and specifies used port.  
      - `SERVER` : The mqtt broker address. It can be specified domain or IP address.  
      - `PORT` : Port used for MQTT protocol. Default port is 1883. To use encrypted communication with TLS, specify 8883. ESPShaker does not supported client certificate with port 8884. 
    - `mqtt con CLIENT_ID USER PASSWORD` : Connects ESP8266 as a **CLIENT_ID** to the server along with specified **USER** and **PASSWORD**.  
      - `CLIENT_ID` : The identifier string that identifies a MQTT client. Each identifier must be unique to only one connected client at a time.  
      - `USER` : Specify the user name to connect to the server.  
      - `PASSWORD` : Specify the password to connect to the server.  
    - `mqtt pub TOPIC PAYLOAD [#r]` : Publishes a message described in **PAYLOAD** to the specified **TOPIC**.   
      - `TOPIC` :  The message topic.  
      - `PAYLOAD` :  The message payload.  
      - `#r` : Retains the message in the server and pass it to the new subscriber.  
    - `mqtt sub TOPIC [QoS]` : Subscribes to messages.  
      - `TOPIC` :  Specify the topic to subscribe.  
      - `QoS` : Subscribe at 0 or 1. Default is 0.  
    - `mqtt sub stop` : Stops subscriptions for recently specified topic.  
    - `mqtt close` : Stops all subscriptions and close the connection with the server. However, the server information set by **mqtt server** is retained. 

    To send and receive messages with the **mqtt** command, first specify the server with the **mqtt server**. Then connect as client with the **mqtt con**. For example, the MQTT message using Watson IoT on IBM Bluemix is as follows.

    > \> mqtt server **YOUR_ORG**.messaging.internetofthings.ibmcloud.com 8883  
    > \> mqtt con d:**YOUR_ORG**:**DEVICE_TYPE**:**DEVICE** use-token-auth **DEVICE_TOKEN**  
    > \> mqtt sub iot-2/cmd/+/fmt/json  
    > \> mqtt pub iot-2/evt/eid/fmt/json {"d":{"GPIO2":false}}  
    > [info] mqtt topic:iot-2/cmd/cid/fmt/json  
    > [info] mqtt payload(13):  
    > 000  7b 22 52 53 54 22 3a 22 ef bc 90 22 7d  {"RST":"０"}  

    This procedure publish an **evt** event as **{"d":{"GPIO2":false}}** while subscribing to **cmd** topic. When **cmd** as ** {"RST":"０"}** coded by UTF-8 is published to the server, its payload is displayed automatically.

13. **persistent**  
    Set whether to store WiFi working mode in flash memory.
    ```
    persistent on | off
    ```
    `on` : Store WiFi working mode in ESP8266's flash memory.  
    `off` : WiFi current working mode is not stored. It would be restored previously after reset.  
    The working mode at the time when **persistent** is `on` is not memorized. It would be memorized from **after setting** `on`.

14. **reset**  
    Reset the module.
    ```
    reset
    ```
    This command invokes `ESP.reset()` function.

15. **scan**  
   Scan all available APs.
   ```
   scan
   ```

16. **show**  
   Display current module saved values as follows.  
   ```
   show
   ```
   The show command allows displaying the following information.  
   + Auto-connect setting.  
   + Host name.  
   + SoftAP MAC Address.  
   + SoftAP IP address.  
   + Local MAC Address.
   + Station local IP address.  
   + Gateway IP address.  
   + Subnet mask.  
   + Saved SSID.  
   + Saved PSK.  
   + RSSI.  
   + Chip configuration.
   + Free heap size.  

17. **sleep**  
    Set sleep mode or embarks a deep sleep.
    ```
    sleep none | light | modem
    sleep deep SLEEP_TIME
    ```
    - `sleep` command sets wifi sleep type for power saving.
      - `none` :  Disable power saving.
      - `light` : Enable Light-sleep mode.
      - `modem` : Enable Modem-sleep mode. 
    - **'sleep deep'** command invokes **ESP.deepSleep** API. It automatically wakes up when `SLEEP_TIME` out.  
     Upon waking up, ESP8266 boots up from setup(), but *GPIO16* and *RST* must be connected. For reference, *GPIO16* is assigned to *D0* on **ESP-12** (NodeMCU) and is **not connected** to external pin with **ESP-01**.
      - `SLEEP_TIME` : Time from deep sleep to waking up (in microseconds unit).  

18. **smartconfig**  
   Start or stop Smart Config by ESP-TOUCH.<img  align="right" alt="esp-touch" src="https://user-images.githubusercontent.com/12591771/33641022-18f32c64-da77-11e7-8492-5460b78d4466.png">
   ```
   smartconfig start | stop | done
   ```
   The *Smart Config* is based on the [ESP-TOUCH](http://espressif.com/en/products/software/esp-touch/resources) protocol developed by Espressif Systems. It is seamlessly configure Wi-Fi devices to connect to the router.  
   ESP-TOUCH Apps for smartphone are released iOS and Android both. *ESP8266 SmartConfig* for iOS is [here](https://itunes.apple.com/jp/app/espressif-esptouch/id1071176700) and for Android is [here](https://play.google.com/store/apps/details?id=com.cmmakerclub.iot.esptouch). So it can be established ESP8266 to the new AP which is connected with smartphone already.  
   - `start` : Start Smart Config. Transits to this mode, then turns on the SmartConfig Apps on a smartphone manually.
   - `stop` : Stop Smart Config.
   - `done` : Inquiry a status of Smart Config.

19. **softap**  
   Start SoftAP operation.
   ```
   softap SSID PASSPHRASE
   ```
   `SSID` : Specify SSID for SoftAP.  
   `PASSPHRASE` : Specify Passphrase for the SSID.

20. **start**  
    Start Web server, DNS server, mDNS service.
    ```
    start web
    start dns [DOMAIN]
    start mdns HOST_NAME SERVICE PROTOCOL [PORT]
    ```
    - **'start web'** command starts web server inside ESPShaker. The http port assumed **#80**.  
    - **'start dns'** command starts DNS server inside ESPShaker. The dns port assumed **#53**.  
    To start the DNS server, the domain name is needed in the operand. It domain's IP address would be assumed SoftAPIP. At DNS server started, the error reply code would be set to `DNSReplyCode::NoError`.  
      - `DOMAIN` : Specify domain name. It assumes `*` if specified **DOMAIN** is missing. 
    - **'start mdns'** command starts mDNS responder.  
       - `HOST_NAME` : Specify mDNS host name. Actual host name would be resolved as 'HOST_NAME.local'.  
       - `SERVICE` : Specifies the service name to which mDNS responds. Like `http` for example.  
       - `PROTOCOL` : Specifies the protocol like `tcp`.  
       - `PORT` : Specifies the port of the service. When service is http, The port operand could be omitted and assumed **#80**.  

21. **station**  
    Display number of connected stations for SoftAP.
    ```
    station
    ```

22. **status**  
    Display current Wi-Fi status.
    ```
    status
    ```
    Display Wi-Fi connection status via WiFi.status() function. The return value described wl_status_t enum.

23. **stop**  
    Stop ESPShaker's internal server.
    ```
    stop web | dns
    ```
    `web` : Stop Web server.  
    `dns` : Stop DNS server.

24. **wps**  
    Begin WPS configuration.
    ```
    wps
    ```

#### Batch process

If you use a ordinary terminal emulator instead of the Arduino IDE serial monitor for interactive operation, you can process the commands in batches. In one example, [TeraTerm](https://osdn.net/projects/ttssh2/) as emulator can upload text file and can be used to continuously process commands.  

<img align="left" alt="command text file" src="https://user-images.githubusercontent.com/12591771/34244273-de6c4f10-e667-11e7-84bb-27b2157f4ea9.png" />Prepare consecutive commands in a text file. Include `LF` in the line feed code. Output it to the port using transmission function for dumb text file such as "Send file" of the serial terminal emulator.  

![send_file](https://user-images.githubusercontent.com/12591771/34244293-faa2b94e-e667-11e7-86c1-4dc8b9589331.png)  
Executed as follows.  

![term](https://user-images.githubusercontent.com/12591771/34244321-193e88ec-e668-11e7-901b-c128f93700bc.png)  

ESP8266 UART RX buffer size is 128 bytes also ESPShaker has 128 bytes command buffer. The USB Serial bridge interface fetches the contents of the received packet to the hardware buffer all at once. But ESP8266 SDK does not support hardware flow control and RX interrupt can not be properly controlled from the sketch. So at this way, a whole sending data may not be received.

### Change log

#### [1.1] 2018-01-31
- Supports **eeprom** command.
- Supports **mqtt** command.
- Added **Arduino core version** to **show** command output.
- Added **Chip ID** to **show** command output.
- Added **Free heap size** to **show** command output.
- Added **RSSI** to **show** command output.
- Added **SPIFFS** file system size to **show** command output.
- Added start offset of **EEPROM** to **show** command output.
- Suppressed compiler diagnostic as -Wdeprecated-declarations for onEvent method.

#### [1.04] 2017-12-27
- Changed **DOMAIN** operand default of **start dns** command.

#### [1.03] 2017-12-21
- Supports **delay** command.
- Supports **persistent** command.
- Supports **file:** option in **http on** command.
- Added **#wait** option to **begin** command.
- Added **ap** option to **discon** command.
- Modified serial command parser for batch process.

#### [1.02] 2017-12-08
- Supports **fs** command.
- Supports **gpio** command.
- Fixed crash when **apconfig** command's IP address was not specified.
- Changed **apconfig**'s IP address default value.

#### [1.01] 2017-12-06
- Supports **event** command.
- Supports **sleep** command.
- Supports **smartconfig** command.
- Supports **wps** command.
- Added sleep mode status in show command.

#### [1.0] 2017-11-23
- First Release.

### License

ESPShaker is licensed under the [MIT License](LICENSE). Copyright &copy; 2017-2018 hieromon@gmail.com  
[Arduino-SerialCommand](https://github.com/kroimon/Arduino-SerialCommand) by [kroimon](https://github.com/kroimon) licensed under the [GNU General Public License](http://www.gnu.org/licenses/).