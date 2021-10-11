# waspmote_project1
Senzor požara i koncentracije alkohola u industriji.

1.	Uvod


Senzor požara i koncentracije alkohola u industriji
Ovaj senzor se koristi u industriji alkohola npr. vinarija, pivara, kao i ostalih fabrika za proizvodnju proizvoda koji sadrže alkohol. 
Program za očitavanje vrednosti  temperature i koncentracije alkohola sa RTC senzora temperature i MQ-3 senzora gasa se upisuje u memoriju Waspmote v1.1 razvojnog sistema i izvršava na istom.

2.	Analiza problema


Glavni cilj jeste konstantno očitavanje vrednosti temperature i koncentracije alkohola i dati nam doznanje o situaciji u prostoriji fabrike proizvodnje ili skladištu.
Ukoliko senzor detektuje povišenu vrednost u odnosu na uobičajenu, potrebno je upaliti alarm u vidu ispisa poruke upozorenja, blinkati crvenom LED diodom i upisati datum i vreme u EEPROM memoriju.
Za normalna očitavanja će se regularno ispisivati vrednost temperature i koncentracije alkohola i blinkati zelenom LED diodom.

3.	Povezivanje mikrokontrolera sa PC računarom


Povezivanje vršimo putem mini USB porta na razvojnom sistemu
Pre povezivanja potrebno je instalirati potrebne drajvere za komunikaciju putem COM porta.

4.	Detaljan opis svih komponenata uređaja


Za ovaj problem koristimo Waspmote razvojni sistem.
Waspmote je „open-source“ bežična senzorska platforma osmišljena radi implementacije senzorskih čvorova niske potrošnje u IoT platformi. Platforma je kompletno autonomna i napaja se iz baterijskog izvora...
Poseduje mogućnost više tipova komunikacije, kao što su GPRS, GSM, WiFi, BlueTooth... 
Tehnička specifikacija (obavezno proučiti pre povezivanja waspmote-a):
Minimalni radni napon povezanog baterijskog  izvora: 3V3
Maksimalni radni napon povezanog baterijskog izvora: 4V2
USB napon punjenja baterije: 5V
Maksimalna struja na USB konektoru: 480 mA
Apsolutne maksimalne vrednosti:
Napon n a bilo kom I/O pinu:  [-0.5, +3V8]
Maksimalna struja za bilo koji I/O pin: 40 mA
USB napajanje: 7V
Napon napunjene baterije: 4V2

Kopmonente koje se koriste u ovom programu su RTC senzor i MQ-3 senzor gasa.
RTC(Real Time Clock) je elektronski sklop najčešće u vidu integrisanog kola koji meri vreme. RTC je prisutan u gotovo svim elektronskim uređajima koji imaju potrebu da prikazuju tačno vreme. U ovom programu RTC ima ulogu da ispiše datum i vreme očitanih vrednosti. RTC senzor se takodje koristi za očitavanje temperature.
Karakteristike RTC-a:
- realizovan kao zaseban sklop koji vodi računa isključivo o funkciji merenja vremena, njegov klok se ne opterećuje raznim prekidima i izvršavanjem programabilnog koda kao što to čini standardni mikrokontroler.
- ima zaseban baterijski izvor koji mu omogućuje da i nakon prekida napajanja ostatka sklopa i dalje nastavi sa radom
- izuzetno niska potrošnja, koja obezbređuje da uz novčić bateriju uređaj bude napajan i u periodu od nekoliko godina
- poseduje precizni eksterni oscilator 32,768 kHz koji se koristi u  quartz satovima.
Pre preuzimanja podataka sa integrisanog kola RTC, mora se inicijalizovati I2C magistala, koja se koristi za komunikaciju.
MQ-3 senzor gasa
MQ-3 senzor gasa je Metal Oxide Semiconductor (MOS) tip senzora koji se u ovom slučaju koristi za očitavanje koncentracije alkohola u vazduhu i ispisivanje iste na UART. Princip rada je baziran na promeni otpornosti. Termin koncenracija gasa se odnosi na opisivanje zapreminske količine gasa u vazduhu. Objašnjenje za ppm: koncenracija jednog gasa (alkohol) u odnosu na drugi (vazduh). Na primer, 500 ppm znači da od milion molekula gasa, 500 molekula je alkohol, a 999500 molekula tog drugog gasa (vazduh). Takodje obezbeđuje analogni naponski signal proporcionalan koncentraciji alkohola koji se konvertuje i prosledjuje na UART.
Korišćenje MQ-3 senzora gasa:
Vrednost sa senzora očitavamo na sledeći način:
void setup() {
	USB.begin();     //setujemo UART
	USB.println("MQ3 zagrevanje!");
	delay(20000);    }   //sačekajmo da se senzor dovoljno zagreje   
