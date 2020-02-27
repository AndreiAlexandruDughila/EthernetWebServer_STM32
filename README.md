## EthernetWebServer_STM32

[![arduino-library-badge](https://www.ardu-badge.com/badge/EthernetWebServer_STM32.svg?)](https://www.ardu-badge.com/EthernetWebServer_STM32)

This library currently supports
1. STM32 boards with built-in Ethernet such as :
  - ***Nucleo-144 (F429ZI, F767ZI)***
  - ***Discovery (STM32F746G-DISCOVERY)***
2. ***STM32 boards (with 64+K Flash) running EMC28J60 shields***

This is simple yet complete WebServer library for `STM32` boards running built-in Ethernet (Nucleo-144, Discovery) or EMC28J60 Ethernet shields. The functions are similar and compatible to ESP8266/ESP32 WebServer libraries to make life much easier to port sketches from ESP8266/ESP32.

The library supports 
1. HTTP Server and Client
2. HTTP GET and POST requests, provides argument parsing, handles one client at a time.

Library is based on and modified from:
1. [Ivan Grokhotkov's ESP8266WebServer](https://github.com/esp8266/Arduino/tree/master/libraries/ESP8266WebServer)

The EthernetWebServer class found in `EthernetWebServer.h` header, is a simple web server that knows how to handle HTTP requests such as GET and POST and can only support one simultaneous client.

## Prerequisite
1. [`Arduino IDE 1.8.11 or later` for Arduino](https://www.arduino.cc/en/Main/Software)
2. `Arduino AVR core 1.8.2 or later` for Arduino (Use Arduino Board Manager)
3. Depending on which Ethernet card you're using:
   - [STM32Ethernet library](https://github.com/stm32duino/STM32Ethernet) for built-in Ethernet on (Nucleo-144, Discovery)
   - [UIPEthernet library](https://github.com/UIPEthernet/UIPEthernet) for ENC28J60
4. [`Functional-VLPP library`](https://github.com/khoih-prog/functional-vlpp) to use lambda function
5. [LwIP library](https://github.com/stm32duino/LwIP) for built-in Ethernet on (Nucleo-144, Discovery)

## Installation

### Use Arduino Library Manager
Another way is to use `Arduino Library Manager` or [![arduino-library-badge](https://www.ardu-badge.com/badge/EthernetWebServer_STM32.svg?)](https://www.ardu-badge.com/EthernetWebServer_STM32). Search for `EthernetWebServer_STM32`, then select / install the latest version.

### Manual Install

1. Navigate to [EthernetWebServer_STM32](https://github.com/khoih-prog/EthernetWebServer_STM32) page.
2. Download the latest release `EthernetWebServer_STM32-master.zip`.
3. Extract the zip file to `EthernetWebServer_STM32-master` directory 
4. Copy whole 
  - `EthernetWebServer_STM32-master` folder to Arduino libraries' directory such as `~/Arduino/libraries/`.


#### Usage

#### Class Constructor

```cpp
  EthernetWebServer server(80);
```

Creates the EthernetWebServer class object.

*Parameters:* 
 
host port number: ``int port`` (default is the standard HTTP port 80)

#### Basic Operations

***Starting the server***

```cpp
  void begin();
```

***Handling incoming client requests***

```cpp
  void handleClient();
```

***Disabling the server***

```cpp
  void close();
  void stop();
```

Both methods function the same

***Client request handlers***

```cpp
  void on();
  void addHandler();
  void onNotFound();
  void onFileUpload();	
```

Example:*

```cpp
  server.on("/", handlerFunction);
  server.onNotFound(handlerFunction);   // called when handler is not assigned
  server.onFileUpload(handlerFunction); // handle file uploads
```

***Sending responses to the client***

```cpp
  void send();
  void send_P();
```

`Parameters:`

`code` - HTTP response code, can be `200` or `404`, etc.

`content_type` - HTTP content type, like `"text/plain"` or `"image/png"`, etc.

`content` - actual content body

#### Advanced Options

***Getting information about request arguments***

```cpp
  const String & arg();
  const String & argName();
  int args();
  bool hasArg();
```

`Function usage:`

`arg` - get request argument value, use `arg("plain")` to get POST body
	
`argName` - get request argument name
	
`args` - get arguments count
	
`hasArg` - check if argument exist

***Getting information about request headers***

```cpp
  const String & header();
  const String & headerName();
  const String & hostHeader();
  int headers();
  bool hasHeader();
``` 
`Function usage:`

`header` - get request header value

`headerName` - get request header name

`hostHeader` - get request host header if available, else empty string
  
`headers` - get header count
	
`hasHeader` - check if header exist

***Authentication***

```cpp
  bool authenticate();
  void requestAuthentication();
```

`Function usage:`

`authenticate` - server authentication, returns true if client is authenticated else false

`requestAuthentication` - sends authentication failure response to the client

`Example Usage:`

```cpp

  if(!server.authenticate(username, password)){
    server.requestAuthentication();
  }
```

#### Other Function Calls

```cpp
  const String & uri(); // get the current uri
  HTTPMethod  method(); // get the current method 
  WiFiClient client(); // get the current client
  HTTPUpload & upload(); // get the current upload
  void setContentLength(); // set content length
  void sendHeader(); // send HTTP header
  void sendContent(); // send content
  void sendContent_P(); 
  void collectHeaders(); // set the request headers to collect
  void serveStatic();
  size_t streamFile();
```


Also see examples: 
 1. [HelloServer](examples/HelloServer)
 2. [HelloServer2](examples/HelloServer2)
 3. [AdvancedWebServer](examples/AdvancedWebServer)
 4. [HttpBasicAuth](examples/HttpBasicAuth)
 5. [PostServer](examples/PostServer)
 6. [SimpleAuthentication](examples/SimpleAuthentication)

## Example
Please take a look at examples, as well.

```cpp
/*
 * Currently support 
 * 1) STM32 boards with built-in Ethernet (to use USE_BUILTIN_ETHERNET = true) such as :
 *    - Nucleo-144 (F429ZI, F767ZI)
 *    - Discovery (STM32F746G-DISCOVERY)
 * 2) STM32 boards (with 64+K Flash) running EMC28J60 shields (to use USE_BUILTIN_ETHERNET = false)
 * 
 */

#define USE_BUILTIN_ETHERNET   false    //true

#include <EthernetWebServer_STM32.h>
  
// Enter a MAC address and IP address for your controller below.

byte mac[] = {
  0xDE, 0xAD, 0xBE, 0xEF, 0xFE, 0xED
};

// Select the IP address according to your local network
IPAddress ip(192, 168, 2, 200);

EthernetWebServer server(80);

const int led = 13;

void handleRoot() 
{
  server.send(200, "text/plain", "Hello from EthernetWebServer");
}

void handleNotFound()
{
  String message = "File Not Found\n\n";
  message += "URI: ";
  message += server.uri();
  message += "\nMethod: ";
  message += (server.method() == HTTP_GET)?"GET":"POST";
  message += "\nArguments: ";
  message += server.args();
  message += "\n";
  for (uint8_t i=0; i<server.args(); i++)
  {
    message += " " + server.argName(i) + ": " + server.arg(i) + "\n";
  }
  server.send(404, "text/plain", message);
  digitalWrite(led, 0);
}

void setup(void)
{
  // Open serial communications and wait for port to open:
  Serial.begin(115200);
  delay(1000);
  Serial.println("\nStarting HelloServer on Nucleo-144");

  // start the ethernet connection and the server:
  Ethernet.begin(mac, ip);

  server.on("/", handleRoot);

  server.on("/inline", [](){
    server.send(200, "text/plain", "This works as well");
  });

  server.onNotFound(handleNotFound);

  server.begin();

  Serial.print(F("HTTP EthernetWebServer is @ IP : "));
  Serial.println(Ethernet.localIP());
}

void loop(void)
{
  server.handleClient();
}
```

The following are debug terminal output and screen shot when running example [AdvancedWebServer](examples/AdvancedWebServer) on Teensy 4.0

<p align="center">
    <img src="https://github.com/khoih-prog/EthernetWebServer_STM32/blob/master/pics/AdvancedWebServer.png">
</p>

```

Starting AdvancedServer
HTTP EthernetWebServer is @ IP : 192.168.2.100
[ETHERNET_WEBSERVER] send1: len =  289
[ETHERNET_WEBSERVER] content =  <html><head><meta http-equiv='refresh' content='5'/><title>ESP8266 Demo</title><style>body { background-color: #cccccc; font-family: Arial, Helvetica, Sans-Serif; Color: #000088; }</style></head><body><h1>Hello from ESP8266!</h1><p>Uptime: 00:00:27</p><img src="/test.svg" /></body></html>
[ETHERNET_WEBSERVER] send1: len =  1946
[ETHERNET_WEBSERVER] content =  <svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="310" height="150">
<rect width="310" height="150" fill="rgb(250, 230, 210)" stroke-width="1" stroke="rgb(0, 0, 0)" />
<g stroke="black">
<line x1="10" y1="77" x2="20" y2="67" stroke-width="1" />
<line x1="20" y1="67" x2="30" y2="98" stroke-width="1" />
<line x1="30" y1="98" x2="40" y2="111" stroke-width="1" />
<line x1="40" y1="111" x2="50" y2="90" stroke-width="1" />
<line x1="50" y1="90" x2="60" y2="22" stroke-width="1" />
<line x1="60" y1="22" x2="70" y2="98" stroke-width="1" />
<line x1="70" y1="98" x2="80" y2="64" stroke-width="1" />
<line x1="80" y1="64" x2="90" y2="104" stroke-width="1" />
<line x1="90" y1="104" x2="100" y2="31" stroke-width="1" />
<line x1="100" y1="31" x2="110" y2="59" stroke-width="1" />
<line x1="110" y1="59" x2="120" y2="139" stroke-width="1" />
<line x1="120" y1="139" x2="130" y2="117" stroke-width="1" />
<line x1="130" y1="117" x2="140" y2="75" stroke-width="1" />
<line x1="140" y1="75" x2="150" y2="72" stroke-width="1" />
<line x1="150" y1="72" x2="160" y2="137" stroke-width="1" />
<line x1="160" y1="137" x2="170" y2="20" stroke-width="1" />
<line x1="170" y1="20" x2="180" y2="94" stroke-width="1" />
<line x1="180" y1="94" x2="190" y2="81" stroke-width="1" />
<line x1="190" y1="81" x2="200" y2="38" stroke-width="1" />
<line x1="200" y1="38" x2="210" y2="33" stroke-width="1" />
<line x1="210" y1="33" x2="220" y2="53" stroke-width="1" />
<line x1="220" y1="53" x2="230" y2="88" stroke-width="1" />
<line x1="230" y1="88" x2="240" y2="32" stroke-width="1" />
<line x1="240" y1="32" x2="250" y2="110" stroke-width="1" />
<line x1="250" y1="110" x2="260" y2="87" stroke-width="1" />
<line x1="260" y1="87" x2="270" y2="11" stroke-width="1" />
<line x1="270" y1="11" x2="280" y2="98" stroke-width="1" />
<line x1="280" y1="98" x2="290" y2="76" stroke-width="1" />
<line x1="290" y1="76" x2="300" y2="121" stroke-width="1" />
</g>
</svg>
```

### Version v1.0.0

This is simple yet complete WebServer library for `STM32` boards running built-in Ethernet (Nucleo-144, Discovery) or EMC28J60 Ethernet shields. ***The functions are similar and compatible to ESP8266/ESP32 WebServer libraries*** to make life much easier to port sketches from ESP8266/ESP32.

This library currently supports
1. STM32 boards with built-in Ethernet such as :
  - Nucleo-144 (F429ZI, F767ZI)
  - Discovery (STM32F746G-DISCOVERY)
2. STM32 boards (with 64+K Flash) running EMC28J60 shields
3. HTTP Server and Client
4. HTTP GET and POST requests, provides argument parsing, handles one client at a time.

## TO DO
1. Bug Searching and Killing
2. Add SSL/TLS Client and Server support
3. Support more types of boards using Ethernet shields.


### Contributions and thanks
1. Forked from [Ivan Grokhotkov's ESP8266WebServer](https://github.com/esp8266/Arduino/tree/master/libraries/ESP8266WebServer)
2. [jandrassy](https://github.com/jandrassy) for [UIPEthernet library](https://github.com/UIPEthernet/UIPEthernet)

## Contributing

If you want to contribute to this project:
- Report bugs and errors
- Ask for enhancements
- Create issues and pull requests
- Tell other people about this library

## Copyright

Copyright 2020- Khoi Hoang


