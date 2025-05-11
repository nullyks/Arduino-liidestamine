### Saatja kood
~~~cpp
#include <SoftwareSerial.h>
#include <ArduinoJson.h> // JSON-andmete loomiseks

SoftwareSerial mySerial(10, 11); // RX, TX (ühendus teise Arduino või muu seadmega)

void setup() {
  mySerial.begin(4800); // Serial-ühenduse kiirus teise seadmega
}

void loop() {
  // Loe analoogväärtused ja teisenda need vahemikku 0–255
  int R = map(analogRead(A0), 0, 1023, 0, 255);
  int G = map(analogRead(A1), 0, 1023, 0, 255);
  int B = map(analogRead(A2), 0, 1023, 0, 255);

  // Loo JSON objekt ja täida andmetega
  StaticJsonDocument<100> doc; // Väike JSON dokument (~50 baiti piisab 3 väärtuse jaoks)
  doc["R"] = R;
  doc["G"] = G;
  doc["B"] = B;

  // Serialiseerib JSON objekti ja saadab tekstina
  serializeJson(doc, mySerial);
  mySerial.println(); // Lisa reavahetus, et saaja saaks sõnumi lõpust aru

  delay(1000); // Väike paus enne uut saatmist
}

~~~

### Vastuvõtja kood

~~~cpp
#include <SoftwareSerial.h>
#include <ArduinoJson.h> // JSON-parsimiseks

#define G 3
#define B 5
#define R 6

SoftwareSerial mySerial(10, 11); // RX (Arduino kuulab siin), TX

void setup() {
  pinMode(R, OUTPUT);
  pinMode(G, OUTPUT);
  pinMode(B, OUTPUT);
  mySerial.begin(4800); // Serial-ühendus teise Arduino'ga
}

void loop() {
  // Kui on saadaval sissetulev sõnum
  if (mySerial.available()) {
    String data = mySerial.readStringUntil('\n'); // Loe kuni reavahetuseni

    StaticJsonDocument<100> doc;
    DeserializationError error = deserializeJson(doc, data); // Proovi parsida JSON

    if (!error) {
      // Kui parsimine õnnestus, loe väärtused ja kasuta
      int r = doc["R"];
      int g = doc["G"];
      int b = doc["B"];

      analogWrite(R, r);
      analogWrite(G, g);
      analogWrite(B, b);
    } else {
      // Vea korral võib siia lisada näiteks vilgutamise või vea logimise
    }
  }
}
~~~