# Dokumentation der Timerverwaltung in SCL

## Inhaltsverzeichnis

1. [Einleitung](#einleitung)
2. [Struktur des UDT](#struktur-des-udt)
3. [Datenblock (TimerDatabase)](#datenblock-timerdatabase)
4. [Funktionen](#funktionen)
   - [TimerReset](#funktion-timerreset)
   - [TimerStart](#funktion-timerstart)
   - [TimerTOF](#funktion-timertof)
   - [TimerTON](#funktion-timerton)
   - [TimerPause](#funktion-timerpause)
   - [TimerResume](#funktion-timerresume)
   - [TimerHandling](#funktion-timerhandling)
   - [TimerElapsed](#funktion-timerelapsed)
5. [Zusammenfassung](#zusammenfassung)

## Einleitung

Diese Dokumentation beschreibt die Implementierung und Funktionsweise eines Timer-Systems, das in der SPS mit der Programmiersprache **Structured Control Language (SCL)** erstellt wurde. Das System ermöglicht die Verwaltung mehrerer Timer und deren Zustände in einem industriellen Automatisierungsumfeld. Verschiedene Funktionen wie das Starten, Pausieren, Zurücksetzen und Überwachen der Timer werden detailliert beschrieben.

## Struktur des UDT

Der benutzerdefinierte Datentyp (UDT) `udt_timer` wird verwendet, um die Informationen eines Timers zu speichern.

```scl
TYPE "udt_timer"
VERSION : 0.1
   STRUCT
      timerState : Int;      // Aktueller Zustand des Timers: 0=Aus, 1=Laufend, 2=Pausiert, 3=Abgelaufen
      startTime : Time;      // Startzeit des Timers
      setTime : Time;        // Vordefinierte Laufzeit des Timers (z.B. 2500 ms)
      elapsedTime : Time;    // Bereits abgelaufene Zeit seit dem Start des Timers
   END_STRUCT;
END_TYPE
```

### Beschreibung der Felder:
- **timerState**: Zeigt den aktuellen Zustand des Timers an.
- **startTime**: Zeitpunkt, zu dem der Timer gestartet wurde.
- **setTime**: Definierte Dauer, für die der Timer laufen soll.
- **elapsedTime**: Zeit, die seit dem Start des Timers vergangen ist.

## Datenblock TimerDatabase

Der Datenblock `TimerDatabase` enthält alle Timer-Datenstrukturen sowie weitere Verwaltungsvariablen.

```scl
DATA_BLOCK "TimerDatabase"
{ S7_Optimized_Access := 'TRUE' }
VERSION : 0.1
NON_RETAIN
   VAR 
      timersTime : Time;        // Aktuelle Systemzeit für die Timer
      timersAmount : Int;       // Gesamtanzahl der Timer im Array
      timersPerBatch : Int;     // Anzahl der Timer, die pro Zyklus/Batch verarbeitet werden
      timersBatchCounter : Int; // Zähler, um den aktuellen Batch von Timern zu verfolgen
      timers { S7_SetPoint := 'False'} : Array[0..599] of "udt_timer"; // Array, das alle Timer enthält
   END_VAR
END_DATA_BLOCK
```

### Beschreibung der Variablen:
- **timersTime**: Systemzeit, die für die Berechnung der Timer verwendet wird.
- **timersAmount**: Gesamtzahl der Timer im System (max. 600).
- **timersPerBatch**: Anzahl der Timer, die pro Zyklus verarbeitet werden.
- **timersBatchCounter**: Zähler für die Verfolgung der Batches.
- **timers**: Array von Timer-Datensätzen (bis zu 600 Timer).

## Funktionen

### Funktion TimerReset

Diese Funktion setzt den Zustand eines Timers auf den Ausgangszustand (0=Aus) zurück.

```scl
FUNCTION "TimerReset" : Void
{ S7_Optimized_Access := 'TRUE' }
VERSION : 0.1
   VAR_INPUT 
      Timer_Nr : Int; // Index des Timers, der zurückgesetzt werden soll
   END_VAR
BEGIN
   "TimerDatabase".timers[#Timer_Nr].timerState := 0;
END_FUNCTION
```

### Funktion TimerStart

Startet einen Timer, indem seine Startzeit auf die aktuelle Systemzeit gesetzt wird.

```scl
FUNCTION "TimerStart" : Void
{ S7_Optimized_Access := 'TRUE' }
VERSION : 0.1
   VAR_INPUT 
      Timer_Nr : Int; // Index des Timers, der gestartet werden soll
   END_VAR
BEGIN
   IF "TimerDatabase".timers[#Timer_Nr].timerState = 0 THEN
      "TimerDatabase".timers[#Timer_Nr].startTime := "TimerDatabase".timersTime;
      "TimerDatabase".timers[#Timer_Nr].timerState := 1; // Timer läuft
   END_IF;
END_FUNCTION
```

### Funktion TimerTOF

Ein **Timer Off-Delay** (TOF) startet den Timer, wenn die Eingangsbedingung wahr ist, und setzt ihn zurück, wenn die Eingangsbedingung falsch ist.

```scl
FUNCTION "TimerTOF" : Void
{ S7_Optimized_Access := 'TRUE' }
VERSION : 0.1
   VAR_INPUT 
      TimerNr : Int; // Index des Timers
      IN : Bool;     // Eingangssignal zum Starten/Zurücksetzen des Timers
   END_VAR
   VAR_OUTPUT 
      OUT : Bool;    // Ausgangssignal, ob der Timer abgelaufen ist
   END_VAR
BEGIN
   IF #IN AND ("TimerDatabase".timers[#TimerNr].timerState <> 2) THEN
      "TimerReset"(Timer_Nr := #TimerNr);
      "TimerStart"(Timer_Nr := #TimerNr);
      "TimerDatabase".timers[#TimerNr].timerState := 2;
   ELSIF NOT #IN AND ("TimerDatabase".timers[#TimerNr].timerState = 2) THEN
      "TimerReset"(Timer_Nr := #TimerNr);
      "TimerStart"(Timer_Nr := #TimerNr);
   END_IF;
   #OUT := #IN OR "TimerDatabase".timers[#TimerNr].timerState = 1;
END_FUNCTION
```

### Funktion TimerTON

Ein **Timer On-Delay** (TON) startet den Timer, wenn die Eingangsbedingung wahr ist, und setzt ihn zurück, wenn die Eingangsbedingung falsch ist.

```scl
FUNCTION "TimerTON" : Void
{ S7_Optimized_Access := 'TRUE' }
VERSION : 0.1
   VAR_INPUT 
      TimerNr : Int; // Index des Timers
      IN : Bool;     // Eingangssignal zum Starten/Zurücksetzen des Timers
   END_VAR
   VAR_OUTPUT 
      OUT : Bool;    // Ausgangssignal, ob der Timer abgelaufen ist
   END_VAR
BEGIN
   IF #IN AND ("TimerDatabase".timers[#TimerNr].timerState = 0) THEN
      "TimerReset"(Timer_Nr := #TimerNr);
      "TimerStart"(Timer_Nr := #TimerNr);
   ELSIF NOT #IN AND NOT ("TimerDatabase".timers[#TimerNr].timerState = 0) THEN
      "TimerReset"(Timer_Nr := #TimerNr);
   END_IF;
   #OUT := "TimerDatabase".timers[#TimerNr].timerState = 3;
END_FUNCTION
```

### Funktion TimerPause

Pausiert einen laufenden Timer.

```scl
FUNCTION "TimerPause" : Void
{ S7_Optimized_Access := 'TRUE' }
VERSION : 0.1
   VAR_INPUT 
      Timer_nr : Int; // Index des zu pausierenden Timers
   END_VAR
BEGIN
   IF "TimerDatabase".timers[#Timer_nr].timerState = 1 THEN
      "TimerDatabase".timers[#Timer_nr].timerState := 2; // Timer pausieren
   END_IF;
END_FUNCTION
```

### Funktion TimerResume

Setzt einen pausierten Timer fort.

```scl
FUNCTION "TimerResume" : Void
{ S7_Optimized_Access := 'TRUE' }
VERSION : 0.1
   VAR_INPUT 
      Timer_nr : Int; // Index des Timers, der fortgesetzt werden soll
   END_VAR
BEGIN
   IF "TimerDatabase".timers[#Timer_nr].timerState = 2 THEN
      "TimerDatabase".timers[#Timer_nr].startTime := "TimerDatabase".timersTime - "TimerDatabase".timers[#Timer_nr].elapsedTime;
      "TimerDatabase".timers[#Timer_nr].timerState := 1; // Timer läuft wieder
   END_IF;
END_FUNCTION
```

### Funktion TimerHandling

Verarbeitet die Timer in Batches, aktualisiert die abgelaufene Zeit und prüft, ob ein Timer abgelaufen ist.

```scl
FUNCTION "TimerHandling" : Void
{ S7_Optimized_Access := 'TRUE' }
VERSION : 0.1
   VAR_TEMP 
      ret : Int;
      CurrentTime_DTL : DTL; // Aktuelle Systemzeit
      nStartTimerIndex : Int;
      nEndTimerIndex : Int;
      iTimerIndex : Int;
      nTimersPerBatch : Int;
      timersArraySize : Int;
   END_VAR
BEGIN
   // Aktuelle Systemzeit lesen und in

 TIME umwandeln
   #ret := RD_LOC_T(#CurrentTime_DTL);
   "TimerDatabase".timersTime := TOD_TO_TIME(IN := DTL_TO_TOD(IN := #CurrentTime_DTL));
   
   // Timer-Update pro Batch
   FOR #iTimerIndex := #nStartTimerIndex TO #nEndTimerIndex DO
      IF "TimerDatabase".timers[#iTimerIndex].timerState = 1 THEN
         "TimerDatabase".timers[#iTimerIndex].elapsedTime := "TimerDatabase".timersTime - "TimerDatabase".timers[#iTimerIndex].startTime;
         IF "TimerDatabase".timers[#iTimerIndex].elapsedTime >= "TimerDatabase".timers[#iTimerIndex].setTime THEN
            "TimerDatabase".timers[#iTimerIndex].timerState := 3; // Timer abgelaufen
         END_IF;
      END_IF;
   END_FOR;
END_FUNCTION
```

### Funktion TimerElapsed

Prüft, ob ein Timer abgelaufen ist (Zustand 3).

```scl
FUNCTION "TimerElapsed" : Bool
{ S7_Optimized_Access := 'TRUE' }
VERSION : 0.1
   VAR_INPUT 
      Timer_Nr : Int; // Index des Timers, der überprüft werden soll
   END_VAR
BEGIN
   #TimerElapsed := "TimerDatabase".timers[#Timer_Nr].timerState = 3;
END_FUNCTION
```

## Zusammenfassung

Diese Dokumentation beschreibt ein umfassendes Timer-Verwaltungssystem für SPS-Anwendungen. Die verschiedenen Funktionen ermöglichen das Starten, Pausieren, Zurücksetzen und Überwachen der Timer in einem optimierten Batch-Verfahren, um die Leistung der SPS zu maximieren.
