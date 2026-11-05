# Nexys A7 – Hőmérséklet- és gyorsulásmérő szenzorok UART-kapcsolattal

## 1. A projekt célja

A projekt célja egy FPGA-alapú mérőrendszer megvalósítása a Digilent Nexys A7 fejlesztőpanelen.

A rendszer feladatai:

- Az ADT7420 digitális hőmérséklet-érzékelő adatainak kiolvasása I²C kommunikációval.
- Az ADXL362 gyorsulásmérő adatainak kiolvasása SPI kommunikációval.
- A mért adatok feldolgozása és szöveges formátumba alakítása.
- Az eredmények továbbítása UART soros kommunikáción keresztül számítógépre.
- A rendszer működésének és hibáinak jelzése a fejlesztőpanel LED-jeivel.

A fejlesztés Verilog nyelven, Xilinx Vivado 2022.2 környezetben történik.

## 2. A jelenlegi program felépítése

A projekt több különálló modulból áll.

| Modul                | Feladat                                        |
| -------------------- | ---------------------------------------------- |
| `top.v`              | Fő vezérlőmodul, a részegységek összehangolása |
| `adt7420_reader.v`   | Hőmérséklet-érzékelő kezelése I²C-n            |
| `adxl362_reader.v`   | Gyorsulásmérő kezelése SPI-n                   |
| `sensor_formatter.v` | A mérési adatok szöveges formázása             |
| `uart_tx.v`          | Adatok továbbítása UART-on                     |

A `top.v` egy állapotgéppel vezérli a mérési folyamatot:

1. Várakozás a mintavételi időpontra.
2. Hőmérséklet kiolvasása.
3. Gyorsulásadatok kiolvasása.
4. Az adatok formázása.
5. Az eredmények továbbítása UART-on.

A mintavételezés jelenleg körülbelül másodpercenként történik.

## 3. Mit sikerült eddig megvalósítani?

Sikeres lépések

- A Vivado projekt létrehozása és szintetizálása.
- A Nexys A7 panelhez szükséges lábkiosztások beállítása az XDC fájlban.
- A bitstream sikeres előállítása.
- A számítógép és az FPGA közötti JTAG-kapcsolat létrehozása.
- Az FPGA programozása sikeresen megtörtént.
- A rendszer órajele működik, amit a LED0 villogása is igazol.
- Az ADT7420 I²C-olvasómodulját átdolgoztuk, különös tekintettel az I²C adatátvitelre és az ACK/NACK kezelésre.
- Az új modullal a LED1 bekapcsol, ami a fő vezérlő szerint sikeres hőmérsékletolvasást jelez.

## 4. A jelenlegi LED-ek jelentése

A legutóbbi programozás után a következő állapot látható:

| LED  | Funkció                                   | Jelenlegi állapot                                                            |
| ---- | ----------------------------------------- | ---------------------------------------------------------------------------- |
| LED0 | FPGA órajelének működését jelző heartbeat | Gyorsan villog                                                               |
| LED1 | Sikeres hőmérsékletolvasás                | Folyamatosan világít                                                         |
| LED2 | Sikeres gyorsulásadat-olvasás             | Nem világít                                                                  |
| LED3 | Formázott adat rendelkezésre áll          | Nem világít                                                                  |
| LED4 | Időtúllépés jelzése                       | Lassan villog                                                                |
| LED5 | ADT7420 hibajelzése                       | A következő tesztnél ellenőrizendő                                           |
| LED6 | `temp_done` jel                           | Közvetlenül egy rövid impulzust mutat, ezért szemmel nem feltétlenül látható |

Mit jelent ez?

- A LED0 igazolja, hogy az FPGA fő órajele működik.
- A LED1 azt mutatja, hogy a fő vezérlő érzékelte a hőmérsékletolvasás befejezését, és akkor nem jelzett ADT7420-hibát.
- A LED4 azt jelzi, hogy a vezérlő valamelyik további feldolgozási szakaszban időtúllépést érzékelt.

A jelenlegi állapot alapján a következő vizsgálati cél az ADXL362 kommunikációja, illetve az utána következő adatfeldolgozás. A LED2 és LED3 állapota miatt egyelőre nem állíthatjuk biztosan, hogy a gyorsulásmérő adatolvasása sikeres.

## 5. A további fejlesztési és hibakeresési terv

1\. ADT7420 modul végleges ellenőrzése

- Az I²C kommunikáció szimulációja.
- A hőmérsékletregiszter két byte-jának ellenőrzése.
- ACK/NACK jelzések ellenőrzése.
- A beolvasott hőmérsékleti érték ellenőrzése.

2\. ADXL362 modul ellenőrzése

- SPI órajel és adatvonalak vizsgálata.
- A szenzor inicializálásának ellenőrzése.
- Az X, Y és Z tengely adatainak ellenőrzése.
- Az `acc_done` és `acc_error` jelek vizsgálata.

3\. Adatfeldolgozás ellenőrzése

- A `sensor_formatter.v` modul működésének vizsgálata.
- A formázott adatok helyességének ellenőrzése.
- A `fmt_valid` és `fmt_done` jelzések vizsgálata.

4\. UART kommunikáció ellenőrzése

- A 115200 baud sebesség ellenőrzése.
- A UART TX kimenet vizsgálata.
- A PuTTY terminálon megjelenő adatok ellenőrzése.

5\. A teljes rendszer tesztelése

Ha minden részegység külön-külön működik, következhet a teljes mérőrendszer tesztelése, majd a végleges bitstream elkészítése.

## 6. Hol tart most a fejlesztés?

## Fejlesztési állapot

FPGA projekt és programozás

Működik

Fő órajel és vezérlés

Működik

ADT7420 hőmérsékletolvasás

Továbbjut a vezérlés

ADXL362 gyorsulásmérő

Ellenőrzés alatt

Adatformázás

Ellenőrzés alatt

UART adatküldés

Még nem igazolt

Fontos: az ADT7420 sikeresnek látszó működését egyelőre a vezérlő LED-jelzése támasztja alá; a tényleges mért hőmérsékleti értéket és az I²C tranzakciót még szimulációval vagy logikai analizátorral is ellenőrizni kell.
