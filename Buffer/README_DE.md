
---

# Buffer Management System

Diese Dokumentation beschreibt die Funktionsweise des Buffer Management Systems zur Verwaltung von Lade- und Entladeprozessen von Blechen. Der Code enthält benutzerdefinierte Datentypen und eine Funktion zur Erfassung und Verarbeitung der Pufferdaten.

## Inhaltsverzeichnis

1. [Überblick](#überblick)
2. [Strukturen und Datentypen](#strukturen-und-datentypen)
   - [BufferUDT](#bufferudt)
   - [BufferStructUDT](#bufferstructudt)
3. [Datenbaustein BufferDB](#datenbaustein-bufferdb)
4. [Funktion BufferManager](#funktion-buffermanager)
   - [Eingangsvariablen](#eingangsvariablen)
   - [Temporäre Variablen](#temporäre-variablen)
   - [Funktionsablauf](#funktionsablauf)
5. [Versionsinformationen](#versionsinformationen)

---

## Überblick

Das Buffer Management System verwaltet die Pufferplätze für Bleche, wobei die Lade- und Entladeprozesse sowie die Überwachung des Status der Bleche (z. B. Beladungszeit und Verweildauer) organisiert werden. Die Funktion `BufferManager` ermöglicht die Verwaltung dieser Prozesse und führt Statistiken über die Nutzung der Pufferplätze.

---

## Strukturen und Datentypen

### BufferUDT

Diese Struktur repräsentiert einen einzelnen Pufferplatz für ein Blech und enthält Informationen über den Ladezustand, die Beladungszeit und die abgelaufene Zeit seit der Beladung.

```scl
TYPE BufferUDT
    TITLE   = 'BufferDaten'
    FAMILY  : 'Buffer' 
    AUTHOR  : 'Lopez'
STRUCT
    loaded              : BOOL;       // TRUE = Ein Blech ist geladen
    loadedcurrentDay    : INT;        // Tag der Beladung (1=Sonntag bis 7=Samstag)
    loadedTime          : TOD;        // Uhrzeit der Beladung (Zeit des Tages)
    loadedExpiredTime   : TIME;       // Verstrichene Zeit seit der Beladung (ms)
END_STRUCT
END_TYPE
```

### BufferStructUDT

Diese Struktur fasst mehrere Pufferplätze und deren Steuer- und Statusinformationen zusammen. Sie dient als Container für eine Sammlung von Blechen und enthält zusätzliche Steuerungs- und Statistikvariablen.

```scl
TYPE BufferStructUDT
    TITLE   = 'BufferStruct'
    FAMILY  : 'Buffer' 
    AUTHOR  : 'Lopez'
STRUCT
    amountLoaded     : INT;               // Aktuelle Anzahl geladener Bleche im Puffer
    firstLoadedTime  : TOD;               // Uhrzeit des ersten geladenen Blechs
    firstLoadedcurrentDay   : INT;        // Tag der Beladung des ersten Blechs (1=Sonntag bis 7=Samstag)
    firstExpiredTime : TIME;              // Verstrichene Zeit des ersten Blechs im Puffer
    load             : BOOL;              // Startsignal für den Ladeprozess
    unload           : BOOL;              // Startsignal für den Entladeprozess
    manualLoad       : BOOL;              // Startsignal für den Ladeprozess (manuell)
    manualUnload     : BOOL;              // Startsignal für den Entladeprozess (manuell)
    loadFlag         : BOOL;              // Zustandsflag zur Flankenerkennung des Ladeprozesses
    unloadFlag       : BOOL;              // Zustandsflag zur Flankenerkennung des Entladeprozesses
    reset            : BOOL;              // Signal zum Zurücksetzen des gesamten Puffers
    AmountFlag       : INT;               // Vorheriger Wert (zur Berechnung des StatAmount0)
    currentDayFlag   : INT;               // Vorheriger Wert (zur Berechnung)
    StatBuffered     : ARRAY[1..7] OF INT; // Maximale Anzahl an gepufferten Blechen pro Tag
    StatMaxTime      : ARRAY[1..7] OF TIME; // Maximale gepufferte Zeit pro Tag
    StatAmount0      : ARRAY[1..7] OF INT; // Anzahl der Tage, an denen der Speicher komplett leergefahren wurde
    StatPorzLoaded   : INT;               // Prozentsatz des beladenen Puffers
    BufferSize       : INT;               // Größe des Puffers bzw. des Arrays       
    Buffer           : ARRAY [1..100] OF BufferUDT;  // Array für 100 Pufferplätze
END_STRUCT;
END_TYPE
```

---

## Datenbaustein BufferDB

Der Datenbaustein `BufferDB` enthält die Pufferstrukturen und Informationen zur aktuellen Zeit. Die Struktur kann zwei Pufferstrukturen aufnehmen.

```scl
DATA_BLOCK BufferDB
    TITLE   = 'Pufferplätze'
    FAMILY  : 'Buffer' 
    AUTHOR  : 'Lopez'
STRUCT 
    currentDT        : DT;         // Aktuelle Systemzeit im Format DATE_AND_TIME
    currentDay       : INT;        // Aktueller Wochentag
    currentTime      : TOD;        // Aktuelle Uhrzeit (Zeit des Tages)
    Buffer           : ARRAY[1..2] OF BufferStructUDT; // Array für 2 Pufferstrukturen
END_STRUCT

BEGIN
    // Initialisierungen
    Buffer[1].BufferSize := 90;
    Buffer[2].BufferSize := 89;
END_DATA_BLOCK
```

---

## Funktion BufferManager

Die Funktion `BufferManager` verwaltet den Lade- und Entladeprozess der Bleche im Puffer, aktualisiert Zeitstempel und erstellt eine statistische Auswertung der Pufferauslastung.

### Eingangsvariablen

| Variable | Typ  | Beschreibung |
|----------|------|--------------|
| `IN_P1`  | BOOL | Steuersignal für den Ladevorgang Puffer 1 |
| `OUT_P1` | BOOL | Steuersignal für den Entladevorgang Puffer 1 |
| `IN_P2`  | BOOL | Steuersignal für den Ladevorgang Puffer 2 |
| `OUT_P2` | BOOL | Steuersignal für den Entladevorgang Puffer 2 |

### Temporäre Variablen

| Variable           | Typ                | Beschreibung |
|--------------------|--------------------|--------------|
| `ret`              | INT                | Rückgabewert für Systemzeiterfassung |
| `j`, `i`           | INT                | Schleifenindizes für Puffer und Plätze |
| `load`, `unload`   | ARRAY[1..2] OF BOOL | Temporäre Lade- und Entladevariablen |
| `currTime`         | DATE_AND_TIME      | Aktuelle Systemzeit |
| `timeDiff`         | TIME               | Zeitdifferenz zwischen Lade- und Entladezeit |
| `amountCounter`    | INT                | Zähler für geladene Bleche |
| `firstLoadedTime`  | TOD                | Zeitstempel des ersten geladenen Blechs |
| `addStatBuffered`  | BOOL               | Flag zur Aktualisierung der Statistik |

### Funktionsablauf

1. **Eingabe-Handling:** Die Eingangsvariablen werden Puffersteuersignalen zugeordnet.
2. **Systemzeiterfassung:** Liest die aktuelle Systemzeit, setzt den aktuellen Tag und die Tageszeit.
3. **Lade- und Entladevorgänge:**
   - **Laden:** Überprüft das `load` und `manualLoad` Signal; bei positiver Flanke wird ein Blech geladen.
   - **Entladen:** Überprüft das `unload` und `manualUnload` Signal; bei positiver Flanke wird ein Blech entladen.
4. **Zeitberechnung:** Berechnet die `loadedExpiredTime` für jedes geladene Blech.
5. **Statistikberechnung:** Aktualisiert Statistiken wie `StatBuffered`, `StatMaxTime` und berechnet den Prozentsatz der Pufferbelegung.
6. **Puffer-Reset:** Setzt den gesamten Puffer zurück, falls das `reset`-Signal aktiv ist.

### Versionsinformationen

- **Version:** 0.1
- **Autor:** Lopez
- **Familie:** Buffer

---