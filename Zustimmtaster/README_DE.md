# Zustimmtaster_SSP Funktionsblock Dokumentation

## Allgemeine Informationen
- **Titel**: Zustimmtaster_SSP
- **Familie**: Zustimm
- **Autor**: Lopez

## Übersicht
Der Funktionsblock `Zustimmtaster_SSP` steuert einen Sicherheits-Zustimmtaster und die zugehörige Ausgangslogik, die abhängig von verschiedenen Eingangsbedingungen und dem aktuellen Zustand des Tasters aktiviert wird. Dabei wird die Steuerung für LED-Anzeigen, Ausgänge und eine BCD-basierte Zahlenermittlung durchgeführt.

---

## Schnittstellen

### Eingangsvariablen (`var_input`)
| Variable         | Typ    | Beschreibung                                             |
|------------------|--------|----------------------------------------------------------|
| `not_halt_OK`    | `bool` | Signal für Not-Halt-Status (aktiviert, wenn Not-Halt OK). |
| `drei_pos_sw`    | `bool` | Drei-Positionen-Taster (Zustimmtaster).                  |
| `activity_sensor`| `bool` | Aktivitätssensor an Wandhalterung.                       |
| `Bit_0`          | `bool` | Bit 0 für BCD-Zählung.                                   |
| `Bit_1`          | `bool` | Bit 1 für BCD-Zählung.                                   |
| `Bit_2`          | `bool` | Bit 2 für BCD-Zählung.                                   |
| `Bit_3`          | `bool` | Bit 3 für BCD-Zählung.                                   |
| `SW_Right`       | `bool` | Schalter für Rechtsbewegung.                             |
| `SW_Left`        | `bool` | Schalter für Linksbewegung.                              |
| `LED_Takt`       | `bool` | Takt für LED-Blinken.                                    |
| `BCD_offset`     | `int`  | Offset-Wert für BCD-Berechnung.                          |

### Ausgangsvariablen (`var_output`)
| Variable         | Typ            | Beschreibung                                          |
|------------------|----------------|-------------------------------------------------------|
| `LED_Red`        | `bool`         | Anzeige der roten LED (Statussignal).                 |
| `LED_Green`      | `bool`         | Anzeige der grünen LED (Statussignal).                |
| `Out_Right`      | `bool`         | Ausgangssignal für Rechtsbewegung.                    |
| `Out_Left`       | `bool`         | Ausgangssignal für Linksbewegung.                     |
| `BCD_Nr`         | `int`          | Aktuelle BCD-Nummer (Zähler).                         |
| `OutputLeft`     | `ARRAY[1..16] OF BOOL` | Einzelausgänge für Linksbewegung. |
| `OutputRight`    | `ARRAY[1..16] OF BOOL` | Einzelausgänge für Rechtsbewegung. |

### Interne Variablen (`var`)
| Variable             | Typ    | Beschreibung                                         |
|----------------------|--------|------------------------------------------------------|
| `Freigabe`           | `bool` | Steuerfreigabe zur Aktivierung der Ausgänge.         |
| `Edge_flag_pos_3SW`  | `bool` | Flankenmerker für den Drei-Positionen-Taster.        |
| `Edge_flag_pos_LeftSW` | `bool` | Flankenmerker für den linken Schalter.             |
| `Edge_flag_pos_RightSW` | `bool` | Flankenmerker für den rechten Schalter.            |
| `BCD_is_diff`        | `bool` | Marker für Änderung der BCD-Nummer.                  |
| `BCD_Nr_cache`       | `int`  | Zwischenspeicher für vorherige BCD-Nummer.           |
| `Bit_Wert_0`         | `int`  | Wert des BCD-Bits 0.                                 |
| `Bit_Wert_1`         | `int`  | Wert des BCD-Bits 1.                                 |
| `Bit_Wert_2`         | `int`  | Wert des BCD-Bits 2.                                 |
| `Bit_Wert_3`         | `int`  | Wert des BCD-Bits 3.                                 |
| `Constant_bit0`      | `int`  | Konstante für BCD-Wert von Bit 0.                    |
| `Constant_bit1`      | `int`  | Konstante für BCD-Wert von Bit 1.                    |
| `Constant_bit2`      | `int`  | Konstante für BCD-Wert von Bit 2.                    |
| `Constant_bit3`      | `int`  | Konstante für BCD-Wert von Bit 3.                    |
| `i`                  | `int`  | Schleifenindex für Einzelausgänge.                   |

