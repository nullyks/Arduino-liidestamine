# Värvivalija näidiskood (UNO R3)

Ühenda kahe plaadi GND-viigud, saatja D11 vastuvõtja D10-ga ja vastuvõtja D11 saatja D10-ga. Näide eeldab ühise katoodiga RGB-LED-i, mille iga värvikanal on ühendatud sobiva voolupiirava takisti kaudu PWM-viiguga. Ühise anoodiga LED-i puhul tuleb PWM-väärtused pöörata (`255 - väärtus`). Mõlemale plaadile tuleb paigaldada ArduinoJson 7.

### Saatja kood
~~~cpp
#include <SoftwareSerial.h>
#include <ArduinoJson.h> // JSON-andmete loomiseks

SoftwareSerial mySerial(10, 11); // RX, TX (ühendus teise Arduino või muu seadmega)

void setup() {
  mySerial.begin(4800); // Serial-ühenduse kiirus teise seadmega
}

void loop() {
  // Kõigepealt loeme kolm potentsiomeetrit.
  int punaseNait = analogRead(A0);
  int roheliseNait = analogRead(A1);
  int siniseNait = analogRead(A2);

  // Teisendame näidud PWM-i vahemikku 0–255.
  int punane = map(punaseNait, 0, 1023, 0, 255);
  int roheline = map(roheliseNait, 0, 1023, 0, 255);
  int sinine = map(siniseNait, 0, 1023, 0, 255);

  // Loo JSON objekt ja täida andmetega
  JsonDocument doc;
  doc["R"] = punane;
  doc["G"] = roheline;
  doc["B"] = sinine;

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

const int rohelineViik = 3;
const int sinineViik = 5;
const int punaneViik = 6;

SoftwareSerial mySerial(10, 11); // RX (Arduino kuulab siin), TX

void setup() {
  pinMode(punaneViik, OUTPUT);
  pinMode(rohelineViik, OUTPUT);
  pinMode(sinineViik, OUTPUT);
  Serial.begin(9600); // Veateated arvuti jadamonitorile
  mySerial.begin(4800); // Serial-ühendus teise Arduino'ga
  mySerial.setTimeout(300);
}

void loop() {
  // Kui on saadaval sissetulev sõnum
  if (mySerial.available() > 0) {
    String data = mySerial.readStringUntil('\n'); // Loe kuni reavahetuseni

    JsonDocument doc;
    DeserializationError error = deserializeJson(doc, data); // Proovi parsida JSON

    if (error) {
      Serial.print("JSON-i lugemisviga: ");
      Serial.println(error.c_str());
    } else {
      int saadudPunane = doc["R"].as<int>();
      int saadudRoheline = doc["G"].as<int>();
      int saadudSinine = doc["B"].as<int>();

      int punane = constrain(saadudPunane, 0, 255);
      int roheline = constrain(saadudRoheline, 0, 255);
      int sinine = constrain(saadudSinine, 0, 255);

      analogWrite(punaneViik, punane);
      analogWrite(rohelineViik, roheline);
      analogWrite(sinineViik, sinine);
    }
  }
}
~~~
