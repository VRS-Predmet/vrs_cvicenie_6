# Náplň cvičenia
- zoznámenie sa s I2C
- komunikácia s IMU prostredníctvom I2C
- predstavenie zadania s I2C senzorom

# I2C zbernica

### Základné posielanie dát
<p align="center">
    <img src="https://github.com/VRS-Predmet/vrs_cvicenie_9/blob/master/docs/I2C_Basic_Address_and_Data_Frames.jpg" width="950" title="">
</p>

### Posielanie väčšieho množstva dát
<p align="center">
    <img src="https://github.com/VRS-Predmet/vrs_cvicenie_9/blob/master/docs/I2C_Repeated_Start_Conditions.jpg" width="550" title="">
</p>

# Konfigurácia I2C

<p align="center">
    <img src="https://github.com/VRS-Predmet/vrs_cvicenie_9/blob/master/images/i2c_conf.PNG" width="850" title="GPIO pin block scheme">
</p>

- na obrázku je zobrazená konfigurácia I2C - okrem zobrazených parametrov je nutné povoliť globálne prerušenia od I2C ("error interrupt" nie je momentálne potrebný)
- MCU má rolu "master", číta z registrov alebo zapisuje do registrov senzora ("slave") 


# IKS01A2 senzorová doska

<p align="center">
    <img src="https://github.com/VRS-Predmet/vrs_cvicenie_9/blob/master/images/sensor_board.jpg" width="350" title="GPIO pin block scheme">
</p>

- obsahuje viacero senzorových jednotiek: acc + magnetometer (LSM303AGR), IMU - acc + gyro (LSM6DSL), sonzor vlhkosti a teploty (HTS221), barometrický snímač (LPS22HB) ...
- v tomto prípade MCU komunikuje s IMU (acc a gyro) LSM6DSL
- na obrázku je znázornené zapojenie senzorovej dosky k vývojovej doske s MCU

- ukážkový program komunikuje s LSM6DSL prostredníctvom I2C a číta hodnotu zrýchlení v osiach x, y, z

# Serial oscilloscope
<p align="center">
    <img src="https://github.com/VRS-Predmet/vrs_cvicenie_9/blob/master/images/serial_osc.png" width="950" title="Serial oscilloscope">
</p>

- aplíkácia pre vykreslovanie dát, ktoré sú priaté prostredníctvom sériovej komunikácie vo formáte CSV (comma-separated values) - napríklad "123,158,39.789,5639"
- možnosť používať aj ako obyčajnú terminálovú aplikáciu (Putty)
- nutnosť nastaviť COM port a prenosovú rýchlosť (baud rate)
- po kliknutí na "Osciloscope" za otvorí nové okno s grafickým zobrazením priebehov (maximálne 9 kanálov, 3 kanály pre jedno okno), s ktorým je možné manipulovať (posúvať priebeh, meniť časovú základňu a rozsah zobrazovaných hodnôt)
- k dispozícii na stiahnutie tu: https://x-io.co.uk/serial-oscilloscope/  

# Zadanie 5 (10b)

Vytvoriť aplikáciu, ktorá bude posielať dáta získané zo senzorov (LPS22HB, HTS221) cez USART do PC. Zobrazovanými údajmi budú teplota, relatívna vlhkosť vzduchu, tlak vzduchu a relatívna výška od zeme.

### Úlohy

1. Vytvorenie vlastného git repozitára - viditeľné commity, nastavenie ".gitignore".

2. Vytvorenie funkcii v zdrojovom subore "i2c.c" pre čítanie a zápis dát po I2C zbernici s možnosťou single aj multi byte prenosu. Ak hodnota registra pozostava z viacerych bytov, musi sa pouzit multibyte prenos a byt vycitana naraz (nie postupne po bytoch vo viacerych citaniach). Funckie z "i2c.c/h" nesmu byt priamo pouzivane v kniznici senzora ale musia byt registrovane ako "callback funkcie" (oddelenie konfiguracie MCU od konfiguracie senzora a jeho aplikacie).

3. Vytvoriť si vlastnú knižnicu pre senzory LPS22HB a HTS221
   -  Zdrojovy priecinok - v "Project explorer" -> pravy click "Vas project" -> "new" -> "Source folder" (nazov priecinku podla senzora) -> pravy click na novy priecinok a "Add/remove include path ..." .
   - V precinku senzora vytvorit zdrojovy a hlavickovy subor pre dany senzor.
   - V hlavickovom subore sa budu nachadzat makra pre adresy registrou senzora a pripadne hodnoty danych registrov. Deklaracie funkcii pre pracu so senzorom.
   - V zdrojovom subore sa budu nachadzat funkcie pre inicializaciu, vycitavanie a zapisovanie dat, spracovanie vycitanych dat, ziskanie hodnoty meranej veliciny.
     
4. Knižnica musí obsahovať inicializačnú funkciu, ktorej úlohou je overiť pripojenie senzora a vykonať počiatočnú konfiguráciu senzora. Ako prvé overíte, či viete prečítať "WHO_AM_I" register a či hodnota, ktorú vám senzor vráti je totožná s hodnotou z dokumentácie. Následne zapíšete do registrov sonzora svoju vlastnú konfiguráciu. Prejdite si dokumentáciu, zistite čo všetko viete nastavovať pomocou registrov a podľa potreby si zvoľte vlastnú konfiguráciu senzora (frekvencia merania, merací rozsah ... ).

5. Knižnica musí obsahovať funkciu na čítanie/zapisovanie dáť zo/do senzora. Zapisovanie do senzora bude slúžiť napr. pri konfigurácii senzora a čítanie bude slúžiť na získavanie aktuálneho stavu senzora (ak je to potrebné), hodnôt meraných veličín ... . Ak zo senzora budete čítať viac ako jednu veličinu (napr. teplota a vlhkosť), tak pre každú meranú veličinu vytvorte samostatnú funkciu na jej čítanie.

6. Údaje, ktoré sa majú zobrazovať ale nie su priamo získateľné zo senzora je potrebné na základe dostupných meraných hodnôt vypočítať (napr. relatívna výška od zeme, ...). Pre takýto "post processing", kedy sa z meraných údajov snažite niečo vypočítať vytvorte samostatnú funkciu.

7. Údaje sa budú posielať cez USART vo formáte CSV a zobrazovať na PC pomocou "serial osciloscope".

8. Formátovanie odosielaných hodnôt. Hranatá zatvorka predstavuje v akých jednotkách je zobrazovaná hodnota. Niektoré hodnoty sú zobrazované s presnosťou na 1 alebo 2 desatinné miesta. "xx.x" predstavuje digity vyhradené pre číslice - jedno desatinné miesto, jednotky a desiatky:
   - teplota [°C]: "xx.x"
   - rel. vlhkosť [%]: "xx"
   - tlak vzduchu [hPa]: "xxxx.xx"
   - relatívna výška od zeme [m]: "xxx.xx"
   
9. Odovzdáva sa projekt osobne na začiatku 8. cvičenia + počas odovzdávania sa skontroluje aj git.