void loop() {
	sensorValue = analogRead(ANALOG4);    //čitamo analogni unos sa pina 0
	USB.print(“Vrednost ocitana sa senzora: ");
	USB.println(sensorValue);  }
Upis i čitanje EEPROM-a:
Primer upisivanja sadržaja u određenu lokaciju:
{
	Utils.WriteEEPROM(1024, ‘B’);  // upisivanje ascii karaktera ‘B’ u adresu 1024
}
Primer iščitavanja sadržaja iz određene lokacije:
{
	uint8_t podatak = Utils.readEEPROM(1024); // iščitava podatak sa adrese 1024
}
Nakon svakog upisa kritične vrednosti očitane sa senzora, inkrementujemo adresu kako ne bismo overvrajtovali staru vrednost. Takođe je bitno da se ne koriste adrese niže od 512 jer može overvrajtovati konfiguarioni deo memorije mikrokontrolera!
Rad sa LED diodama:
Paljenje i gašenje zelene diode izvršavamo sledećim kodom:
        Utils.setLED(LED1, LED_ON);
        Utils.setLED(LED1, LED_OFF);
Paljenje i gašenje crvene diode izvršavamo sledećim kodom:

        Utils.setLED(LED0, LED_ON);
        Utils.setLED(LED0, LED_OFF);
       
Ukoliko želimo simulirati blinkanje, to činimo ubacivanjem for petlje i delay-ova u kod. 

5.	Koncept rešenja


Da bi se pristupilo zadatku, prva stvar koju treba uraditi je povezati eksterne senzore na mikrokontroler. U našem slučaju RTC je već integrisan, dok MQ-3 senzor gasa povezujemo kao što je u opisu komponenata prikazano. Nakon toga počinjemo sa ispisom koda u razvojnom okruženju Wasp IDE koji će se izvršavati na našem mikrokontroleru. Kod pišemo u programu za upravljanje Waspmote mikrokontrolerom, kompajlujemo i spuštamo na isti.
Kod se sastoji iz setup() i void () zaglavlja. U setup-u inicijalizujemo komunikaciju sa PC računarom pomoću USB porta koje nam omogućuje spuštanje koda na mikrokontroler. Kada se USB koristi za komunikaciju preko UART0, čip FT232RL radi konverziju na USB standard. Pored inicijalizacije komunikacije preko USB-a, palimo i sve senzore koje koristimo, u ovom slučaju palimo i RTC.
Nakon inicijalizacije svih komponenti, potrebno je definisati i promenljive koje ćemo koristiti za izradu koda. Naime želimo da na svake 3 sekunde bude ispisan rezultat sa stanjem temperature i koncentracije alkohola i dok su vrednosti uobičajene, blinkaće zelena LED dioda. U suprotnom, upalićemo alarm u vidu ispisa na UART, datum i vreme detektovanja ćemo upisati u EEPROM i ispisati, takođe ćemo upaliti brzo blinkanje crvenog LED-a.
Kod programa se može naći na kraju dokumentacije.

7.	Zaključak


Konstantno merenje ključnih vrednosti u preduzećima navedenih u uvodu nam omogućuje veću kontrolu situacije. Samim tim možemo i predvideti havariju ukoliko vidimo postepeno povećavanje merenih vrednosti i sprečiti potencijalnu. Mikrokontroler sa navedenim senzorima i upisanim programom bi se instalirao u prostorijjama tipa sale za proizvodnju, skladišta i ostalih gde se radi sa zapaljivom supstancom.

8.	Literatura


Literatura korišćena za izradu dokumentacije je sadržana u prezentacijama laboratorijskih vežbi.
