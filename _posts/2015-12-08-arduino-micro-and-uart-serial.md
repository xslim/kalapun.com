---
title: Arduino Micro and UART serial
---

In Arduino UNO it's very easy to debug a sensor (say GPS). Just connect to pins RX and TX, open a Serial monitor and off you go. 

This is not the case with Arduino Micro, as it shares Serial with USB.

## Hardware serial

So here is a solution:

``` c
void setup() {
  Serial.begin(115200); //This pipes to the serial monitor
  while(!Serial);
  Serial1.begin(9600); //This is the UART, pipes to sensors attached to board
  while(!Serial1);
}

void loop() {
  if ( Serial.available() ) {
    int inByte = Serial.read();
    Serial1.write( inByte );
  }
  if ( Serial1.available() ) {
    int inByte = Serial1.read();
    Serial.write( inByte );  
  }
}
```

Now connect your GPS RX to Micro's TX and GPS TX to RX on Micro, open the Serial monitor and see the data coming in!

## Software serial

If you want to use Software serial, here's example:

``` c
#include <SoftwareSerial.h>

SoftwareSerial mySerial(10, 11); // RX, TX

void setup() {
  Serial.begin(115200); //This pipes to the serial monitor
  while(!Serial);

  mySerial.begin(9600);   
}

void loop() {
  if (mySerial.available()) {
    Serial.write(mySerial.read());
  }
  if (Serial.available()) {
    mySerial.write(Serial.read());
  }
}
```

Now conect sensor RX to pin 11 (TX) and sensor TX to pin 10 (RX)
