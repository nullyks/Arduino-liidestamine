### Saatja kood
~~~cpp
// CLIENT – UNO R4 WiFi (TCP)
// Vajab: WiFiS3, ArduinoJson
#include <WiFiS3.h>
#include <ArduinoJson.h>

char ssid[] = "SINUVÕRK";
char pass[] = "SINUPAROOL";

const int BUTTON_PIN = 2;     // kasutame INPUT_PULLUP-i
bool buttonPressed = false;
unsigned long pressStart = 0;

// >>> ASENDA  IP-ga, mida server Serial näitas <<<
IPAddress serverIP(10, 167, 75, 167);  // NÄIDE
const uint16_t serverPort = 6000;

WiFiClient client;

void setup() {
  pinMode(BUTTON_PIN, INPUT_PULLUP);

  Serial.begin(9600);
  while (!Serial) {}

  WiFi.begin(ssid, pass);
  Serial.print("Ühendan WiFi-ga");
  while (WiFi.status() != WL_CONNECTED) { delay(500); Serial.print("."); }
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
  int state = digitalRead(BUTTON_PIN);

  // vajutuse algus (aktiivne LOW)
  if (state == LOW && !buttonPressed) {
    pressStart = millis();
    buttonPressed = true;
  }

  // vabastus → arvuta kestus ja saada
  if (state == HIGH && buttonPressed) {
    unsigned long duration = millis() - pressStart;
    buttonPressed = false;

    StaticJsonDocument<128> doc;
    doc["type"] = "press_duration";
    doc["duration_ms"] = duration;
    Serial.println(duration); //kontroll, et saime nupuvajutuse aja

    if (client.connect(serverIP, serverPort)) {
      // Saadame ühe rea JSON-i + '\n', et serveri readStringUntil('\n') lõpetaks
      serializeJson(doc, client);
      client.println();
      client.flush();
      client.stop();

      Serial.print("Saadetud kestus (ms): ");
      Serial.println(duration);
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
const int BUZZER_PIN = 3;

WiFiServer server(PORT);

void setup() {
  pinMode(BUZZER_PIN, OUTPUT);
  digitalWrite(BUZZER_PIN, LOW);

  Serial.begin(9600);
  while (!Serial) {}

  WiFi.begin(ssid, pass);
  Serial.print("Ühendan WiFi-ga");
  while (WiFi.status() != WL_CONNECTED) { delay(500); Serial.print("."); }
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
  if (!client) return;

  Serial.println("Klient ühendus.");

  // Loeme ühe rea (kliendi lõpus peab olema '\n')
  String line = client.readStringUntil('\n');

  StaticJsonDocument<256> doc;
  DeserializationError err = deserializeJson(doc, line);

  if (err) {
    Serial.print("JSON viga: ");
    Serial.println(err.c_str());
  } else {
    const char* type = doc["type"] | "";
    if (strcmp(type, "press_duration") == 0) {
      unsigned long duration = doc["duration_ms"] | 0UL;
      Serial.print("Saadi kestus (ms): ");
      Serial.println(duration);

      digitalWrite(BUZZER_PIN, HIGH);
      delay(duration);
      digitalWrite(BUZZER_PIN, LOW);
    } else {
      Serial.print("Tundmatu type: ");
      Serial.println(type);
    }
  }

  client.stop();
  Serial.println("Klient katkestas.\n");
}
~~~
