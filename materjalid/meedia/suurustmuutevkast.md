# Suurust muutva kasti näidiskood

Ühenda potentsiomeetrite keskmised klemmid viikudega A0 ja A1 ning äärmised klemmid 5 V ja GND-ga. UNO R4 WiFi puhul on `analogRead()` vaikimisi samuti 10-bitine (0–1023). Sulge Arduino IDE jadamonitor enne Processingu käivitamist. Vali Processingu konsoolis kuvatud portide loendist õige indeks.

### Arduino kood
~~~cpp
const int potWidthPin = A0;   // Laiust määrav potentsiomeeter
const int potHeightPin = A1;  // Kõrgust määrav potentsiomeeter

void setup() {
  Serial.begin(9600);         // Serial ühendus Processingule
}

void loop() {
  int widthVal = analogRead(potWidthPin);    // 0–1023
  int heightVal = analogRead(potHeightPin);  // 0–1023

  // Saadame väärtused komadega eraldatuna ühel real (nt "400,750")
  Serial.print(widthVal);
  Serial.print(",");
  Serial.println(heightVal);

  delay(50);  // Vähendab koormust
}

~~~

### Processing kood
~~~java
import processing.serial.*;

Serial myPort;
int rectWidth = 100;
int rectHeight = 100;

void setup() {
  size(600, 600);
  printArray(Serial.list());  // Prindi saadaolevad pordid konsooli
  myPort = new Serial(this, Serial.list()[0], 9600);  // Asenda indeks õige pordi järgi
  myPort.bufferUntil('\n');  // Loeme ühe reavahetuseni korraga
}

void draw() {
  background(255);  // Valge taust

  // Joonista ristkülik ekraani keskele
  rectMode(CENTER);
  fill(0, 150, 255);
  int keskpunktX = width / 2;
  int keskpunktY = height / 2;
  rect(keskpunktX, keskpunktY, rectWidth, rectHeight);
}

void serialEvent(Serial p) {
  String data = p.readStringUntil('\n');
  if (data != null) {
    data = trim(data);  // Eemalda tühikud ja \n
    String[] parts = split(data, ',');
    if (parts.length == 2) {
      int saadudLaius = int(parts[0]);
      int saadudKõrgus = int(parts[1]);
      int piiratudLaius = constrain(saadudLaius, 0, 1023);
      int piiratudKõrgus = constrain(saadudKõrgus, 0, 1023);

      // Kaardista potentsiomeetrite näidud akna mõõtmeteks.
      float uusLaius = map(piiratudLaius, 0, 1023, 10, width - 20);
      float uusKõrgus = map(piiratudKõrgus, 0, 1023, 10, height - 20);
      rectWidth = int(uusLaius);
      rectHeight = int(uusKõrgus);
    }
  }
}

~~~
