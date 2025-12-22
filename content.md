---
title: 'Sending HTTP requests using Finder OPTA'
description: 'Learn how to send HTTP requests over Ethernet using Finder OPTA.'
author: 'Fabrizio Trovato'
libraries:
  - name: 'ArduinoHttpClient'
    url: https://www.arduino.cc/reference/en/libraries/arduinohttpclient/
difficulty: easy
tags:
  - HTTP
  - Ethernet
software:
  - ide-v1
  - ide-v2
  - arduino-cli
  - web-editor
hardware:
  - hardware/07.opta/opta-family/opta
---

## Overview

In this tutorial, we will learn how to send POST requests via Ethernet using
Finder OPTA. Thanks to its built-in Ethernet connectivity, Finder OPTA can
easily communicate with a remote server using the HTTP protocol. We will set a
static IP address for Finder OPTA and then send POST requests to a specific HTTP
server. The server response will be captured and displayed on the serial
monitor, allowing us to monitor the entire interaction in real time.

The most common use cases for sending HTTP requests on embedded devices include
communicating with web services for monitoring, remote control, or data logging.
For example, Finder OPTA can send requests to a server to receive a
configuration or to send telemetry data.

## Requirements

### Hardware Requirements

* Finder OPTA PLC (x1).
* USB-C® cable (x1).
* ETH RJ45 cable (x1).

### Software

* [Arduino IDE 2.0+](https://www.arduino.cc/en/software) or [Arduino Web
Editor](https://create.arduino.cc/editor).
* If you use the offline Arduino IDE, you must install the
`ArduinoHttpClient` library using the Library Manager of
the Arduino IDE.
* [Example code](assets/OptaHttpClientTutorial.zip).
* An HTTP server: for simplicity, we recommend
  [http-echo-server](https://github.com/watson/http-echo-server), a server
  that is easy to set up on any computer with a few commands. This server
  responds to requests by returning an "echo" of the content sent,
  making it ideal for testing HTTP communication.

### Connectivity

To follow this tutorial, Finder OPTA must be connected via Ethernet to a device
capable of routing packets from Finder OPTA to the HTTP server, and vice versa.

## Finder OPTA and the HTTP Protocol

Thanks to the `ArduinoHttpClient` library, Finder OPTA can easily generate and
send HTTP POST requests. To enable communication via the HTTP protocol, it is
necessary to configure a network connection: in this case we use the
`EthernetClient` class.

## Instructions

### Arduino IDE Configuration

To follow this tutorial, you will need [the latest version of the Arduino
IDE](https://www.arduino.cc/en/software). If this is your first time setting up
a Finder OPTA, check out the tutorial
[Getting Started with OPTA](https://opta.findernet.com/en/tutorial/getting-started):
in this tutorial, we explain how to install the Board Manager for the Mbed OS
OPTA platform, which is the set of basic tools needed to create and use a
sketch for Finder OPTA with Arduino IDE.

Make sure you install the latest version of the
[`ArduinoHttpClient`](https://www.arduino.cc/reference/en/libraries/arduinohttpclient/)
library, as it will be used to implement the communication with the server.

For further details on how to manually install libraries refer to [this
article](https://support.arduino.cc/hc/en-us/articles/5145457742236-Add-libraries-to-Arduino-IDE).

### Code overview

The purpose of this tutorial is to send messages from the Finder OPTA to an HTTP
server via Ethernet, printing the received response on the serial monitor.

The full code of the example is available
[here](assets/OptaHttpClientTutorial.zip). You can extract the contents of the
`.zip` file and copy it to the ~/Documents/Arduino folder, or alternatively
create a new sketch called `OptaHttpClientTutorial` using the Arduino
IDE and paste the code from the tutorial.

Let's now write the `OptaHttpClientTutorial` sketch, which like all
Arduino sketches will consist of a `setup()` function and a `loop()` function:

```cpp
void setup()
{
  // Setup code, executed at startup
}

void loop()
{
  // Loop code, running infinitely
}
```

At the beginning of our sketch we import the libraries necessary for the
program:

```cpp
#include <Arduino.h>
#include <Ethernet.h>
#include <ArduinoHttpClient.h>

void setup()
{
  // Setup code, executed at startup
}

void loop()
{
  // Loop code, executed infinitely
}
```

In particular we have imported the libraries:

* `Arduino`: contains numerous basic functionalities for Arduino boards, and it
   is therefore good practice to import it at the beginning of all sketches.
* `Ethernet.h`: required to initialize Ethernet connectivity.
* `ArduinoHttpClient.h`: implements the HTTP protocol.

We proceed by configuring an HTTP client, specifying the IP address and port
of the server to which we will send requests:

```cpp
#include <Arduino.h>
#include <Ethernet.h>
#include <ArduinoHttpClient.h>

#define SERVER_IP "192.168.1.1"
#define SERVER_PORT 8080

EthernetClient ethernetClient;
HttpClient httpClient = HttpClient(ethernetClient, SERVER_IP, SERVER_PORT); 

void setup()
{
  // Setup code, executed at startup
}

void loop()
{
  // Loop code, executed infinitely
}
```

The `setup()` function, executed a single time at the startup of Finder OPTA,
will configure the static IP address of Finder OPTA:

```cpp
#include <Arduino.h>
#include <Ethernet.h>
#include <ArduinoHttpClient.h>

#define SERVER_IP "192.168.1.1"
#define SERVER_PORT 8080

EthernetClient ethernetClient;
HttpClient httpClient = HttpClient(ethernetClient, SERVER_IP, SERVER_PORT);

void setup()
{
    IPAddress ip(192, 168, 1, 100);
    Ethernet.begin(ip);
}

void loop()
{
    // Loop code, executed infinitely
}
```

In the `loop()` function, we will send a POST request every 5 seconds, printing
the received response to the serial monitor:

```cpp
#include <Arduino.h>
#include <Ethernet.h>
#include <ArduinoHttpClient.h>

#define SERVER_IP "192.168.1.1"
#define SERVER_PORT 8080

EthernetClient ethernetClient;
HttpClient httpClient = HttpClient(ethernetClient, SERVER_IP, SERVER_PORT);

void setup()
{
    Serial.begin(9600);
    IPAddress ip(192, 168, 1, 100);
    Ethernet.begin(ip);
}

void loop()
{
    httpClient.post("/", "text/plain", "Hello!");
    Serial.println(httpClient.responseBody());
    delay(5000);
}
```

Our sketch will send a POST request every 5 seconds with a `Content-Type`of 
`text/plain` and a body of `Hello!`. On the serial monitor, we will see the
response received from the server printed, in this case, a detail of the request
itself. 

A possible example of the output is as follows:

```text
POST / HTTP/1.1
Host: 192.168.1.1:8080
User-Agent: Arduino/2.2.0
Connection: close
Content-Type: text/plain
Content-Length: 6

Hello!
```

## Conclusion

In this tutorial, we explored how to configure Finder OPTA to send HTTP POST 
requests via an Ethernet connection using a static IP address. This approach 
opens the door to numerous use cases, including remote monitoring, sending data 
to telemetry services, or controlling connected devices via commands 
encapsulated within HTTP requests.
