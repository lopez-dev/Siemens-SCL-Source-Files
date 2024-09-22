# Dokumentation zum Timer-Management in SCL

## Übersicht

Diese Dokumentation beschreibt die Verwaltung und Verwendung von Timern in einer SPS-Anwendung, die mit SCL (Structured Control Language) programmiert ist. Die Timer werden in einer strukturierten Weise verwaltet, um verschiedene Zustände zu verfolgen und entsprechende Aktionen auszuführen.

**Hinweis:** Der hier dargestellte Code ist in Structured Control Language (SCL) geschrieben, was bedeutet, dass er direkt in Siemens TIA Portal importiert und verwendet werden kann. SCL ermöglicht eine klare und strukturierte Programmierung von speicherprogrammierbaren Steuerungen (SPS).

## Timer-Zustände

Die Timer können sich in einem der folgenden vier Zustände befinden:

1. **State 0 - Off (Aus):**
   - Der Timer ist inaktiv. Er wurde entweder noch nicht gestartet oder wurde zurückgesetzt.

2. **State 1 - Running (Laufend):**
   - Der Timer läuft. Er wurde gestartet und zählt die Zeit basierend auf der definierten `init_time` hoch.

3. **State 2 - Held (Gehalten):**
   - Der Timer wurde angehalten. Er befindet sich in einem Zustand der Pause und die Zeitzählung ist gestoppt, kann jedoch fortgesetzt werden.

4. **State 3 - Elapsed (Abgelaufen):**
   - Der Timer hat die definierte `init_time` erreicht. Die Zeit ist abgelaufen und der Timer signalisiert diesen Zustand.

## Funktionen des Timer-Managements

### 1. TimerCheck

- **Beschreibung:** Diese Funktion prüft den aktuellen Zustand eines Timers und gibt diesen Zustand zurück. Sie ist nützlich, um den Status eines Timers abzufragen, um zu entscheiden, welche Aktion als nächstes durchgeführt werden soll.

- **Anwendung:** Wird verwendet, um festzustellen, ob ein Timer gerade läuft, angehalten ist, abgelaufen ist oder ausgeschaltet ist.

### 2. TimerReset

- **Beschreibung:** Diese Funktion setzt einen Timer zurück, indem sie den Zustand des Timers auf 0 (Aus) stellt und die aktuelle Zeit auf null zurücksetzt. Außerdem wird das `time_elapsed`-Flag auf `false` gesetzt.

- **Anwendung:** Wird verwendet, um einen Timer zu initialisieren oder ihn nach einem Lauf auf den Ausgangszustand zurückzusetzen.

### 3. TimerStart

- **Beschreibung:** Diese Funktion startet einen Timer, indem sie den Zustand auf 1 (Laufend) setzt und die initiale Zeit (`init_time`) mit einem vorgegebenen Wert festlegt.

- **Anwendung:** Wird verwendet, um einen neuen Timerlauf zu beginnen. Der Timer beginnt von diesem Zeitpunkt an, die Zeit hochzuzählen.

### 4. TimerElapsed

- **Beschreibung:** Diese Funktion überprüft, ob der Timer abgelaufen ist, indem sie den Zustand des Timers mit 3 (Abgelaufen) vergleicht.

- **Anwendung:** Wird verwendet, um festzustellen, ob die eingestellte Zeit abgelaufen ist und eine entsprechende Aktion ausgelöst werden sollte.

### 5. TimerHold

- **Beschreibung:** Diese Funktion hält einen laufenden Timer an, indem sie den Zustand des Timers auf 2 (Gehalten) setzt.

- **Anwendung:** Wird verwendet, um die Zeitzählung eines Timers vorübergehend zu stoppen. Nützlich, wenn eine zeitbasierte Aktion unterbrochen werden muss.

### 6. TimerResume

- **Beschreibung:** Diese Funktion setzt einen angehaltenen Timer fort, indem sie den Zustand zurück auf 1 (Laufend) setzt.

- **Anwendung:** Wird verwendet, um die Zeitzählung eines angehaltenen Timers fortzusetzen, sodass er seine Arbeit abschließen kann.

### 7. TimerManager

- **Beschreibung:** Diese Funktion ist das zentrale Element der Timerverwaltung. Sie überprüft den Zustand aller Timer, zählt die Zeit für laufende Timer hoch und setzt den Zustand auf 3 (Abgelaufen), wenn die eingestellte Zeit erreicht ist. Die Funktion wird regelmäßig in einem zyklischen Interrupt-OB aufgerufen.

- **Anwendung:** Wird verwendet, um die Timer zu verwalten und sicherzustellen, dass alle Timer korrekt laufen und überwacht werden. Es wird empfohlen, diese Funktion in regelmäßigen Zeitintervallen auszuführen, die durch einen zyklischen Interrupt-OB gesteuert werden.

### Wichtig: Zyklischer Interrupt-OB

- **Erstellung eines zyklischen Interrupt-OBs:** Um die `TimerManager`-Funktion korrekt auszuführen, muss ein zyklischer Interrupt-OB manuell im TIA Portal erstellt werden.
- **Einstellung der Interrupt-Zeit:** Die Interrupt-Zeit für diesen OB sollte auf **10.000 Mikrosekunden** (10 ms) eingestellt werden. Dies gewährleistet eine genaue und zuverlässige Timerverwaltung.

### 8. TimerTON

- **Beschreibung:** Diese Funktion implementiert eine Einschaltverzögerung. Sie startet den Timer, wenn ein Eingangssignal aktiv ist, und hält den Timer an, wenn das Signal inaktiv wird.

- **Anwendung:** Wird verwendet, um zeitgesteuerte Prozesse zu kontrollieren, die eine Verzögerung beim Einschalten erfordern.

## Anmerkungen

- Alle Timer-Funktionen sind so ausgelegt, dass sie in zyklischen Prozessumgebungen arbeiten, in denen die Zeit auf Basis wiederholter Zyklusaufrufe verwaltet wird.
- Die Funktionen und Zustände sind so gestaltet, dass sie flexible und präzise Steuerungen in Automatisierungssystemen ermöglichen, die auf Siemens SPS und dem TIA Portal basieren.
- Stellen Sie sicher, dass der zyklische Interrupt-OB korrekt konfiguriert ist, um eine reibungslose Funktion der Timer-Management-Logik zu gewährleisten.

---

Diese Dokumentation bietet eine umfassende Übersicht über die Verwendung und Verwaltung von Timern in einem SPS-Programm. Sie richtet sich an Ingenieure und Programmierer, die mit SCL und der Programmierung von Siemens SPS arbeiten.
