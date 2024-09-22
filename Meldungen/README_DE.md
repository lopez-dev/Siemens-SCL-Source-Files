# Funktionsbaustein `fb_alarming`

## Übersicht

Der Funktionsbaustein `fb_alarming` ist für das Verwalten von Alarmbedingungen zuständig, indem er eine Reihe von Signalen überwacht und neue Fehler erkennt. Er verwendet eine Flankenerkennung, um Änderungen zu identifizieren und speichert Momentaufnahmen der Alarmzustände zum Vergleich in späteren Zyklen.

---

## Schnittstelle

### Eingangsparameter

| Name   | Typ   | Beschreibung |
|--------|-------|--------------|
| `reset` | `Bool` | Reset-Eingang zum Zurücksetzen von Alarmen und Zuständen. |

### Ausgangsparameter

| Name      | Typ  | Beschreibung |
|-----------|------|--------------|
| `error`   | `Bool` | Gibt an, ob derzeit ein Fehler aktiv ist. |
| `new_error` | `Bool` | Gibt an, ob seit dem letzten Reset ein neuer Fehler erkannt wurde. |
| `state`   | `Byte` | Zeigt den aktuellen Zustand der Alarme an. |

### In/Out-Parameter

| Name       | Typ           | Beschreibung |
|------------|----------------|--------------|
| `Meldungen` | `Array[0..7] of Word` | Array der überwachten Alarmwörter. |

### Interne Variablen

| Name            | Typ   | Beschreibung |
|-----------------|-------|--------------|
| `MeldeSnapshot` | `Array[0..7] of Word` | Speichert die Momentaufnahme des vorherigen Alarmzustands zum Vergleich. |
| `reset_puls_pos` | `Bool` | Speichert die positive Flanke des Reset-Signals. |
| `st_error` | `Bool` | Interne Flagge zur Verfolgung aktiver Fehler. |
| `st_new_error` | `Bool` | Interne Flagge zur Erkennung neuer Fehler. |
| `R_TRIG_Instance` | `R_TRIG` | Instanz zur Erkennung von steigenden Flanken. |
| `R_TRIG_Instance` | `Array[0..15] of R_TRIG` | Array von Flankentriggern (steigende Flanke). |
| `F_TRIG_Instance` | `Array[0..15] of F_TRIG` | Array von Flankentriggern (fallende Flanke). |
| `dummy` | `Bool` | Temporäre Variable für ungenutzte Ausgänge. |
| `i` | `Int` | Schleifenindex zum Durchlaufen der Alarme. |

---

## Logikstruktur

### Alarmüberwachung (`#error`-Region)

Diese Region überprüft jedes Alarmwort im Array `Meldungen`, um festzustellen, ob ein Alarm aktiv ist. Wenn ein Bit in den Wörtern gesetzt ist, wird die Flagge `st_error` auf `TRUE` gesetzt.

```scl
IF (#Meldungen[0] OR #Meldungen[1] OR ... OR #Meldungen[7]) <> 0 THEN
    #st_error := TRUE;
ELSE
    #st_error := FALSE;
END_IF;
```

### Reset-Flankenerkennung (`reset pulse`-Region)

Diese Region ruft die Funktion `fc_pulse` auf, um eine Flanke vom Reset-Signal zu generieren. Sie erkennt eine positive Flanke am Reset-Eingang, um weitere Aktionen auszulösen.

```scl
"fc_pulse"(In_Var := #reset,
           Pulse_pos_Var => #reset_puls_pos,
           Pulse_neg_Var => #dummy,
           Edge_flag_pos := #dummy,
           Edge_flag_neg := #dummy);
```

### Momentaufnahme und Reset (`reset`-Region)

Wenn eine Reset-Flanke erkannt wird (`reset_puls_pos`), wird der aktuelle Zustand des Arrays `Meldungen` in `MeldeSnapshot` gespeichert, und die Flagge `st_new_error` wird auf `FALSE` zurückgesetzt.

```scl
IF #reset_puls_pos THEN
    #MeldeSnapshot := #Meldungen;
    #st_new_error := FALSE;
END_IF;
```

### Schleife zur Fehlererkennung (`Schleife für erkennung von neue Fehler`-Region)

Diese Schleife durchläuft jedes Bit im Array `Meldungen` und vergleicht es mit dem entsprechenden Bit in `MeldeSnapshot`, um festzustellen, ob ein neuer Fehler aufgetreten ist. Wenn eine steigende Flanke erkannt wird (Übergang von 0 auf 1), wird `st_new_error` auf `TRUE` gesetzt.

```scl
FOR #i := 0 TO 7 DO
    IF #Meldungen[#i].%X0 = 1 AND #MeldeSnapshot[#i].%X0 = 0 THEN
        #st_new_error := TRUE;
    END_IF;
    ...
END_FOR;
```

### Zustandsverwaltung (`state`-Region)

Der Zustand der Alarme wird basierend auf dem Status von `st_error` und `st_new_error` bestimmt:
- `state = 0`: Kein Fehler
- `state = 1`: Fehler, aber kein neuer Fehler
- `state = 2`: Neuer Fehler erkannt
- `state = 3`: Undefinierter Zustand

```scl
IF NOT #st_error AND NOT #new_error THEN
    #state := 0;
ELSIF #st_error AND NOT #st_new_error THEN
    #state := 1;
ELSIF #error AND #st_new_error THEN
    #state := 2;
ELSE
    #state := 3;
END_IF;
```

### Ausgabezuweisung (`OUTPUTS`-Region)

Die Ausgangsvariablen `error` und `new_error` werden mit den Werten der internen Flaggen `st_error` und `st_new_error` belegt.

```scl
#error := #st_error;
#new_error := #st_new_error;
```

---

## Verwandte Funktionen

### `fc_pulse`

Diese Funktion generiert positive und negative Flanken von einer Booleschen Eingabe. Sie wird in der `reset pulse`-Region verwendet, um die steigenden und fallenden Flanken des Reset-Signals zu erkennen.

| Eingang | Typ  | Beschreibung |
|---------|------|--------------|
| `In_Var` | `Bool` | Die Boolesche Eingabevariable, von der die Flanke gebildet wird. |

| Ausgang | Typ  | Beschreibung |
|---------|------|--------------|
| `Pulse_pos_Var` | `Bool` | Ausgang, der die positive Flanke (steigende Flanke) anzeigt. |
| `Pulse_neg_Var` | `Bool` | Ausgang, der die negative Flanke (fallende Flanke) anzeigt. |

| In/Out | Typ  | Beschreibung |
|--------|------|--------------|
| `Edge_flag_pos` | `Bool` | Speichert den Zustand zur Erkennung der positiven Flanke. |
| `Edge_flag_neg` | `Bool` | Speichert den Zustand zur Erkennung der negativen Flanke. |

---

## Datentypen

### `typ_Meld_HMI`

Dieser benutzerdefinierte Datentyp wird zum Speichern und Verwalten von Alarm- und Warnmeldungen für das HMI verwendet.

| Name | Typ  | Beschreibung |
|------|------|--------------|
| `ACK` | `Bool` | Quittierungssignal zum Zurücksetzen von Alarmen. |
| `Alarm` | `Bool` | Signal zur Anzeige von Alarmmeldungen. |
| `Warn` | `Bool` | Signal zur Anzeige von Warnmeldungen. |
| `Meld_Quitt` | `Array[0..19] of Word` | Array, das den Status von Alarmen und deren Quittierungen speichert. |
| `Warnungen` | `Array[0..1] of Word` | Array, das die Warnmeldungen speichert. |

---
