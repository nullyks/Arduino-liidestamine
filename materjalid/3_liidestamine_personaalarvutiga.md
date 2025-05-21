# Arduino Uno liidestamine personaalarvutiga üle jadaliidese

- [Lihtne andmevahetus arvuti ja Arduino vahel](#lihtne-andmevahetus-arvuti-ja-arduino-vahel)
- [Arduino liidestamine Processing arenduskeskkonnaga](#arduino-liestamine-processing-arenduskeskkonnaga)
    - [Näide: sündmus Processingust Arduinosse](#näide-sündmus-processingust-arduinosse)
        - [Arduino kood](#arduino-kood)
        - [Processing kood](#processing-kood)
    - [Näide: andmed Arduinost Processingusse](#näide-andmed-arduinost-processingusse)
      - [Arduino kood](#arduino-kood-1)
      - [Processing kood](#processing-kood-1)

Arduino Uno suhtleb arvutiga eelkõige jadaliidese (ingl *serial*) ühenduse kaudu, kasutades füüsilise meediumina USB-kaablit. See ühendus võimaldab andmevahetust Arduino plaadi ja arvutis töötava rakenduse, näiteks Arduino IDE või mõne muu jadaliidese monitori vahel. Jadaliidese ühendust kasutatakse programmide üleslaadimiseks arendusplaadile, aga ka Arduino saadetud andmete lugemiseks ja plaadile käskude saatmiseks. See on oluline nii arenduse, testimise kui ka seadme hilisema juhtimise seisukohalt.

Arvuti ja Arduino ühendamisel USB kaabliga saab Arduino Uno voolu otse USB-pordi kaudu, mistõttu eraldi toiteallikat pole vaja lihtsate katsetuste puhul. Kui ühendus on loodud, peaks arvuti automaatselt tuvastama Arduino plaadi ja määrama sellele virtuaalse COM-pordi. Arduino IDE suudab seejärel tuvastada ühendatud plaadi ja sobiva pordi.

## Lihtne andmevahetus arvuti ja Arduino vahel
Arduino IDE-s saab Serial ühendust kasutada kahes peamises kontekstis. Esiteks toimub programmide üleslaadimine plaadile just selle ühenduse kaudu. Teiseks saab Serial Monitori või Serial Plotteri abil vaadelda Arduino poolt saadetud andmeid (nt andurite väärtused, veateated vms). Selleks tuleb programmis kasutada *Serial.begin()* funktsiooni, mis määrab baudimäära ehk andmeedastuskiiruse (nt *Serial.begin(9600);*), ja *Serial.print()* või *Serial.println()* funktsioone andmete saatmiseks.

Lisaks andmete väljastamisele võimaldab jadaliidese ühendus ka andmete saatmist Arduino arendusplaadile. Näiteks saab Serial Monitori kaudu sisestada väärtusi või käske, mida Arduino suudab *Serial.read()*, *Serial.readString()* või *Serial.parseInt()* abil vastu võtta ja töödelda. See võimaldab kasutajatel reaalajas seadme käitumist mõjutada ning luua interaktiivseid süsteeme ilma lisanuppude või liidesteta.

Vaatame järgnevalt lihtsat näidet, kus Arduino arendusplaadil töötav rakkendus küsib üle jadaliidese kasutaja nime ja kui kasutaja on selle arvutis sisestanud, siis tervitab kasutajat nimepidi.
![Nime küsimise näide](meedia/Nimenäide.png)
~~~cpp

void setup()
{
  Serial.begin(9600); //Alustame jadaliidese ühendusega
  Serial.println("Mis su nimi on?"); //Saadame arvuti pooole teate küsimusega 
}

void loop()
{
  if(Serial.available()) { //kui arvuti poolt on plaadile saadetud andmeid
  	Serial.print("Tere "); //saadame arvuti poole "Tere " ilma reavahetuseta.
    Serial.print(Serial.readString()); //saadame arvuti poole tagasi sealt saadetud andmed, mida käsitleme kui stringi (eeldame, et see on kasutaja nimi)
    Serial.println("!"); //Saadame arvuti poole "!" ja reavahetussümboli.
  }
}
~~~
Jadaühendusega seotud funktsioonide kohta saad täpsemalt [lugeda siit](https://docs.arduino.cc/language-reference/en/functions/communication/serial/)

## Arduino liidestamine Processing arenduskeskkonnaga

Sageli on tarvilik töödelda Arduino ja sellega liidestatud andurite abil saadud andmeid reaalajas või juhtida Arduino tööd läbi graafilise kasutajaliidese. 
Arduino endal kipub sellise funktsionaalsuse jaoks jõudlust väheks jääma. Lahenduseks on taaskord vahetada andmeid arvutiga, pannes oluliselt suurema jõudlusega seadme täitma funktsioone, mida Arduino ise sooritada ei jõua.

Üheks sageli kasutatavaks lahenduseks on [Processing](https://processing.org/) arenduskeskkond, mille saab paigaldadda arvutisse (töötab nii Windowsis, Linuxis kui macOS-is).

Processing on avatud lähtekoodiga programmeerimiskeel ja arenduskeskkond, mis loodi visuaalse kunsti ja disaini õpetamiseks ning interaktiivsete rakenduste loomiseks. Seda kasutatakse laialdaselt visuaalsete ja interaktiivsete projektide, generatiivse kunsti, andmete visualiseerimise ning loovprogrammeerimise õpetamiseks. Processing põhineb Java programmeerimiskeelel, kuid selle süntaks on lihtsustatud, et muuta see algajatele kergemini arusaadavaks.

Processing ja Arduino suhtlevad omavahel jadaliidese kaudu. Arduino saadab andmeid USB-kaabli kaudu arvutisse, kus Processing saab neid vastu võtta ja töödelda – näiteks kuvada reaalajas andurite näitu. Samuti pakkuda graafilist kasutajaliidest Arduinol põhineva seadme juhtimiseks. See võimaldab luua interaktiivseid süsteeme, mis ühendavad füüsilise maailma ja arvutiekraani.
## Näide: sündmus Processingust Arduinosse
Näiteks võime joonistada Processingus nupu mille vajutamise peale lülitab Arduino LED-i sisse ja välja.
### Arduino kood
~~~cpp
#define ledPin 13 // LED ühendatud digitaalsele pin 13

void setup() {
  pinMode(ledPin, OUTPUT);
  Serial.begin(9600); // Alustame Serial ühenduse
}

void loop() {
  if (Serial.available()) { //kui Serial ühenduse peale on saadetud andmeid
    char command = Serial.read(); // Loeme ühe märgi
    if (command == '1') { 
      digitalWrite(ledPin, HIGH); // Lülita LED sisse
    } else if (command == '0') {
      digitalWrite(ledPin, LOW); // Lülita LED välja
    }
  }
}
~~~

### Processing kood
~~~java
import processing.serial.*;

// Serial-ühenduse objekt
Serial myPort;

// Muutuja, mis hoiab LEDi olekut (true = põleb, false = kustus)
boolean ledState = false;

void setup() {
  size(300, 200); // Määrame akna suuruseks 300x200 pikslit
  printArray(Serial.list()); // Prindime saadaolevad Serial-pordid (nt COM3 jne)
  
  // Avame ühenduse esimese saadaoleva Serial-pordiga kiirusel 9600 baudi
  // Vajadusel muuda Serial.list()[0] sobivaks (nt Serial.list()[1])
  // Praegu eeldame, et esimene port sobib
  myPort = new Serial(this, Serial.list()[0], 9600);
}

void draw() {
  background(240); // Helehall taust

  // Joonistame toggle-nupu
  // Kui LED on sees, siis roheline nupp, muidu punane
  fill(ledState ? color(0, 200, 0) : color(200, 0, 0));
  rect(100, 70, 100, 60, 10); // x=100, y=70, laius=100, kõrgus=60, ümardus=10px

  // Tekst nupule
  fill(255); // Valge tekst
  textAlign(CENTER, CENTER);
  textSize(16);
  text(ledState ? "LED ON" : "LED OFF", 150, 100); // Tekst nupu keskel
}

// See funktsioon käivitub iga kord, kui kasutaja klikib hiirega
void mousePressed() {
  // Kontrollime, kas klikk oli nupu alas (koordinaadid vastavad nupule)
  if (mouseX >= 100 && mouseX <= 200 && mouseY >= 70 && mouseY <= 130) {
    ledState = !ledState; // Vaheta LEDi olekut (true <-> false)
    
    // Saadame Arduinole ühe märgi: '1' kui LED sisse, '0' kui välja
    myPort.write(ledState ? '1' : '0');
  }
}
~~~

## Näide: andmed Arduinost Processingusse
Nüüd vaatame keerukamat näidet, kus DHT22 anduri andmete abil joonistatakse õhutemperatuuri ja -niiskuse graafik.

### Arduino kood
~~~cpp
#include "DHT.h"

DHT dht(2, DHT22);

void setup() {
  Serial.begin(9600);
  dht.begin();
}

void loop() {
  float humidity = dht.readHumidity();
  float temperature = dht.readTemperature();

  // Kontrollime, kas väärtused on kehtivad
  if (isnan(humidity) || isnan(temperature)) {
    Serial.println("N/A,N/A");
  } else { //kui väärtused on kehtivad kirjutame need jadaühenduse peale välja eraldatuna komaga
    Serial.print(temperature);
    Serial.print(",");
    Serial.println(humidity);
  }

  delay(1000); // Saadame andmeid kord sekundis
}
~~~

### Processing kood
~~~java
// Importime Serial teegi, mis võimaldab suhelda Arduinoga
import processing.serial.*;

// Serial objekt
Serial myPort;

// Serialist loetav rida
String inString;

// Andmete hoidmiseks loome listid (dünaamilised massiivid)
ArrayList<Float> tempData = new ArrayList<Float>();
ArrayList<Float> humData = new ArrayList<Float>();

// Maksimaalne punktide arv, mida graafikul korraga näitame
int maxDataPoints = 300;

void setup() {
  size(600, 400); // Akna suurus (laius x kõrgus)
  printArray(Serial.list()); // Prindib välja kõik saadaolevad COM-pordid

  // Loome Serial ühenduse teise pordiga loendis (olenevalt arvutist võib olla vaja muuta)
  myPort = new Serial(this, Serial.list()[1], 9600);

  // Määrame, et Serial loeb kuni uue reani '\n'
  myPort.bufferUntil('\n');
}

void draw() {
  background(255); // Valge taust

  // Joonistame kaks eraldi graafikut – temperatuuri ja niiskuse
  drawGraph(tempData, color(255, 0, 0), "Temperatuur (°C)", 20);
  drawGraph(humData, color(0, 0, 255), "Õhuniiskus (%)", height/2 + 20);
}

// Funktsioon ühe graafiku joonistamiseks
void drawGraph(ArrayList<Float> data, int col, String label, float yStart) {
  stroke(col); // Määrame joone värvi
  noFill();
  beginShape(); // Alustame joonistamist

  for (int i = 0; i < data.size(); i++) {
    // Teisendame andmeväärtuse y-koordinaadiks (graafiku kõrguse piires)
    float yVal = map(data.get(i), 0, 100, height / 2, 0);
    // Joonistame jooni andmepunktide vahel
    vertex(i * (width / float(maxDataPoints)), yStart + yVal);
  }

  endShape(); // Lõpetame joonistamise

  // Näitame hetke viimast väärtust tekstina
  fill(0);
  text(label + ": " + (data.size() > 0 ? nf(data.get(data.size()-1), 1, 1) : "…"), 10, yStart - 5);
}

// Serial sündmus – käivitatakse iga kord, kui uus rida on saadaval
void serialEvent(Serial p) {
  inString = trim(p.readStringUntil('\n')); // Loeme ja puhastame rea

  // Kontrollime, kas andmestring sisaldab koma – eeldame formaati "23.5,45.2"
  if (inString != null && inString.contains(",")) {
    String[] values = split(inString, ",");
    if (values.length == 2) {
      try {
        // Teisendame stringid ujukomaarvudeks
        float t = float(values[0]);
        float h = float(values[1]);

        // Lisame andmed listi ja eemaldame vanimad, kui neid on liiga palju
        tempData.add(t);
        humData.add(h);
        if (tempData.size() > maxDataPoints) tempData.remove(0);
        if (humData.size() > maxDataPoints) humData.remove(0);
      } catch (Exception e) {
        println("Andmeviga: " + inString); // Väljastame veateate, kui teisendus ebaõnnestub
      }
    }
  }
}
~~~
