# Keerukamate andmete saatmine kahe Arduino vahel

- [Andmete edastamine jadaliidese kaudu](#andmete-edastamine-jadaliidese-kaudu)
- [JSON-vormingus andmesõnumid](#json-vormingus-andmesõnumid)
- [Andmete edastamine WiFi kaudu](#andmete-edastamine-wifi-kaudu)

Üks digitaaljuhe edastab korraga ühe kahendväärtuse. Teksti või mitut anduriväärtust on mugavam saata jadaliidese või võrgu kaudu. Allpool kasutame iga sõnumi lõpus reavahetust (`\n`), et vastuvõtja teaks, millal üks sõnum lõpeb.

## Andmete edastamine jadaliidese kaudu

UART-jadaliidesega ühendatakse saatja TX vastuvõtja RX-iga, vastuvõtja TX saatja RX-iga ning mõlema plaadi GND-viigud omavahel. Mõlemal poolel peab olema sama edastuskiirus. Enne juhtmete ühendamist lülita toide välja.

UNO R3 plaadil on D0/D1 UART jagatud USB-ühendusega, mistõttu võivad teise seadme juhtmed takistada programmi laadimist või arvutiga suhtlust. Siinses R3 näites kasutame `SoftwareSerial` teeki: `SoftwareSerial(10, 11)` tähendab RX = D10 ja TX = D11. Ühenda ühe plaadi D11 teise D10-ga ja vastupidi. R3 simulatsioonis kujutatud ühendust kontrolli koodis määratud viikude järgi.

UNO R4 WiFi plaadil on USB `Serial` ja D0/D1 `Serial1` eraldi. Ühenda ühe plaadi D1 teise D0-ga ja vastupidi. `SoftwareSerial` näited on mõeldud ainult UNO R3 jaoks.

### Näide: ping ja pong UNO R3-ga

![Kahe Arduino jadaliidese ühendus](meedia/Serial_signaaliedastus.png)

Saatja saadab ühe rea `ping` ja vastuvõtja saadab vastu ühe rea `pong`. Mõlemas programmis peab jadapordi kiirus olema 9600.

Saatja jätab pärast sõnumi saatmist vastuse ootamiseks kuni ühe sekundi. `millis()` annab programmi käivitamisest möödunud aja millisekundites; `break` lõpetab ootesilmuse kohe, kui vastus on saabunud.

**Saatja kood**

```cpp
#include <SoftwareSerial.h>

SoftwareSerial link(10, 11); // RX, TX

void setup() {
  Serial.begin(9600);
  link.begin(9600);
  link.setTimeout(200);
}

void loop() {
  link.println("ping");
  Serial.println("Saadetud: ping");

  unsigned long algus = millis();
  while (millis() - algus < 1000) {
    if (link.available() > 0) {
      String vastus = link.readStringUntil('\n');
      vastus.trim();
      Serial.print("Vastus: ");
      Serial.println(vastus);
      break;
    }
  }
  delay(1000);
}
```

**Vastuvõtja kood**

```cpp
#include <SoftwareSerial.h>

SoftwareSerial link(10, 11); // RX, TX

void setup() {
  Serial.begin(9600);
  link.begin(9600);
  link.setTimeout(200);
}

void loop() {
  if (link.available() > 0) {
    String teade = link.readStringUntil('\n');
    teade.trim();
    if (teade == "ping") {
      link.println("pong");
      Serial.println("Saadetud: pong");
    }
  }
}
```

[UNO R3 interaktiivne simulatsioon](https://www.tinkercad.com/things/aWeu8I6izJ3-ping-pong?sharecode=jb9FeLlcVBtkklh1f_wDyDHqJns84sqinYvRK5oNSmI)

### Sama näide UNO R4 WiFi-ga

Kahe UNO R4 WiFi plaadiga saab teha sama katse: üks plaat saadab sõnumi `ping` ja teine vastab sõnumiga `pong`. Selles näites liiguvad andmed juhtmete kaudu. Plaatide WiFi-ühendust ei kasutata.

**Miks erineb ühendus UNO R3 näitest?**

UNO R3 plaadil kasutavad USB kaudu arvutiga suhtlemine ja viigud D0/D1 sama riistvaralist UART-jadaliidest. Kui ühendada teine Arduino nende viikude külge, võib selle andmevahetus häirida programmi üleslaadimist ja jadamonitori kasutamist. Seepärast lõime R3 näites `SoftwareSerial` teegiga täiendava tarkvaralise jadaliidese viikudele D10 ja D11.

UNO R4 WiFi plaadil on USB kaudu arvutiga suhtlemine ja viikude D0/D1 riistvaraline UART-jadaliides eraldi. Neid kasutatakse programmis kahe erineva objektiga:

* `Serial` suhtleb USB kaudu arvutiga. Selle abil kuvame jadamonitoris saadetud sõnumid ja saabunud vastused.
* `Serial1` suhtleb teise Arduinoga viikude D0 (RX ehk vastuvõtt) ja D1 (TX ehk saatmine) kaudu.

Seetõttu pole R4 näites vaja `SoftwareSerial` teeki ega `link` objekti loomist. `Serial1` on plaadi jaoks juba olemas ning kasutab riistvaralist jadaliidest. Pane tähele, et R3 näite D10/D11 juhtmed tuleb siin ühendada D0/D1 külge. Lisateavet leiab [Arduino UNO R4 WiFi ametlikust juhendist](https://docs.arduino.cc/tutorials/uno-r4-wifi/cheat-sheet/).

**Plaatide ühendamine ja katse käivitamine**

Enne juhtmete ühendamist lülita mõlema plaadi toide välja. Ühenda plaadid järgmiselt:

| Saatja UNO R4 WiFi | Vastuvõtja UNO R4 WiFi |
|---|---|
| D1 (TX) | D0 (RX) |
| D0 (RX) | D1 (TX) |
| GND | GND |

Saatja väljund ühendatakse vastuvõtja sisendiga ja vastupidi, et sõnumid saaksid liikuda mõlemas suunas. Ühine GND annab signaalidele ühise võrdlustaseme.

Vali Arduino IDE-s plaadiks **Arduino UNO R4 WiFi**. Laadi ühele plaadile allolev saatja programm ja teisele vastuvõtja programm. Mõlemas programmis käivitab `Serial1.begin(9600)` plaatidevahelise ühenduse kiirusega 9600 baudi. Ava vastava plaadi USB-pordi jadamonitor ja vali ka seal kiiruseks 9600.

**Saatja kood**

```cpp
void setup() {
  Serial.begin(9600);  // USB-ühendus arvuti jadamonitoriga
  Serial1.begin(9600); // Ühendus teise Arduinoga: RX = D0, TX = D1
  Serial1.setTimeout(200); // Ühe rea lugemise ooteaeg millisekundites
}

void loop() {
  Serial1.println("ping"); // Saadame teisele plaadile ühe rea.
  Serial.println("Saadetud: ping");

  // Ootame vastuse saabumist kuni ühe sekundi.
  unsigned long algus = millis();
  while (millis() - algus < 1000) {
    if (Serial1.available() > 0) {
      String vastus = Serial1.readStringUntil('\n');
      vastus.trim(); // Eemaldame realõpu ja ümbritsevad tühikud.
      Serial.print("Vastus: ");
      Serial.println(vastus);
      break; // Vastus on käes, lõpetame ootamise.
    }
  }
  delay(1000); // Paus enne järgmist sõnumit
}
```

**Vastuvõtja kood**

```cpp
void setup() {
  Serial.begin(9600);  // USB-ühendus arvuti jadamonitoriga
  Serial1.begin(9600); // Ühendus teise Arduinoga: RX = D0, TX = D1
  Serial1.setTimeout(200); // Ühe rea lugemise ooteaeg millisekundites
}

void loop() {
  if (Serial1.available() > 0) {
    String teade = Serial1.readStringUntil('\n');
    teade.trim(); // Eemaldame realõpu ja ümbritsevad tühikud.

    if (teade == "ping") {
      Serial1.println("pong"); // Saadame vastuse esimesele plaadile.
      Serial.println("Saadetud: pong");
    }
  }
}
```

`Serial1.println()` lisab sõnumi lõppu realõpu. `Serial1.readStringUntil('\n')` loeb saabunud märgid kuni reavahetuseni ja `trim()` eemaldab teksti ümbert tühimärgid, sealhulgas alles jäänud `\r` märgi. Nii saab vastuvõtja võrrelda saabunud teksti sõnega `"ping"`.

Saatja jadamonitoris peaksid korduma read `Saadetud: ping` ja `Vastus: pong`. Vastuvõtja jadamonitoris kuvatakse iga vastuse saatmisel `Saadetud: pong`. Kui vastust ei tule, kontrolli esmalt TX- ja RX-juhtmete ristühendust, ühist GND-d ning seda, et mõlemad programmid kasutaksid sama `Serial1` edastuskiirust.

## JSON-vormingus andmesõnumid

JSON (JavaScript Object Notation) lubab ühes sõnumis saata mitu nimega väärtust. Näiteks:

```json
{
  "temperatuur": 21.5,
  "niiskus": 60,
  "valgus": true,
  "asukoht": "elutuba"
}
```

`JsonDocument` hoiab sõnumi välju. Saatja lisab väärtused kujul `doc["temperatuur"] = temperatuur;`; vastuvõtja loeb sama välja näiteks kujul `doc["temperatuur"].as<float>()`. `as<float>()` teisendab väärtuse komakohaga arvuks.

Järgmine UNO R3 näide vajab `ArduinoJson` versiooni 7 ja DHT22-anduri jaoks `DHT sensor library` teeki. Fototakisti peab olema koos sobiva takistiga pingejaguris, mille väljund on ühendatud A0-ga. `valgusePiir` sõltub pingejaguri ühendusest ja valgustingimustest; vajaduse korral muuda võrdlusmärki. Andurite näidiskoodi saab kasutada ka UNO R4 WiFi-ga, kui asendad `SoftwareSerial` eelmises jaotises kirjeldatud viisil `Serial1` objektiga.

**Saatja kood**

```cpp
#include <SoftwareSerial.h>
#include <ArduinoJson.h>
#include <DHT.h>

SoftwareSerial link(10, 11); // RX, TX
DHT dht(2, DHT22);
const int valgusViik = A0;
const int valgusePiir = 500;

void setup() {
  Serial.begin(9600);
  link.begin(9600);
  dht.begin();
}

void loop() {
  float temperatuur = dht.readTemperature();
  float niiskus = dht.readHumidity();

  // Vigase andurinäidu korral ootame enne järgmist mõõtmist.
  if (isnan(temperatuur) || isnan(niiskus)) {
    Serial.println("DHT22 lugemisviga");
    delay(2000);
    return;
  }

  JsonDocument doc;
  doc["temperatuur"] = temperatuur;
  doc["niiskus"] = niiskus;
  int valguseNait = analogRead(valgusViik);
  bool onValgus = valguseNait > valgusePiir;
  doc["valgus"] = onValgus;
  doc["asukoht"] = "elutuba";

  serializeJson(doc, link);
  link.println(); // Üks JSON-sõnum ühel real
  serializeJson(doc, Serial);
  Serial.println();
  delay(2000);
}
```

**Vastuvõtja kood**

```cpp
#include <SoftwareSerial.h>
#include <ArduinoJson.h>

SoftwareSerial link(10, 11); // RX, TX

void setup() {
  Serial.begin(9600);
  link.begin(9600);
  link.setTimeout(300);
}

void loop() {
  if (link.available() > 0) {
    String rida = link.readStringUntil('\n');
    JsonDocument doc;
    DeserializationError viga = deserializeJson(doc, rida);

    if (viga) {
      Serial.print("JSON-i lugemisviga: ");
      Serial.println(viga.c_str());
    } else {
      // Võtame sõnumist iga välja eraldi muutujasse.
      String asukoht = doc["asukoht"].as<String>();
      float temperatuur = doc["temperatuur"].as<float>();
      float niiskus = doc["niiskus"].as<float>();
      bool onValgus = doc["valgus"].as<bool>();

      Serial.print("Asukoht: ");
      Serial.println(asukoht);
      Serial.print("Temperatuur: ");
      Serial.println(temperatuur);
      Serial.print("Niiskus: ");
      Serial.println(niiskus);
      Serial.print("Valgus: ");
      if (onValgus) {
        Serial.println("jah");
      } else {
        Serial.println("ei");
      }
    }
  }
}
```

## Andmete edastamine WiFi kaudu

See näide töötab kahe **UNO R4 WiFi** plaadiga samas WiFi-võrgus. Plaadid kasutavad `WiFiS3` teeki ning TCP-ühendust. Mõne kooli või külalisvõrgu kliendieraldus võib takistada seadmete omavahelist ühendust. Paigalda lisaks `ArduinoJson` versioon 7 ja `DHT sensor library`. Kasuta eespool kirjeldatud DHT22 ning fototakisti ühendust.

Laadi kõigepealt serveri programm ühele plaadile ja vaata selle IP-aadressi jadamonitorist. Asenda kliendi koodis `serverIP` selle aadressiga; näites olev aadress on ainult kohatäitja. Sea mõlemas programmis sama WiFi-võrgu nimi ja parool.

### Serveri kood

```cpp
#include <WiFiS3.h>
#include <ArduinoJson.h>

char ssid[] = "SINUVORK";
char pass[] = "SINUPAROOL";
WiFiServer server(5000);

void setup() {
  Serial.begin(9600);
  WiFi.begin(ssid, pass);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
  }
  Serial.print("Serveri IP: ");
  Serial.println(WiFi.localIP());
  server.begin();
}

void loop() {
  WiFiClient klient = server.available();
  if (klient) {
    klient.setTimeout(1000);
    String rida = klient.readStringUntil('\n');
    JsonDocument doc;
    DeserializationError viga = deserializeJson(doc, rida);

    if (viga) {
      Serial.print("JSON-i lugemisviga: ");
      Serial.println(viga.c_str());
    } else {
      String asukoht = doc["asukoht"].as<String>();
      float temperatuur = doc["temperatuur"].as<float>();
      float niiskus = doc["niiskus"].as<float>();
      bool onValgus = doc["valgus"].as<bool>();

      Serial.print("Asukoht: ");
      Serial.println(asukoht);
      Serial.print("Temperatuur: ");
      Serial.println(temperatuur);
      Serial.print("Niiskus: ");
      Serial.println(niiskus);
      Serial.print("Valgus: ");
      if (onValgus) {
        Serial.println("jah");
      } else {
        Serial.println("ei");
      }
    }
    klient.stop();
  }
}
```

### Kliendi kood

```cpp
#include <WiFiS3.h>
#include <ArduinoJson.h>
#include <DHT.h>

char ssid[] = "SINUVORK";
char pass[] = "SINUPAROOL";
IPAddress serverIP(192, 168, 1, 100); // Asenda serveri tegeliku IP-aadressiga
const uint16_t serveriPort = 5000;
DHT dht(2, DHT22);

void setup() {
  Serial.begin(9600);
  dht.begin();
  WiFi.begin(ssid, pass);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
  }
}

void loop() {
  float temperatuur = dht.readTemperature();
  float niiskus = dht.readHumidity();
  if (isnan(temperatuur) || isnan(niiskus)) {
    Serial.println("DHT22 lugemisviga");
    delay(2000);
    return;
  }

  JsonDocument doc;
  doc["temperatuur"] = temperatuur;
  doc["niiskus"] = niiskus;
  int valguseNait = analogRead(A0);
  bool onValgus = valguseNait > 500;
  doc["valgus"] = onValgus;
  doc["asukoht"] = "elutuba";

  WiFiClient klient;
  if (klient.connect(serverIP, serveriPort)) {
    serializeJson(doc, klient);
    klient.println();
    klient.flush();
    klient.stop();
    Serial.println("Andmed saadetud");
  } else {
    Serial.println("Ühendus serveriga ebaõnnestus");
  }
  delay(2000);
}
```

Lisalugemiseks: [UNO R4 WiFi jadaliides ja WiFi](https://docs.arduino.cc/tutorials/uno-r4-wifi/cheat-sheet/) ning [ArduinoJson 7 üleminekujuhend](https://arduinojson.org/v7/how-to/upgrade-from-v6/).
