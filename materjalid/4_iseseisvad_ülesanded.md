# Iseseisvad ülesanded

Iga ülesande lahenduse juurde lisa ühendusjoonis ja kommenteeritud kood. Enne kahe plaadi juhtmega ühendamist lülita toide välja ning ühenda ka GND-viigud. UNO R3 jaoks mõeldud Tinkercadi näited ei pruugi kujutada UNO R4 WiFi erisusi.

## 1. Alarm

Ehita kahe Arduino UNO, PIR-liikumisanduri ja aktiivsummeriga alarm. PIR on ühe plaadi, summer teise küljes. Liikumise tuvastamisel annab teine plaat helisignaali. Vali summeri juhtskeem selle voolutarbe järgi.

[UNO R3 näidislahendus Tinkercadis](https://www.tinkercad.com/things/dUG47nIvkQw-alarm?sharecode=JJag4QnCHDe9HKRtAbpsH-j5zrdGKm9BzvGwASghZ44)

## 2. Värvivalija

Ehita kahe Arduino UNO, RGB-LED-i ja kolme potentsiomeetriga seade. Iga potentsiomeeter määrab ühe värvikanali heleduse. Paigalda RGB-LED-i igale kanalile voolupiirav takisti ning arvesta, kas LED-il on ühine anood või ühine katood.

[UNO R3 näidislahendus ilma JSON-ita](https://www.tinkercad.com/things/lNQ0qwEMnPZ-varvivalija?sharecode=JFCJ7t9JcX8cWCKqZBTOaufWxw40wfnbUTa97Aod0Ec) · [Näidiskood JSON-iga](meedia/värvivalija_JSON.md)

## 3. Kuulekas ventilaator

Ehita Arduino UNO, alalisvoolumootori ja mootoridraiveriga ventilaator. Võta arvuti jadaliidese kaudu vastu käsud `start`, `stopp`, `kiiremini` ja `aeglasemalt`. `start` seab PWM-i töötsükli 50%-le; kiiruse muutmise käsud muudavad seda 10 protsendipunkti kaupa vahemikus 0–100%. `stopp` peatab mootori. Mootorit ei ühendata otse Arduino viiguga; vali draiver ning toide mootori andmete järgi.

[UNO R3 näidislahendus Tinkercadis](https://www.tinkercad.com/things/aaTSHRr008n-kuulekas-ventilaator?sharecode=alywwVQjurgwfiuHY3e4iVdKW8BbrgWaX0BBD_IXNrk)

## 4. Kaasaegne morse

Ehita kahe **UNO R4 WiFi** plaadi, nupu ja aktiivsummeriga süsteem. Esimene plaat mõõdab nupu allhoidmise kestuse ja saadab selle WiFi kaudu teisele plaadile; teine annab sama pika helisignaali. Mõlemad plaadid peavad saama samasse WiFi-võrku. Lisa nupule tarkvaraline või riistvaraline võnkumise summutus, et üks vajutus ei tekitaks mitut sõnumit.

[Näidiskood](meedia/kaasaaegneMorse.md)

## 5. Suurust muutev kast

Loo **ühe** Arduino UNO, kahe potentsiomeetri ja Processinguga süsteem. Potentsiomeetrid määravad Processingu aknas joonistatava ristküliku laiuse ja kõrguse.

[Näidiskood](meedia/suurustmuutevkast.md)
