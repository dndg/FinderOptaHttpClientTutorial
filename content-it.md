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

In questo tutorial impareremo a inviare richieste POST tramite Ethernet
utilizzando Finder Opta. Grazie alla connettività Ethernet integrata, Finder
Opta può comunicare facilmente con un server remoto tramite il protocollo HTTP.
Configureremo un indirizzo IP statico per Finder Opta e successivamente
invieremo richieste POST a un server HTTP specifico. La risposta del server
verrà catturata e visualizzata sul monitor seriale, permettendoci di monitorare
l'intera interazione in tempo reale.

Gli use case più comuni per l'invio di richieste HTTP su dispositivi embedded
includono la comunicazione con servizi web per monitoraggio, controllo remoto o
logging di dati. Ad esempio, Finder Opta può inviare richieste ad un server per
ricevere una configurazione o per inviare dati di telemetria.

## Requisiti

### Hardware

* PLC Finder Opta (x1).
* Cavo USB-C® (x1).
* Cavo ETH RJ45 (x1).

### Software

* [Arduino IDE 2.0+](https://www.arduino.cc/en/software) o [Arduino Web
Editor](https://create.arduino.cc/editor).
* Se si utilizza Arduino IDE offline, è necessario installare la libreria
  `ArduinoHttpClient` utilizzando il Library Manager dell'Arduino IDE.
* [Codice di esempio](assets/OptaHttpClientTutorial.zip).
* Un server HTTP: per semplicità consigliamo
  [http-echo-server](https://github.com/watson/http-echo-server), un server
  facile da configurare su qualsiasi computer con pochi comandi. Questo server
  risponde alle richieste restituendo un "echo" del contenuto inviato,
  rendendolo ideale per testare la comunicazione HTTP.

### Connettività

Per seguire questo tutorial, Finder Opta deve essere connesso tramite Ethernet
ad un dispositivo in grado di instradare i pacchetti da Finder Opta al server
HTTP, e viceversa.

## Finder Opta e il protocollo HTTP

Grazie alla libreria `ArduinoHttpClient`, Finder Opta può generare e inviare
facilmente richieste HTTP di tipo POST.

Per consentire la comunicazione tramite il protocollo HTTP, è necessario
configurare una connessione di rete: in questo caso utilizziamo la classe
`EthernetClient`.

## Istruzioni

### Configurazione dell'Arduino IDE

Per seguire questo tutorial, sarà necessaria [l'ultima versione dell'Arduino
IDE](https://www.arduino.cc/en/software). Se è la prima volta che configuri un
Finder Opta, dai un'occhiata al tutorial [Getting Started with
Opta](/tutorials/opta/getting-started): in questo tutorial spieghiamo come
installare il Board Manager per la piattaforma Mbed OS Opta, ovvero l'insieme
di tool di base necessari a creare e utilizzare uno sketch per Finder Opta con
Arduino IDE.

Assicurati di installare l'ultima versione della libreria
[`ArduinoHttpClient`](https://www.arduino.cc/reference/en/libraries/arduinohttpclient/)
poiché verrà utilizzata per implementare la comunicazione con il server.

Per una breve spiegazione su come installare manualmente le librerie
all'interno di Arduino IDE, consulta [questo
articolo](https://support.arduino.cc/hc/en-us/articles/5145457742236-Add-libraries-to-Arduino-IDE).

### Panoramica del codice

Lo scopo di questo tutorial è inviare messaggi da Finder Opta ad un server HTTP
tramite Ethernet, stampando la risposta ricevuta su monitor seriale.

Il codice completo dell'esempio è disponibile
[qui](assets/OptaHttpClientTutorial.zip). È possibile estrarre il contenuto del
file `.zip` e copiarlo nella cartella ~/Documents/Arduino, o alternativamente
creare un nuovo sketch chiamato `OptaHttpClientTutorial` utilizzando Arduino
IDE ed incollare il codice presente nel tutorial.

Passiamo ora a scrivere lo sketch `OptaHttpClientTutorial`, che come tutti gli
sketch per Arduino sarà composto da una funzione di `setup()` e una funzione
`loop()`:

```cpp
void setup()
{
  // Codice di setup, eseguito all'avvio
}

void loop()
{
  // Codice di loop, eseguito all'infinito
}
```

All'inizio del nostro sketch importiamo le librerie necessarie al funzionamento
del programma:

```cpp
#include <Arduino.h>
#include <Ethernet.h>
#include <ArduinoHttpClient.h>

void setup()
{
  // Codice di setup, eseguito all'avvio
}

void loop()
{
  // Codice di loop, eseguito all'infinito
}
```

In particolare abbiamo importato le librerie:

* `Arduino`: contiene numerose funzionalità di base per le schede Arduino, ed è
  quindi buona norma importarla all'inizio di tutti gli sketch.
* `Ethernet.h`: necessaria a inizializzare la connettività Ethernet.
* `ArduinoHttpClient.h`: implementa il protocollo HTTP.

Procediamo configurando un client HTTP, specificando l'indirizzo IP e la porta
del server al quale invieremo le richieste:

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
    // Codice di setup, eseguito all'avvio
}

void loop()
{
    // Codice di loop, eseguito all'infinito
}
```

La funzione `setup()`, eseguita una singola volta all'avvio di Finder Opta,
dovrà occuparsi di configurare l'indirizzo IP statico di Finder Opta:

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
    // Codice di loop, eseguito all'infinito
}
```

Nella funzione `loop()` invieremo una richiesta POST ogni 5 secondi, stampando
la risposta ricevuta su monitor seriale:

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

Il nostro sketch invierà ogni 5 secondi una richiesta POST con `Content-Type`
di tipo `text/plain` e body `Hello!`. Su monitor seriale vedremo stampata la
risposta ricevuta dal server, in questo caso un dettaglio della richiesta
stessa. Un possibile esempio di output è il seguente:

```text
POST / HTTP/1.1
Host: 192.168.1.1:8080
User-Agent: Arduino/2.2.0
Connection: close
Content-Type: text/plain
Content-Length: 6

Hello!
```

## Conclusioni

In questo tutorial abbiamo esplorato come configurare Finder Opta per inviare
richieste HTTP POST tramite una connessione Ethernet utilizzando un indirizzo
IP statico.

Questo approccio apre la strada a numerosi casi d'uso, tra cui il monitoraggio
remoto, l'invio di dati a servizi di telemetria o il controllo di dispositivi
connessi tramite comandi incapsulati all'interno di richieste HTTP.
