---
title: 'Invio di richieste HTTP con Finder Opta'
description: "Imparare ad inviare tramite Ethernet richieste HTTP con Finder
              Opta."
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

## Panoramica

In questo tutorial, impareremo ad inviare richieste POST ad un server HTTP
connesso a Finder Opta tramite Ethernet. In particolare, Finder Opta riceverà
un indirizzo IP tramite DHCP, ed in seguito invierà ogni 5 secondi un
messaggio, a cui seguirà una risposta del server contente i dettagli della
richiesta inviata dal Finder Opta stesso: tale riposta verrà stampata su
monitor seriale.

## Obiettivi

* Imparare ad assegnare un IP dinamico a Finder Opta.
* Imparare ad inviare richieste HTTP su canale Ethernet con Finder Opta.

## Requisiti hardware e software

### Requisiti hardware

* PLC Finder Opta (x1).
* Cavo USB-C® (x1).
* Cavo ETH RJ45 (x1).

### Requisiti Software

* [Arduino IDE 1.8.10+](https://www.arduino.cc/en/software), [Arduino IDE
2.0+](https://www.arduino.cc/en/software) o [Arduino Web
Editor](https://create.arduino.cc/editor).
* Se si utilizza Arduino IDE offline, è necessario installare la libreria
  `ArduinoHttpClient` utilizzando il Library Manager dell'Arduino IDE.
* [Codice di esempio](assets/OptaHttpClientTutorial.zip).
* Un server HTTP: per semplicità è consigliato
  [http-echo-server](https://github.com/watson/http-echo-server), che può
  essere facilmente configurato su un computer con pochi comandi, e di default
  risponde alle richieste con una riposta contente un echo della richiesta
  stessa.

## Finder Opta e il protocollo HTTP

Grazie alla libreria `ArduinoHttpClient`, Finder Opta può facilmente generare
ed inviare richieste HTTP di tipo POST.

A supporto dell protocollo HTTP, è necessario configurare un canale di
comunicazione, in questo caso Ethernet: anche questo passaggio è semplice,
grazie alla classe `EthernetClient`.

## Istruzioni

### Configurazione dell'Arduino IDE

Per seguire questo tutorial, sarà necessaria [l'ultima versione dell'Arduino
IDE](https://www.arduino.cc/en/software). Se è la prima volta che configuri il
Finder Opta, dai un'occhiata al tutorial [Getting Started with
Opta](/tutorials/opta/getting-started).

Assicurati di installare l'ultima versione della libreria
[`ArduinoHttpClient`](https://www.arduino.cc/reference/en/libraries/arduinohttpclient/)
poiché verrà utilizzata per implementare la comunicazione con il server.

Per ulteriori dettagli su come installare manualmente le librerie, consulta
[questo
articolo](https://support.arduino.cc/hc/en-us/articles/5145457742236-Add-libraries-to-Arduino-IDE).

### Connettività

L'unico requisito di questo tutorial è che Finder Opta sia connesso tramite
Ethernet ad un dispositivo in grado di fornire un indirizzo IP dinamico tramite
DHCP, ed in seguito ad instradare i pacchetti dal Finder Opta al server HTTP e
viceversa.

### Panoramica del codice

Lo scopo del seguente esempio è inviare messaggi da Finder Opta ad un server
HTTP tramite Ethernet, stampando la risposta ricevuta su monitor seriale.

Il codice completo dell'esempio è disponibile
[qui](assets/OptaHttpClientTutorial.zip): dopo aver estratto i file, lo sketch
può essere compilato e caricato sul Finder Opta.

#### Setup del programma

Nella parte iniziale dello sketch dichiareremo le variabili necessarie a
stabilire una comunicazione HTTP:

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

In particolare la classe `OptaBoardInfo` permette di accedere al MAC address di
Finder Opta, in modo da effettuare un lease DHCP, mostrato nel seguente codice
di `setup()`:

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

In caso di problemi di connettività il LED 0 lampeggerà, altrimenti il nostro
Finder Opta disporrà di un indirizzo IP dinamico e potremo procedere con
l'esecuzione del programma.

#### Invio di richieste POST

Di seguito troviamo il codice della funzione `loop()`:

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

Questa funzione invia ogni 5 secondi una richiesta POST con `Content-Type` di
tipo `text/html`, avente body `hello there!`: in caso di successo vedremo la
risposta stampata su monitor seriale, che nel caso di `http-echo-server` sarà
un echo della richiesta stessa. Un possibile esempio di output è il seguente:

```text
hello there!
POST / HTTP/1.1
Host: 192.168.10.1:64738
User-Agent: Arduino/2.2.0
Connection: close
Content-Type: text/html
Content-Length: 5
```

## Conclusioni

Questo tutorial mostra come assegnare un indirizzo IP dinamico a Finder Opta,
ed in seguito utilizzare il canale Ethernet per inviare richieste HTTP di tipo
POST ad un server.
