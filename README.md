## ESPShaker
*ESP8266 interactive serial command processor via Arduino core.* [![Build Status](https://travis-ci.org/Hieromon/ESPShaker.svg?branch=master)](https://travis-ci.org/Hieromon/ESPShaker)   

It is implemented several APIs provided by ESP8266 arduino core so that they can be executed with interactive commands. You can control esp8266 interactively with the command via serial interface. With ESPShaker you can investigate the behavior of ESP8266, without having to write program code every time.  
Since ESPShaker does not use the [AT SDK](http://espressif.com/en/support/download/at) provided by Espressif Systems, it can examine the pure behavior by [ESP8266 arduino core](https://github.com/esp8266/Arduino).  

![screen_shot](https://user-images.githubusercontent.com/12591771/33471881-b76d6332-d6b2-11e7-9603-6ff4cc6ac6ed.png)

### Features

* Implemented a commonly used basic WiFi API to the command
* Supports WIFI_AP, WIFI_STA, WIFI_AP_STA mode
* Built in Web server and DNS server
* Supports http get method
* It can build simple web pages interactively for response

### Works on

It has tested on NodeMCU v1.0 module (ESP-12E). Other ESP8266 modules will also work.  
Required [Arduino IDE](http://www.arduino.cc/en/main/software) is current upstream at **the 1.8 level or later**, and also [ESP8266 Arduino core 2.3.0](https://github.com/esp8266/Arduino).

### Installation

Download ESPShaker file as a zip from this repository, and extract the resulting folder into your Arduino sketch folder and build it with Arduino IDE. In addition to this file, two external libraries are required as [PseudoPWM](https://github.com/Hieromon/PseudoPWM) for LED blinking and [PageBuilder](https://github.com/Hieromon/PageBuilder) for html page dynamic creation.  
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
  ...here, you cann access the web server in ESPShaker at http://192.168.4.1/hello
  start mdns mywebserver http tcp
  ...here, you cann access the web server in ESPShaker at http://mywebserver.local/hello
  ```

- STA
  ```
  mode sta
  begin anyssid password
  -> WiFi.Status(WL_CONNECTED) or WiFi.Status(WL_NO_SSID_AVAIL) or WiFi.Status(WL_CONNECT_FAILED)
  http get http://some.where.url/
  -> response from server
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
| discon | Disconnect WiFi | WiFi.disconnect |
| http | Issues HTTP GET method or Register web page | HTTPClient.begin and get, ESP8266WebServer.addHandler |
| mode | Set Wi-Fi working mode | WiFi.mode |
| scan | Scan all available APs | WiFi.scanNetworks |
| show | Display the current IPs the module has | WiFi.localIP and Others |
| softap | Start SoftAP operation or stops it | WiFi.softAP |
| start | Start Web server or DNS server | ESP8266WebServer.begin or DNSServer.start |
| station | Display number of stations current connected SoftAP | WiFi.softAPgetStationNum |
| status | Display current connection status of station | WiFi.status |
| stop | Stop server | ESP8266WebServer.close or DNSServer.stop |
| reset | Reset esp8266 module | ESP.reset |

The `WiFi` object is an exported instance of `ESP8266WiFiClass` described in `ESP8266WiFi.h`. The `HTTPClient` is defined using `ESP8266HTTPClient.h`. `ESP8266WebServer` is defined class in `ESP8266WebServer.h` and `DNSServer` class is declared in `DNSServer.h`.  
Enter `help` or `?` will display commands list.

#### Command specifications

1. appconfig  
   Configure IPs of AP mode operation.
   ```
   apconfig AP_IP GW_IP NETMASK
   ```
   `AP_IP` : AP's local IP address.  
   `GW_IP` : Gateway IP adderss.  
   `NETMASK` : Subnet mask.

2. autoconnect  
   Set the ESP8266 Station to connect to the AP (whose ID is cached) automatically or not when powered on.
   ```
   autoconnect on | off
   ```
   `on` : Auto-connect on.  
   `off` : Auto-connect off.

3. begin  
   Connect to the AP and starts working the WiFi station. Specified `ssid` and `pass` would be saved to the flash in ESP8266 module.  
   ```
   begin [SSID [PASS]]
   ```
   `SSID` : SSID that should be connected.  
   `PASS` : Passphrase.  
   If `SSID` or `PASS` omitted, the previous value in the flash will be used.

4. discon  
   Disconnects Wi-Fi Station from AP.
   ```
   discon
   ```

5. http  
   Issues the GET method to specified Web site or define web page as plain texts.
   ```
   http get URL
   http on URI PAGE_CONTENT
   ```
   + `http get` command issues a GET method to the URL and displays the response of the content including the header from server.  
   `URL` : Specify url to GET like `http://www.example.com/bar.html`.  
   + `http on` command dynamically creates a simple web page consisting only of text with the following operands.  
   `URI` : Specify uri of the simple web page to be created.  
   `PAGE_CONTENT` : A page content text. However, it can include simple HTML tags as `<p>`, `<b>`.  
   + e.g.  
   `http get http://www.google.com/` will display the response from the http\://www.google.com server. At this time, ESP8266 module must be connected to the other AP connected by WiFi.begin to the Internet in STA mode or AP_STA mode.  
   `http on /hello Hello, world!` creates 'Hello, world!' page in the internal Web server of ESPShaker. When a client like smartphone accesses to uri of ESPShaker internal Web server which running in AP mode (e.g. http\://192.168.4.1/hello), it should see the page the created by `http on` command.

6. mode  
   Set Wi-Fi working mode to Station mode (WIFI_STA), SoftAP (WIFI_AP) or Station + SoftAP (WIFI_APSTA), and save it in flash. Immediately after resetting, the default mode is SoftAP mode.
   ```
   mode ap | sta | apsta | off
   ```
   `ap` :  Set WIFI_AP mode.  
   `sta`  : Set WIFI_STA mode.  
   `apsta` : Set WIFI_APSTA mode.  
   `off` : Shutdown Wi-Fi working.  

7. scan  
   Scan all available APs.
   ```
   scan
   ```

8. show  
   Display current module saved values as follows.  
   ```
   show
   ```
   The show command allows displaying the following information.  
   + Auto-connect setting.  
   + Host name.  
   + SoftAP MAC Adress.  
   + SoftAP IP address.  
   + Local MAC Address.
   + Station local IP address.  
   + Gateway IP address.  
   + Subnet mask.  
   + Saved SSID.  
   + Saved PSK.  
   + Chip configuration.

9. softap  
   Start SoftAP operation.
   ```
   softap SSID PASS
   ```
   `SSID` : Specify SSID for SoftAP.  
   `PASS` : Specify pass-phrase for the SSID.

10. start  
    Start Web server, DNS server, mDNS service.
    ```
    start web
    start dns DOMAIN
    start mdns HOST_NAME SERVICE PROTOCOL [PORT]
    ```
    - `start web` command starts web server inside ESPShaker. The http port assumed #80.  
    - `start dns` command starts DNS server inside ESPShaker. The dns port assuemd #53. To start the DNS server, the domain name is needed in the operand. It domain's IP address would be assumed SoftAPIP.  
      - `DOMAIN` : Specify domain name.  
    - `start mdns` command starts mDNS responder.  
       - `HOST_NAME` : Specify mDNS host name. Actual host name would be resolved as 'HOST_NAME.local'.  
       - `SERVICE` : Specifies the service name to which mDNS responds. Like `http` for example.  
       - `PROTOCOL` : Specifies the protocol like `tcp`.  
       - `PORT` : Specifies the port of the service. When service is http, The port operand could be omitted and assumed #80.  

11. station  
    Display number of connected stations for SoftAP.
    ```
    station
    ```

12. status  
    Display current Wi-Fi status.
    ```
    status
    ```
    Display Wi-Fi connection status via WiFi.status() function. The return value described wl_status_t enum.

13. stop  
    Stop ESPShaker's internal server.
    ```
    stop web | dns
    ```
    `web` : Stop web server.  
    `dns` : Stop DNS server.

14. reset  
    Reset the module.
    ```
    reset
    ```
    This command invokes `ESP.reset()` function.


### License

ESPShaker is licensed under the [MIT License](LICENSE). Copyright &copy; 2017 hieromon@gmail.com  
[Arduino-SerialCommand](https://github.com/kroimon/Arduino-SerialCommand) by [kroimon](https://github.com/kroimon) licensed under the [GNU General Public License](http://www.gnu.org/licenses/).