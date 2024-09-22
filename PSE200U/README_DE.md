# Funktionsbaustein `fb_PSE200U`

## Übersicht

Der Funktionsbaustein `fb_PSE200U` ist verantwortlich für das Verwalten und Überwachen des Status eines PSE200U-Geräts. Er steuert die Ablaufsequenz, überprüft die Status der Kanäle, überwacht den Fortschritt der Schritte und erkennt Fehler in Echtzeit.

### Wichtige Funktionen:
- Überwacht bis zu 4 Kanäle auf korrekte Funktion.
- Implementiert eine sequentielle Schrittsteuerung.
- Erkennt und behandelt Fehler während der Ausführung.
- Stellt detaillierte Status- und Fehlerausgaben zur Verfügung.

---

## Schnittstelle

### Eingangsparameter

| Name | Typ  | Beschreibung |
|------|------|--------------|
| `S`  | `Bool` | Statussignal vom PSE200U-Gerät. Dieses Eingangssignal startet die Sequenz, wenn es TRUE ist. |

### Ausgangsparameter

| Name | Typ  | Beschreibung |
|------|------|--------------|
| `CH1` | `Bool` | Ausgang, der den Status von Kanal 1 anzeigt (TRUE, wenn korrekt funktionierend). |
| `CH2` | `Bool` | Ausgang, der den Status von Kanal 2 anzeigt (TRUE, wenn korrekt funktionierend). |
| `CH3` | `Bool` | Ausgang, der den Status von Kanal 3 anzeigt (TRUE, wenn korrekt funktionierend). |
| `CH4` | `Bool` | Ausgang, der den Status von Kanal 4 anzeigt (TRUE, wenn korrekt funktionierend). |
| `error` | `Bool` | Fehleranzeige, auf TRUE gesetzt, wenn ein Fehler in der Sequenz erkannt wird. |
| `status` | `Int`  | Statusausgang, der einen detaillierten Fehler- oder Betriebszustandscode angibt. |

---

## Interne Variablen

### Timer

| Name        | Typ       | Beschreibung |
|-------------|-----------|--------------|
| `TIME`      | `TON_TIME` | Der Haupttimer, der die Sequenzzeit steuert. |
| `CHECK`     | `TON_TIME` | Timer, der verwendet wird, um jeden Schritt in der Sequenz zu validieren. |
| `ERROR_TON` | `TON_TIME` | Timer, der Inaktivität verfolgt, um Fehler zu erkennen. |

### Zustandsvariablen

| Name           | Typ           | Beschreibung |
|----------------|---------------|--------------|
| `state`        | `Int`         | Der aktuelle Zustand der Betriebssequenz. |
| `StepStatus`   | `Array[0..9] of Bool` | Boolean-Array, das den Status jedes Schritts in der Sequenz speichert. |
| `ChannelStatus` | `Array[1..4] of Bool` | Boolean-Array, das den Status jedes Kanals speichert (TRUE, wenn korrekt funktionierend). |
| `saved_state`  | `Int`         | Speichert den letzten bekannten Zustand, um Änderungen zu erkennen. |

### Temporäre Variablen

| Name          | Typ  | Beschreibung |
|---------------|------|--------------|
| `i`           | `Int` | Indexvariable für Schleifenoperationen. |
| `status_tmp`  | `Int` | Temporäre Variable, um den Status vor der Ausgabe zu speichern. |

### Konstanten

| Name          | Typ   | Standardwert | Beschreibung |
|---------------|-------|--------------|--------------|
| `StartTimeTrig` | `Time` | `T#500MS`   | Verzögerungszeit, bevor die Sequenz beginnt. |
| `deviation`     | `Time` | `T#10MS`    | Zulässige Zeitabweichung für Zeitprüfungen. |
| `MaxTime`       | `Time` | `T#2S_750MS` | Maximale Zeit, die für die gesamte Sequenz erlaubt ist. |
| `T_Interval`    | `Time` | `T#250MS`   | Zeitintervall zwischen Schritten und Kanalprüfungen. |

---

## Logikstruktur

### Haupttimer (`Start main Timer`-Region)
Dieser Abschnitt startet den Haupttimer, wenn die Sequenz beginnt (`state > 0`). Der Timer zählt die Gesamtdauer der Sequenz und stellt sicher, dass die vordefinierte Maximalzeit (`MaxTime`) nicht überschritten wird.

### Sequenzsteuerung (`Check Sequence`-Region)
Dieser Abschnitt steuert die Betriebsschritte. Die Sequenz beginnt mit der Initialisierung der Schritte (Zustand 0), dann wechseln Pausen und Kanalprüfungen (Zustände 1 bis 9) ab. Der Status jedes Schritts wird gespeichert, und die entsprechenden Kanäle werden aktualisiert.

- **Initialisierung (Zustand 0):** Alle Schritte werden zurückgesetzt und die Sequenz beginnt.
- **Zustände 1 bis 9:** Abwechselnde Überprüfung von Pausen und Kanälen mit den entsprechenden Timern für jeden Schritt.
- **Abschluss (Zustand 10):** Markiert den Abschluss der Sequenz und setzt den Zustand zurück.

### Fehlererkennung (`Error Handling`-Region)
Dieser Abschnitt erkennt Fehler basierend auf der Inaktivität oder einem abnormalen Ablauf der Sequenz. Fehler werden mit Statuscodes klassifiziert:

- `0`: Kein Fehler.
- `1`: Kanalfehler.
- `2`: Pausenfehler.
- `3`: Sequenz nicht gestartet.
- `4`: Sequenz unerwartet gestoppt.

Timer werden verwendet, um den Sequenzfortschritt zu überwachen. Wenn der Zustand sich innerhalb der erlaubten Zeit nicht ändert, wird ein Fehler ausgelöst.

### Ausgabezuweisung (`Assign Outputs`-Region)
Die letzte Region weist den Status der Kanäle den entsprechenden Ausgängen (`CH1`, `CH2`, `CH3`, `CH4`) zu und setzt die `error`-Flagge, falls ein Problem erkannt wird. Der `status`-Ausgang spiegelt den spezifischen Fehler- oder Betriebszustandscode wider.

---

## Fehlerstatuscodes

Die folgenden Fehlerstatuscodes werden verwendet, um Probleme in der Sequenz zu diagnostizieren:

| Statuscode | Beschreibung |
|------------|--------------|
| `0`        | Kein Fehler. |
| `1`        | Kanalfehler (ein Kanal wurde nicht erfolgreich abgeschlossen). |
| `2`        | Pausenfehler (eine Pause wurde nicht wie erwartet abgeschlossen). |
| `3`        | Sequenz wurde nicht gestartet. |
| `4`        | Sequenz wurde unerwartet gestoppt. |

---

## Ablauf der Sequenz

1. **Initialisierung (Zustand 0):** Die Sequenz beginnt mit dem Zurücksetzen aller Schritte.
2. **Zwischenzustände (1-9):** Abwechselnde Überprüfung von Pausen und Kanälen.
   - Ungerade Zustände repräsentieren Pausen.
   - Gerade Zustände überprüfen den Status der entsprechenden Kanäle.
3. **Abschluss (Zustand 10):** Die Sequenz wird abgeschlossen, Fehlererkennung tritt in Kraft, und der Zustand wird zurückgesetzt.
4. **Fehlerbehandlung:** Überwacht Zustandsänderungen und löst Fehlercodes bei Unregelmäßigkeiten aus.