### Temporäre Variablen (`var_temp`)
| Variable               | Typ    | Beschreibung                                           |
|------------------------|--------|--------------------------------------------------------|
| `BCD_Result`           | `int`  | Berechneter BCD-Wert aus den Bit-Eingängen und Offset. |
| `Pulse_GrundStellung`  | `bool` | Puls für Grundstellung des Zustimmtasters.             |
| `Pulse_Left`           | `bool` | Puls für linken Schalter.                              |
| `Pulse_Right`          | `bool` | Puls für rechten Schalter.                             |

---

## Funktionsbeschreibung

### LED-Anzeige
Die LEDs werden basierend auf den Eingangssignalen gesteuert:
- **LED_Red** zeigt an, wenn `not_halt_OK` nicht aktiv ist.
- **LED_Green** blinkt oder leuchtet dauerhaft basierend auf dem `activity_sensor`, `drei_pos_sw`, `LED_Takt` und `not_halt_OK`.

### BCD-Berechnung
Die BCD-Berechnung erfolgt durch Summierung der Bits `Bit_0` bis `Bit_3` mit dem `BCD_offset`:
1. **Bit-Werte**: Jedes Bit (`Bit_0` bis `Bit_3`) wird entsprechend seiner Konstante zu einem Wert umgerechnet.
2. **Summierung**: Die Summe der Bit-Werte mit dem `BCD_offset` ergibt den aktuellen BCD-Wert (`BCD_Result`).
3. **Änderungsprüfung**: Eine Änderung (`BCD_is_diff`) wird erkannt, wenn `BCD_Result` sich von `BCD_Nr_cache` unterscheidet.

### Freigabe-Steuerung
Die Freigabe (`Freigabe`) wird bei Änderung des BCD-Wertes oder Loslassen des Zustimmtasters (`drei_pos_sw`) zurückgesetzt. Eine erneute Aktivierung erfolgt bei positiver Flanke des Zustimmtasters (`Pulse_GrundStellung`).

### Ausgangssteuerung
- **Links**: Der linke Ausgang (`Out_Left`) wird bei positivem Puls von `SW_Left`, aktivierter Freigabe und `not_halt_OK` aktiviert, sofern `SW_Right` nicht aktiv ist.
- **Rechts**: Der rechte Ausgang (`Out_Right`) wird bei positivem Puls von `SW_Right`, aktivierter Freigabe und `not_halt_OK` aktiviert, sofern `SW_Left` nicht aktiv ist.

### Einzelausgänge
Jeder Einzelausgang von `OutputLeft` und `OutputRight` wird aktiviert, wenn die aktuelle BCD-Nummer (`BCD_Result`) dem Index des Ausgangs entspricht und der jeweilige Hauptausgang (`Out_Left` bzw. `Out_Right`) aktiv ist.

---

## Ablauf

1. **Initialisierung**: Variablen und Bit-Werte werden basierend auf den Eingängen berechnet.
2. **LED-Steuerung**: Anzeigen werden basierend auf `not_halt_OK` und anderen Parametern gesetzt.
3. **Freigabe-Prüfung**: Die Freigabe wird basierend auf dem Zustand des Zustimmtasters und den Schaltern gesetzt oder zurückgesetzt.
4. **Ausgangssteuerung**: Die Ausgänge `Out_Left` und `Out_Right` sowie die Einzelausgänge werden basierend auf `BCD_Result` und den Hauptausgängen aktiviert oder deaktiviert.

---

## Hinweise
- Die Freigabe wird nur bei aktiven Sicherheits- und Bedingungssignalen sowie positiver Flanke des Zustimmtasters gesetzt.
- Die LED- und Ausgangssteuerung basieren auf synchronisierten Flanken und Zustandsprüfungen, um Fehlsteuerungen zu verhindern.

--- 

## Changelog
- **Version 1.0** - Erstveröffentlichung durch Autor Lopez
