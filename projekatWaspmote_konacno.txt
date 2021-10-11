int adresa=1024;       //adrese EEPROM-a
int brojac=0;        //brojac koji cemo koristiti za blinkanje LED-a
int a=3;      //pomocna za brojanje blinkovanja LED-a
int b=50;       //pomocna za brojanje blinkovanja LED-a
char *writer;       //pokazivac na char u koji upisujemo vrednost sa RTC-a
char rider;       //vrednost u koju ce biti upisano vreme i datum i koje cemo printovati
char vremeHavarije[30];      //string u koji svlacimo vreme i datum sa RTC-a( koristimo jer ne mozemo upisati uint8_t format u EEPROM!
float sensorValue;      //vrednost iscitana sa MQ-3 senzora

void setup(){
  USB.begin();     //Inicijalizacija UART-a u Waspmote IDE okruzenju
  USB.println("USB port startovan...");
  USB.println("Inicijalizujem RTC..");
  RTC.ON();  //Palimo Real Time Clock
  RTC.setTime("17:06:21:05:15:30:00");   //podesavamo vreme
  USB.println("Zagrevam MQ-3 senzor.");   //cekamo 20000ms da se zagreje MQ-3 seznor
  delay(20000); 
}

void loop(){ // Ispis ocitanih vrednosti temperature i alkohola
    USB.print("Temperatura: ");
    USB.println(RTC.getTemperature(),DEC);   //ispisujemo vrednost temperature ocitane na RTC senzoru
    sensorValue = analogRead(ANALOG4);   //citamo vrednost alkohola ocitanu na MQ-3 senzoru
    USB.print("Alkohol u vazduhu: ");
    USB.print(sensorValue);      //ispisujemo vrednost alkohola ocitanu na MQ-3 senzoru
    USB.println("ppm");
    
    if (RTC.getTemperature() > 30) {   //ukoliko temperatura predje 30stepeni, radimo sledece
           USB.println("PREVISOKA TEMPERATURA! ALARM JE AKTIVIRAN! ");     //javljamo problem preko UARTa
             writer=RTC.getTime();   //uzimamo vreme i datum sa RTC-a
            	for(int i=0; i<30; i++) {   //prolazimo kroz string
                  Utils.writeEEPROM(adresa,writer[i]);    upisujemo datum i vreme iz stringa u EEPROM
                  adresa++;   //inkrementujemo(povecavamo) adresu
                }
           for(int i=0; i<30; i++) {     //prolazimo kroz string
                  rider=Utils.readEEPROM(adresa);     //citamo datum i vreme iz EEPROM-a
                  vremeHavarije[i]=rider;    //upisujemo u string
                  adresa++;      //inkrementujemo(povecavamo) adresu
                }
                USB.print("Upisano vreme havarije: ");
                USB.println(vremeHavarije);     //stampamo string u koji smo upisali datum i vreme
      for(brojac=0; brojac<b; brojac++) {       //blinkanje crvenog LED-a
        Utils.setLED(LED0, LED_ON);       //pali LED
        delay(100);      //ceka 100ms
        Utils.setLED(LED0, LED_OFF);      //gasi LED
        delay(100);     //ceka 100ms
  }
    }
    if (sensorValue > 600) {      //ukoliko koncentracija alkohola predje 600ppm, radimo sledece
      USB.println("Povisen nivo alkohola u vazduhu, moguc izliv supstance! Proverite stanje kontejnera! "); 
              writer=RTC.getTime();     //uzimamo vreme i datum sa RTC-a
            	for(int i=0; i<30; i++){    //prolazimo kroz string
                  Utils.writeEEPROM(adresa, writer[i]);       //citamo datum i vreme iz EEPROM-a
                  adresa++;     //inkrementujemo(povecavamo) adresu
                }
           for(int i=0; i<30; i++){    //prolazimo kroz string
                  rider=Utils.readEEPROM(adresa);    //citamo podatak sa adrese
                  vremeHavarije[i]=rider;    //iscitani string upisujemo u promenjivu
                  adresa++;    //povecavamo adresu
                }
                USB.print("Tacno vreme havarije: ");  
                USB.println(vremeHavarije);     //ispisujemo vreme i datum detektovanja
      for(brojac=0; brojac<b; brojac++) {    //blinkanje crvenog LED-a
        Utils.setLED(LED0, LED_ON);    //pali LED
        delay(100);    //ceka 100ms
        Utils.setLED(LED0, LED_OFF);    //gasi LED
        delay(100);
    }
   }
   for(brojac =0; brojac<a; brojac++) {    //blinkanje zelenog LED-a
    Utils.setLED(LED1, LED_ON);
    delay(500);
    Utils.setLED(LED1, LED_OFF);
    delay(500);
  }
 }
