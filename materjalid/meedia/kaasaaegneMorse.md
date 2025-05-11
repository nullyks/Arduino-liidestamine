### Saatja kood
~~~cpp
#include <WiFiS3.h>         // Arduino UNO R4 WiFi võrgumooduli teek
#include <WiFiUdp.h>        // UDP suhtluseks
#include <ArduinoJson.h>    // JSON andmestruktuuride jaoks

const char* ssid = "SINU_WIFI";          // WiFi võrgu nimi (SSID)
const char* password = "SINU_SALASONA";  // WiFi parool

const int buttonPin = 2;     // Nupp ühendatud digitaalsisendile 2
bool buttonPressed = false;  // Kas nuppu hoitakse all?
unsigned long pressStart = 0; // Nupu vajutamise algusaeg

WiFiUDP udp;
IPAddress receiverIP(192, 168, 0, 2);   // Vastuvõtva UNO IP-aadress
unsigned int receiverPort = 4210;        // UDP port, kuhu saata

void setup() {
  pinMode(buttonPin, INPUT_PULLUP);   // Nupp tõmmatakse üles sisemise takistiga
  Serial.begin(9600);                 // Serial-monitori jaoks
  WiFi.begin(ssid, password);         // WiFi ühenduse loomine

  // Ootame, kuni ühendus on loodud
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  Serial.println("\nWiFi ühendatud, IP: " + WiFi.localIP().toString());
}

void loop() {
  int buttonState = digitalRead(buttonPin);

  // Vajutuse algus
  if (buttonState == LOW && !buttonPressed) {
    pressStart = millis();
    buttonPressed = true;
  }

  // Nupu vabastamine → arvutame kestuse ja saadame
  if (buttonState == HIGH && buttonPressed) {
    unsigned long duration = millis() - pressStart;
    buttonPressed = false;

    // Koostame JSON objekti
    StaticJsonDocument<128> doc;
    doc["type"] = "press_duration";      // Tüüp (võimalik hiljem laiendada)
    doc["duration_ms"] = duration;       // Kestus millisekundites

    char buffer[128];
    serializeJson(doc, buffer);          // Teisendame JSON stringiks

    // Saadame UDP paketina
    udp.beginPacket(receiverIP, receiverPort);
    udp.write((const uint8_t*)buffer, strlen(buffer));
    udp.endPacket();

    Serial.print("Saadetud JSON: ");
    Serial.println(buffer);
  }
}

~~~


### Vastuvõtja kood
~~~cpp
#include <WiFiS3.h>         // Arduino UNO R4 WiFi teek
#include <WiFiUdp.h>        // UDP protokolli kasutamiseks
#include <ArduinoJson.h>    // JSON andmete vastuvõtmiseks

IPAddress local_IP(192, 168, 0, 2);     // Soovitud IP
IPAddress gateway(192, 168, 0, 1);      // Ruuteri IP (tavaliselt .1)
IPAddress subnet(255, 255, 255, 0);     // Võrgumask

const char* ssid = "SINU_WIFI";          // WiFi võrgu nimi
const char* password = "SINU_SALASONA";  // WiFi parool

const int buzzerPin = 3;     // Summer ühendatud klemmile 3 (digitaalne väljund)

WiFiUDP udp;
unsigned int localPort = 4210;     // Sama port, kuhu esimene UNO saadab
char packetBuffer[256];            // Vastuvõetud UDP paketi puhver

void setup() {
  pinMode(buzzerPin, OUTPUT);   // Seame summeri väljundiks
  Serial.begin(9600);

  // Määrame staatilise IP
  if (!WiFi.config(local_IP, gateway, subnet)) {
    Serial.println("IP seadistamine ebaõnnestus");
  }

  WiFi.begin(ssid, password);   // Ühendume WiFi-võrguga
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  udp.begin(localPort);         // Alustame UDP kuulamist määratud pordil
  Serial.println("\nWiFi ühendatud, IP: " + WiFi.localIP().toString());
}

void loop() {
  int packetSize = udp.parsePacket();   // Kas saabus UDP pakett?
  if (packetSize) {
    int len = udp.read(packetBuffer, sizeof(packetBuffer) - 1);
    if (len > 0) {
      packetBuffer[len] = '\0';         // Null-lõpetame stringi

      StaticJsonDocument<256> doc;
      DeserializationError error = deserializeJson(doc, packetBuffer);

      if (!error) {
        // Kontrollime, kas sõnum on õiget tüüpi
        if (doc["type"] == "press_duration") {
          unsigned long duration = doc["duration_ms"];  // Loeme kestuse
          Serial.print("Saadi kestus (ms): ");
          Serial.println(duration);

          // Mängime heli
          digitalWrite(buzzerPin, HIGH);
          delay(duration);  // Helisignaali kestus
          digitalWrite(buzzerPin, LOW);
        }
      } else {
        Serial.print("JSON viga: ");
        Serial.println(error.c_str());
      }
    }
  }
}

~~~