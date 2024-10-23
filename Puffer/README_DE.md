# Dokumentation: Puffer-Management in SCL

## Übersicht

Diese Dokumentation beschreibt die Implementierung eines Puffer-Management-Systems in **SCL** (Structured Control Language), das dazu dient, Bleche in einem Puffer zu speichern, zu verwalten und ihre Lade- und Entladezeiten zu verfolgen. Der Puffer speichert Informationen darüber, wann Bleche geladen wurden und berechnet, wie viel Zeit seit dem Laden vergangen ist. Außerdem können Lade- und Entladeoperationen durchgeführt werden, und es ist möglich, den Puffer zu einem bestimmten Zeitpunkt zurückzusetzen.

## Inhaltsverzeichnis

1. [Datentypen](#datentypen)
2. [Datenbaustein (DB)](#datenbaustein-db)
3. [Funktion PufferManager](#funktion-puffermanager)
4. [Funktionsweise](#funktionsweise)
   - [Laden und Entladen von Blechen](#laden-und-entladen-von-blechen)
   - [Berechnung der abgelaufenen Zeit](#berechnung-der-abgelaufenen-zeit)
   - [Reset-Funktion](#reset-funktion)
5. [Versionsinformationen](#versionsinformationen)

## Datentypen

### PufferUDT

Der Datentyp `PufferUDT` speichert die Informationen für jeden einzelnen Pufferplatz.

```scl
TYPE PufferUDT
    TITLE   = 'PufferDaten'
    FAMILY  : 'Puffer' 
    AUTHOR  : 'Lopez'
STRUCT
    loaded              : BOOL;       // Gibt an, ob ein Blech geladen wurde (TRUE = geladen)
    loadedDay           : INT;        // Tag, an dem das Blech geladen wurde (1-7 für Montag bis Sonntag)
    loadedTime          : TOD;        // Uhrzeit (Time of Day), zu der das Blech geladen wurde
    loadedExpiredTime   : TIME;       // Verstrichene Zeit seit dem Laden (in Millisekunden)
END_STRUCT
END_TYPE
```

### Attribute

- **loaded**: Status, ob ein Blech im Puffer geladen ist.
- **loadedDay**: Tag der Woche, an dem das Blech geladen wurde (1 = Montag, 7 = Sonntag).
- **loadedTime**: Zeitpunkt des Ladens.
- **loadedExpiredTime**: Verstrichene Zeit seit dem Laden in Millisekunden.

## Datenbaustein (DB)

Der Datenbaustein `PufferDB` speichert alle relevanten Variablen und Zustände des Puffer-Managements.

```scl
DATA_BLOCK PufferDB
    TITLE   = 'Pufferplätze'
    FAMILY  : 'Puffer' 
    AUTHOR  : 'Lopez'
STRUCT 
    aktuelleZeit        : DT;         // Aktuelle Zeit im Format DATE_AND_TIME
    Tag                 : INT;        // Aktueller Tag der Woche (1-7 für Montag bis Sonntag)
    Zeit                : TOD;        // Aktuelle Uhrzeit (Time of Day)
    load               : BOOL;        // Eingabebedingung für den Ladeprozess
    unload             : BOOL;        // Eingabebedingung für den Entladeprozess
    loadFlag           : BOOL;        // Zustandsflag für die positive Flanke des Ladeprozesses
    unloadFlag         : BOOL;        // Zustandsflag für die positive Flanke des Entladeprozesses
    reset              : BOOL;        // Eingabebedingung für das Zurücksetzen des Puffers
    Puffer             : ARRAY [1..100] OF PufferUDT;  // Array für 100 Pufferplätze, die Bleche speichern
END_STRUCT
END_DATA_BLOCK
```

### Attribute

- **aktuelleZeit**: Die aktuelle Systemzeit im Format DATE_AND_TIME.
- **Tag**: Der aktuelle Wochentag (1 = Montag, 7 = Sonntag).
- **Zeit**: Die aktuelle Uhrzeit (Time of Day).
- **load / unload**: Eingabe-Flags zum Steuern des Lade- bzw. Entladevorgangs.
- **loadFlag / unloadFlag**: Zustands-Flags zur Erkennung positiver Flanken.
- **reset**: Flag für das Zurücksetzen des Puffers.
- **Puffer**: Array für die 100 Pufferplätze, die jeweils eine Instanz von `PufferUDT` speichern.

## Funktion PufferManager

Die Funktion `PufferManager` verwaltet das Laden und Entladen von Blechen in den Puffer. Sie berechnet außerdem die Zeit, die seit dem Laden eines Blechs vergangen ist.

### Funktionskopf

```scl
FUNCTION PufferManager : VOID
    VERSION : '0.1'
    FAMILY  : 'Puffer'
    AUTHOR  : 'Lopez'
```

### Lokale Variablen (VAR_TEMP)

```scl
VAR_TEMP
    load              : BOOL;            // Temporäre Variable für den Ladeprozess
    unload            : BOOL;            // Temporäre Variable für den Entladeprozess
    ret               : INT;             // Rückgabewert für das Lesen der Systemzeit
    currTime          : DATE_AND_TIME;   // Aktuelle Systemzeit
    timeDiff          : TIME;            // Zeitdifferenz zwischen aktueller Uhrzeit und Ladezeitpunkt
    dayDiff           : INT;             // Differenz der Tage zwischen aktuellem Tag und Lade-Tag
    i                 : INT;             // Schleifenindex für das Durchlaufen der Pufferplätze
    PufferSize        : INT;             // Variable für die flexible Größe des Puffers
END_VAR
```

## Funktionsweise

### Laden und Entladen von Blechen

- **Laden**: Wenn das Eingabe-Flag `load` aktiv ist und die Bedingung für das Entladen (`unload`) nicht gesetzt ist, wird ein Blech in den Puffer geladen. Das zuletzt geladene Blech wird nach oben geschoben, und der erste Pufferplatz erhält die aktuellen Zeitinformationen.
  
- **Entladen**: Wenn das Eingabe-Flag `unload` aktiv ist, wird der Inhalt der Pufferplätze nach unten geschoben, und der letzte Platz wird freigegeben.

### Berechnung der abgelaufenen Zeit

Für jeden geladenen Pufferplatz wird die verstrichene Zeit seit dem Laden berechnet. Dies umfasst:

- Berechnung der **Tagesdifferenz** zwischen dem aktuellen Tag und dem Lade-Tag.
- Berechnung der **Zeitdifferenz** zwischen der aktuellen Uhrzeit und der Lade-Uhrzeit.
- Begrenzung der maximalen abgelaufenen Zeit auf **24 Stunden**.

### Reset-Funktion

Wenn das Reset-Flag gesetzt ist, werden alle Pufferplätze zurückgesetzt. Dies umfasst:

- Das Zurücksetzen des Ladezustands.
- Setzen des Lade-Datums und der Lade-Uhrzeit auf die aktuellen Werte.
- Zurücksetzen der abgelaufenen Zeit auf 0.

## Versionsinformationen

- **Version**: 0.1
- **Autor**: Lopez
- **Familie**: Puffer
