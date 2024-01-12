---
title: 'Sending HTTP requests using Finder Opta'
description: 'Learn how to send HTTP requests over Ethernet using Finder Opta.'
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

In this tutorial, we will learn how to send POST requests to an HTTP server
connected to the Finder Opta via Ethernet. In particular, the Finder Opta will
obtain an IP address via DHCP, then it will send a message every 5s, waiting
for a response from the server which will contain the details of the request
sent by the Finder Opta itself. The response will be printed to the serial
monitor.

## Goals

* Learn how to assign a dynamic IP addrress to the Finder Opta.
* Learn how to send HTTP requests over Ethernet using Finder Opta.

## Required Hardware and Software

### Hardware Requirements

* Finder Opta PLC (x1).
* USB-C® cable (x1).
* ETH RJ45 cable (x1).

### Software Requirements

* [Arduino IDE 1.8.10+](https://www.arduino.cc/en/software), [Arduino IDE
2.0+](https://www.arduino.cc/en/software) or [Arduino Web
Editor](https://create.arduino.cc/editor).
* If you choose an offline Arduino IDE, you must install the
`ArduinoHttpClient` library. You can install it using the Library Manager of
the Arduino IDE.
* [Example code](assets/OptaHttpClientTutorial.zip).
* An HTTP server: for the sake of simplicity we recommend something like
  [http-echo-server](https://github.com/watson/http-echo-server) which can be
  easily setup on a computer with a couple commands, and by default it will
  respond to requests with a reply containing an echo of the request itself.

## Finder Opta and the HTTP protocol

Using the `ArduinoHttpClient` library, the Finder Opta can easily generate and
send HTTP requests of POST type.

To support HTTP communication, we must setup a communication channel, in this
case Ethernet: this goal is also easily achieved with the class
`EthernetClient`.

## Instructions

### Setting Up the Arduino IDE

This tutorial will need [the latest version of the Arduino
IDE](https://www.arduino.cc/en/software). If it is your first time setting up
the Finder Opta, check out the [getting started
tutorial](/tutorials/opta/getting-started).

Make sure you install the latest version of the
[`ArduinoHttpClient`](https://www.arduino.cc/reference/en/libraries/arduinohttpclient/)
library, as it will be used to implement the communication with the server.

For further details on how to manually install libraries refer to [this
article](https://support.arduino.cc/hc/en-us/articles/5145457742236-Add-libraries-to-Arduino-IDE).

### Connectivity

The only requirement for this tutorial is that the Finder Opta must be
connected via Ethernet to a device that can assign it a dynamic IP address
using DHCP, and that can later route the packets from the Finder Opta to the
HTTP server and viceversa.

### Code Overview

The goal of the following example is to send messages from the Finder Opta to
an HTTP server via Ethernet, printing on the serial monitor the received
response.

The full code of the example is available
[here](assets/OptaHttpClientTutorial.zip): after extracting the files the
sketch can be compiled and uploaded to the Finder Opta.

#### Setting up the program

At the start of the sketch we will declare the variables needed to setup the
HTTP communication:

```cpp
#include <Arduino.h>
#include <Ethernet.h>
#include <ArduinoHttpClient.h>
#include "opta_info.h"

OptaBoardInfo *info;
OptaBoardInfo *boardInfo();
EthernetClient ethernetClient;
HttpClient httpClient = HttpClient(ethernetClient, "192.168.10.1", 64738); // IP address and port of the HTTP server.
```

In particular the class `OptaBoardInfo` allows to access the MAC address of the
Finder Opta, so that we can perform a DHCP lease, shown in the following
`setup()` code:

```cpp
    info = boardInfo();
    // Check if secure informations are available since MAC Address is among them.
    if (info->magic = 0xB5)
    {
        // Attempt DHCP lease.
        if (Ethernet.begin(info->mac_address) == 0)
        {
            err = true;
        }
    }
    else
    {
        err = true;
    }
```

In case of connectivity issues LED 0 will blink, otherwise the Finder Opta will
receive its dynamic IP address and we will go on with the program execution.

#### Seding POST requests

Below we find the code of the `loop()` function:

```cpp
void loop()
{
    int result = httpClient.post("/", "text/html", "hello there!");
    if (result == 0)
    {
        String response = httpClient.responseBody();
        Serial.println(response);
    }
    else
    {
        Serial.println("fail!");
    }

    delay(5000); // Ping every 5s.
}
```

This function sends every 5s a POST request with `Content-Type` set to
`text/html`, having body `hello there!`. In case the request is successful we
will see a response on the serial monitor, which in the case of the
`http-echo-server` will be a echo of the request itself. A possible output
example is the following:

```text
hello there!
POST / HTTP/1.1
Host: 192.168.10.1:64738
User-Agent: Arduino/2.2.0
Connection: close
Content-Type: text/html
Content-Length: 5
```

## Conclusion

This tutorial shows how to assign a dynamic IP address to the Finder Opta and
then how to use the Ethernet channel to send HTTP requests of POST type to a
sever.
