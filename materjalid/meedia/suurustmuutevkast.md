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
  myPort = new Serial(this, Serial.list()[0], 9600);  // Vali vajadusel õige port
  myPort.bufferUntil('\n');  // Loeme ühe reavahetuseni korraga
}

void draw() {
  background(255);  // Valge taust

  // Joonista ristkülik ekraani keskele
  rectMode(CENTER);
  fill(0, 150, 255);
  rect(width/2, height/2, rectWidth, rectHeight);
}

void serialEvent(Serial p) {
  String data = p.readStringUntil('\n');
  if (data != null) {
    data = trim(data);  // Eemalda tühikud ja \n
    String[] parts = split(data, ',');
    if (parts.length == 2) {
      int potW = int(parts[0]);
      int potH = int(parts[1]);

      // Kaardista 0–1023 → 10–width või height (piirang joonistamisele)
      rectWidth = int(map(potW, 0, 1023, 10, width - 20));
      rectHeight = int(map(potH, 0, 1023, 10, height - 20));
    }
  }
}

~~~