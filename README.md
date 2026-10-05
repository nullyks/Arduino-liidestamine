# Arduino liidestamine teiste seadmetega

Õppematerjal selgitab, kuidas vahetada andmeid kahe Arduino arendusplaadi vahel ning ühendada Arduino arvutis töötava rakendusega. Näited käsitlevad lihtsat digitaalsignaali, jadaliidest, JSON-vormingut, WiFi-ühendust ja Processingut.

## Kasutatavad arendusplaadid

Kursusel kasutatakse Arduino UNO R3 ja Arduino UNO R4 WiFi arendusplaate. Ühendusskeemid ja Tinkercadi simulatsioonid on koostatud UNO R3 jaoks. UNO R4 WiFi kasutamisel järgi iga näite juures toodud plaadipõhiseid juhiseid.

| Teema | UNO R3 | UNO R4 WiFi |
|---|---|---|
| Arvutiga suhtlus | `Serial` USB kaudu; D0/D1 on sama riistvaralise UART-iga seotud | `Serial` USB kaudu |
| Teise plaadiga suhtlus | Näidetes `SoftwareSerial` viikudel D10/D11 | `Serial1` viikudel D0/D1 |
| WiFi-näited | Vajavad eraldi WiFi-riistvara | Sisseehitatud WiFi ja teek `WiFiS3` |

Mõlema plaadi sisendite ja väljundite loogikatase on 5 V. Kahe plaadi juhtmega ühendamisel ühenda ka nende GND-viigud. Eri pingetasemega seadmete ühendamisel kontrolli enne ühendamist pingetasemete sobivust.

## Õpieesmärgid

Materjali läbimisel õpid valima andmete edastamiseks sobiva viisi, saatma andmeid kahe Arduino vahel ning suhtlema arvutirakendusega USB-jadaliidese kaudu.

## Õpiväljundid

Materjali edukalt läbinud õppija oskab:

* edastada lihtsat digitaalsignaali kahe Arduino vahel;
* valida kahe Arduino vahel andmete saatmiseks sobiva ühenduse;
* koostada ja lugeda JSON-vormingus andmesõnumeid;
* saata ja vastu võtta andmeid Arduino ning arvutirakenduse vahel.

## Hindamisjuhend

Selle õppematerjali puhul ei rakendata eristavat hindamist. Õppija on materjali omandanud, kui ta suudab iseseisvalt lahendada vähemalt neli viiest ülesandest. Lahenduse juurde kuuluvad ühendusjoonis ja kommenteeritud programmikood.

## Vajalikud vahendid

* 1 personaalarvuti (Windows, Linux või macOS)
* 2 Arduino UNO R3 või UNO R4 WiFi arendusplaati ning neile sobivad USB-andmesidekaablid
* 1 makettplaat ja makettplaadi juhtmed
* 3 potentsiomeetrit ja 1 nupp
* 2 ühevärvilist LED-i ja 1 RGB-LED ning iga kasutatava LED-kanali jaoks sobiv voolupiirav takisti (näiteks 470 Ω)
* 1 DHT22 temperatuuri- ja niiskusandur ning 1 fototakisti koos sobiva pingejaguri takistiga JSON-näidete jaoks
* 1 PIR-liikumisandur ja 1 aktiivsummer või sobiva juhtskeemiga passiivsummer ülesannete jaoks
* 1 alalisvoolumootor, sellele sobiv mootoridraiver ja vajaduse korral eraldi toiteallikas ventilaatoriülesande jaoks
* tarkvarateegid `ArduinoJson` (versioon 7) ja `DHT sensor library` koos sõltuvustega andurinäidete jaoks; UNO R4 WiFi puhul kuulub `WiFiS3` plaadipaketti

Mootorit ei tohi ühendada otse Arduino sisend-väljundviiguga. Vali draiver ja toide mootori andmesildi järgi ning ühenda draiveri ja Arduino GND ühiseks.

## Õppematerjali osad

* [Lihtsate signaalide saatmine kahe Arduino vahel](materjalid/1_lihtsad_signaalid.md)
* [Keerukamate signaalide saatmine kahe Arduino vahel](materjalid/2_keerukamad_signaalid.md)
* [Arduino liidestamine personaalarvutiga](materjalid/3_liidestamine_personaalarvutiga.md)
* [Iseseisvad ülesanded](materjalid/4_iseseisvad_ülesanded.md)

## Õppematerjali koostajad

Tanel Toova (tanel.toova@tlu.ee)
