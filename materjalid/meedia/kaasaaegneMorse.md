# Kaasaegse morse näidiskood (UNO R4 WiFi)

Kasuta kaht UNO R4 WiFi plaati samas WiFi-võrgus. Paigalda ArduinoJson 7, ühenda nupp viigu D2 ja GND vahele ning kasuta sobiva juhtskeemiga aktiivsummerit. Käivita esmalt vastuvõtja ja asenda saatja koodis `serverIP` vastuvõtja tegeliku IP-aadressiga. Kood piirab ühe helisignaali pikkuse 5 sekundiga; nupu väga kiire vajutamine võib vajada eraldi võnkumise summutust.

### Saatja kood
~~~cpp
// CLIENT – UNO R4 WiFi (TCP)
// Vajab: WiFiS3, ArduinoJson
#include <WiFiS3.h>
#include <ArduinoJson.h>

char ssid[] = "SINUVÕRK";
char pass[] = "SINUPAROOL";

const int nupuViik = 2;
bool nuppOliVajutatud = false;
unsigned long vajutuseAlgus = 0;

// >>> ASENDA  IP-ga, mida server Serial näitas <<<
IPAddress serverIP(192, 168, 1, 100);  // ASENDA vastuvõtja IP-aadressiga
const uint16_t serverPort = 6000;

void setup() {
  pinMode(nupuViik, INPUT_PULLUP);

  Serial.begin(9600);
  WiFi.begin(ssid, pass);
  Serial.print("Ühendan WiFi-ga");
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("\nWiFi OK");

  delay(5000); //Anname DHCP-le aega
  Serial.print("Kliendi IP: ");
  Serial.println(WiFi.localIP());
  Serial.print("Siht: ");
  Serial.print(serverIP);
  Serial.print(":");
  Serial.println(serverPort);
}

void loop() {
  int nupuOlek = digitalRead(nupuViik);

  // INPUT_PULLUP korral tähendab LOW vajutatud nuppu.
  if (nupuOlek == LOW && nuppOliVajutatud == false) {
    vajutuseAlgus = millis();
    nuppOliVajutatud = true;
  }

  // Kui nupp vabastatakse, saadame vajutuse kestuse.
  if (nupuOlek == HIGH && nuppOliVajutatud == true) {
    unsigned long kestus = millis() - vajutuseAlgus;
    nuppOliVajutatud = false;

    JsonDocument doc;
    doc["type"] = "press_duration";
    doc["duration_ms"] = kestus;

    WiFiClient client;
    if (client.connect(serverIP, serverPort)) {
      // Saadame ühe rea JSON-i + '\n', et serveri readStringUntil('\n') lõpetaks
      serializeJson(doc, client);
      client.println();
      client.flush();
      client.stop();

      Serial.print("Saadetud kestus (ms): ");
      Serial.println(kestus);
    } else {
      Serial.println("Ühendus serveriga ebaõnnestus.");
    }
  }
}

~~~


### Vastuvõtja kood
~~~cpp
// SERVER – UNO R4 WiFi (TCP)
// Vajab: WiFiS3, ArduinoJson
#include <WiFiS3.h>
#include <ArduinoJson.h>

char ssid[] = "SINUVÕRK";
char pass[] = "SINUPAROOL";

const uint16_t PORT = 6000;
const int summeriViik = 3;

WiFiServer server(PORT);

void setup() {
  pinMode(summeriViik, OUTPUT);
  digitalWrite(summeriViik, LOW);

  Serial.begin(9600);
  WiFi.begin(ssid, pass);
  Serial.print("Ühendan WiFi-ga");
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("\nWiFi OK");
  delay(3000); // anna DHCP-le veidi aega

  Serial.print("IP: ");
  Serial.println(WiFi.localIP()); // selle IP kopeeri saatjale

  server.begin();
  Serial.print("TCP server kuulab pordil ");
  Serial.println(PORT);
}

void loop() {
  WiFiClient client = server.available();
  if (client) {
    Serial.println("Klient ühendas.");

    // Loeme ühe rea (kliendi sõnum lõpeb reavahetusega).
    client.setTimeout(1000);
    String rida = client.readStringUntil('\n');

    JsonDocument doc;
    DeserializationError viga = deserializeJson(doc, rida);

    if (viga) {
      Serial.print("JSON-i lugemisviga: ");
      Serial.println(viga.c_str());
    } else {
      String teateTyyp = doc["type"].as<String>();
      if (teateTyyp == "press_duration") {
        unsigned long kestus = doc["duration_ms"].as<unsigned long>();
        if (kestus > 5000) {
          kestus = 5000;
        }

        Serial.print("Saadi kestus (ms): ");
        Serial.println(kestus);

        digitalWrite(summeriViik, HIGH);
        delay(kestus);
        digitalWrite(summeriViik, LOW);
      } else {
        Serial.print("Tundmatu teate tüüp: ");
        Serial.println(teateTyyp);
      }
    }
    client.stop();
  }
}
~~~
