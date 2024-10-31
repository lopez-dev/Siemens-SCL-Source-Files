# Dokumentation zur Ventilsteuerung (Valve Control)

## Übersicht

Dieses Dokument beschreibt die Struktur und Funktionsweise der Ventilsteuerung in SCL. Die Implementierung umfasst die Struktur `udt_valveStruct` zur Definition der Ventildaten und den Funktionsbaustein `fb_valveControl` zur Steuerung der Ventilzustände. Zusätzliche Funktionen `fc_valveClose` und `fc_valveOpen` bieten einfache Mechanismen zur Initialisierung der Schließ- und Öffnungszustände des Ventils.

- ✅ **Completed**: This project is fully developed and tested.

## Inhaltsverzeichnis

1. [Struktur `udt_valveStruct`](#udt_valvestruct)
2. [Funktionsbaustein `fb_valveControl`](#fb_valvecontrol)
3. [Hilfsfunktionen](#hilfsfunktionen)
   - [Funktion `fc_valveClose`](#fc_valveclose)
   - [Funktion `fc_valveOpen`](#fc_valveopen)

---

### `udt_valveStruct`

Die Struktur `udt_valveStruct` beschreibt die Eigenschaften und Zustände eines Ventils.

#### Felder:

- **state** : `Int`  
  Aktueller Zustand des Ventils.
  - `0`: geschlossen
  - `1`: öffnet
  - `2`: offen
  - `3`: schließt

- **typeOfValve** : `Int`  
  Art des Ventils.
  - `0`: Einzelsolenoid (Standard)
  - `1`: Doppelsolenoid

- **open** : `Bool`  
  Eingang zum Öffnen des Ventils (TRUE startet das Öffnen).

- **close** : `Bool`  
  Eingang zum Schließen des Ventils (TRUE startet das Schließen).

- **fb_opened** : `Bool`  
  Rückmeldesignal für vollständig geöffnetes Ventil.

- **fb_closed** : `Bool`  
  Rückmeldesignal für vollständig geschlossenes Ventil.

- **y_open** : `Bool`  
  Ausgang zur Steuerung des Öffnungsvorgangs.

- **y_close** : `Bool`  
  Ausgang zur Steuerung des Schließvorgangs.

- **errorState** : `UInt`  
  Fehlerstatus-Code:
  - `0`: Kein Fehler
  - `1`: Ventil öffnet nicht
  - `2`: Ventil schließt nicht
  - `3`: Öffnen und Schließen gleichzeitig aktiviert
  - `4`: Unbekannter Ventilstatus
  - `5`: Offen- und Schließ-Rückmeldung gleichzeitig aktiv
  - `6`: Geschlossen, aber `fb_opened` aktiv
  - `7`: Offen, aber `fb_closed` aktiv

- **errorText** : `String[60]`  
  Textbeschreibung des aktuellen Fehlerstatus.

- **openTime** : `Time`  
  Timeout-Dauer für das Öffnen. `T#0ms` bedeutet deaktiviert.

- **closeTime** : `Time`  
  Timeout-Dauer für das Schließen. `T#0ms` bedeutet deaktiviert.

- **unknowTime** : `Time`  
  Timeout für den Erkennungsstatus bei unbekanntem Zustand. `T#0ms` bedeutet deaktiviert.

- **timerOpen** : `TON_TIME`  
  Timer zur Überwachung des Öffnungsvorgangs.

- **timerClose** : `TON_TIME`  
  Timer zur Überwachung des Schließvorgangs.

- **timerUnknown** : `TON_TIME`  
  Timer zur Überwachung des unbekannten Ventilstatus.

---

### `fb_valveControl`

Der Funktionsbaustein `fb_valveControl` steuert das Verhalten des Ventils basierend auf den Eingängen und Zustandsrückmeldungen.

#### Variablen:

- **valve** : `udt_valveStruct`  
  Die übergebene Ventilstruktur als In-Out-Parameter.

#### Konstante Zustandswerte:

- `STATE_CLOSED` : `Int := 0`  
  Zustand für "geschlossen".
  
- `STATE_OPENING` : `Int := 1`  
  Zustand für "öffnend".

- `STATE_OPENED` : `Int := 2`  
  Zustand für "geöffnet".

- `STATE_CLOSING` : `Int := 3`  
  Zustand für "schließend".

#### Regionen:

- **States legend**  
  Beschreibung der verschiedenen Zustände des Ventils.

- **typeOfValve**  
  Logik zur Überprüfung des Ventiltyps und zur Anwendung spezifischen Verhaltens:
  - Einzelsolenoid: Schließt automatisch, wenn "Öffnen" nicht aktiv ist.
  - Doppelsolenoid: Noch keine spezifische Logik implementiert, Platzhalter für zukünftige Erweiterungen.

- **Valve Control Logic**  
  Steuerung des Ventilverhaltens basierend auf dem aktuellen Zustand und den Eingabebefehlen. Übergänge zwischen den Zuständen werden je nach Eingangs- und Rückmeldesignalen des Ventils definiert.

- **Valve Outputs**  
  Steuerung der Ausgangssignale `y_open` und `y_close` basierend auf dem Ventilzustand.

- **Error handling**  
  Fehlererkennung und -zuweisung basierend auf Timern, Konflikten in Eingaben und Rückmeldesignalen.

- **Error text**  
  Zuweisung von Fehlerbeschreibungen in Abhängigkeit des Fehlerstatus.

---

### Hilfsfunktionen

#### `fc_valveClose`

Die Funktion `fc_valveClose` überprüft, ob das Ventil nicht bereits im Zustand "geschlossen" oder "schließend" ist, und setzt bei Bedarf den Zustand auf `STATE_CLOSING`.

```scl
FUNCTION "fc_valveClose" : Void
    { S7_Optimized_Access := 'TRUE' }
    VERSION : 0.1
VAR_IN_OUT
    valveData : "udt_valveStruct";
END_VAR

BEGIN
    IF #valveData.state <> 0 AND #valveData.state <> 3 THEN
        #valveData.state := 3;
    END_IF;
END_FUNCTION
```

#### `fc_valveOpen`

Die Funktion `fc_valveOpen` prüft, ob das Ventil nicht bereits im Zustand "öffnend" oder "offen" ist, und setzt bei Bedarf den Zustand auf `STATE_OPENING`.

```scl
FUNCTION "fc_valveOpen" : Void
    { S7_Optimized_Access := 'TRUE' }
    VERSION : 0.1
VAR_IN_OUT
    valveData : "udt_valveStruct";
END_VAR

BEGIN
    IF #valveData.state <> 1 AND #valveData.state <> 2 THEN
        #valveData.state := 1;
    END_IF;
END_FUNCTION
```

---

### Änderungen und Versionshistorie

- **Version 0.1**  
  - Initiale Implementierung der Ventilstruktur und des Funktionsbausteins zur Steuerung sowie grundlegende Hilfsfunktionen.

--- 
