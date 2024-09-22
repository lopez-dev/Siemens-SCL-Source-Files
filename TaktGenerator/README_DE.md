# FB_Taktgenerator Dokumentation

## Übersicht

Der **FB_Taktgenerator** ist ein Funktionsbaustein, der verschiedene frequenzbasierte Signale (Taktsignale) generiert und die positiven Flanken (steigende Flanken) für jede Frequenz erkennt. Die Frequenzen reichen von 10Hz bis 0,2Hz. Er kann in jedem Teil des SPS-Programms verwendet werden, und die Ausgänge (Taktsignale und Flankensignale) können benutzerdefinierten UDTs in den jeweiligen Datenbausteinen (DBs) zugewiesen werden.

### Beschreibung der Eingangs- und Ausgangsvariablen:

- **Takt_Signals (`udt_tg_takt`)**: Diese Struktur enthält die Ausgangssignale für die verschiedenen Frequenzen (Taktsignale).
- **Edge_Signals (`udt_tg_takt`)**: Diese Struktur enthält die Signale zur Erkennung der positiven Flanken (Flankensignale).

### Interne Variablen:

- **Internal_TaktSignals**: Speichert den Zustand (True/False) der internen Taktsignale.
- **Pulse_Trigger**: Speichert die Pulse, die beim Start jedes Taktsignals erzeugt werden.
- **Edge_Flag**: Verfolgt den Signalzustand zur Flankenerkennung.
- **StartTime**: Speichert die Startzeit jedes Taktsignals zur Berechnung des Intervalls.
- **Takt_Intervals**: Speichert die vordefinierten Intervalle für jedes Taktsignal.

### Konstanten:

- **MAX_TIME**: Maximale Zeit, um einen Überlauf der Zeit zu handhaben.
- **const_100MS** - **const_5S**: Konstanten, die die Zeitintervalle für jedes Taktsignal definieren.

---

## Verwendung

Um den **FB_Taktgenerator** zu verwenden, befolge diese Schritte:

1. **UDTs definieren**: Erstelle eine Instanz von `udt_tg_takt` in deinem Datenbaustein (DB), in dem du die Ausgangssignale speichern möchtest.
   
2. **Weise die UDTs den Ausgangsvariablen zu**: Weisen Sie die UDT-Instanzen den Ausgängen `Takt_Signals` und `Edge_Signals` des **FB_Taktgenerator** zu.

3. **FB aufrufen**: Der **FB_Taktgenerator** kann überall im Programm aufgerufen werden, und er wird die Taktsignale basierend auf der Systemuhr erzeugen.

### Beispiel

1. **Definiere eine UDT-Instanz in einem DB:**

```plc
DATA_BLOCK "DB_TaktOutputs"
    udt_tg_takt Takt_Output;
    udt_tg_takt Edge_Output;
END_DATA_BLOCK
```

2. **Weise die UDTs im FB-Aufruf zu:**

```plc
CALL "FB_Taktgenerator" (
    Takt_Signals := "DB_TaktOutputs".Takt_Output,
    Edge_Signals := "DB_TaktOutputs".Edge_Output
);
```

---

## Detaillierte Funktionalität

### Time_Read Region

```plc
REGION Time_Read
    // Lese die aktuelle Systemzeit und konvertiere sie in das TIME-Format
    #ret := RD_SYS_T(#CurrentTime_DTL);
    #CurrentTime := TOD_TO_TIME(IN := DTL_TO_TOD(IN := #CurrentTime_DTL));
END_REGION
```

Diese Region liest die aktuelle Systemzeit und konvertiert sie in das TIME-Format, um die Zeitintervalle für die Taktsignale zu berechnen.

### Interval_Assignment Region

```plc
REGION Interval_Assignment
    // Weist die vordefinierten Taktintervalle den entsprechenden Array-Elementen zu
    FOR #i := 0 TO 9 DO
        CASE #i OF
            0: #Takt_Intervals[#i] := #const_100MS;
            1: #Takt_Intervals[#i] := #const_200MS;
            2: #Takt_Intervals[#i] := #const_400MS;
            3: #Takt_Intervals[#i] := #const_500MS;
            4: #Takt_Intervals[#i] := #const_800MS;
            5: #Takt_Intervals[#i] := #const_1S;
            6: #Takt_Intervals[#i] := #const_1S_600MS;
            7: #Takt_Intervals[#i] := #const_2S;
            8: #Takt_Intervals[#i] := #const_4S;
            9: #Takt_Intervals[#i] := #const_5S;
        END_CASE;
    END_FOR;
END_REGION
```

Diese Region weist die Zeitintervalle (z.B. 100ms, 200ms) dem Array `Takt_Intervals` zu, welches zur Steuerung des Taktsignals verwendet wird.

### Takt_Signal_Update Region

```plc
REGION Takt_Signal_Update
    // Iteriere über alle Frequenzen und aktualisiere jedes Taktsignal basierend auf dem Zeitunterschied
    FOR #i := 0 TO 9 DO
        IF (#CurrentTime >= #StartTime[#i]) THEN
            #TimeDifference := #CurrentTime - #StartTime[#i];
        ELSE
            #TimeDifference := (#MAX_TIME - #StartTime[#i]) + #CurrentTime;  // Überlauf handhaben
        END_IF;

        // Schalte das Taktsignal um, wenn der Zeitunterschied das Intervall überschreitet
        IF (#TimeDifference >= #Takt_Intervals[#i]) THEN
            #Internal_TaktSignals[#i] := NOT #Internal_TaktSignals[#i];
            #StartTime[#i] := #CurrentTime;
        END_IF;
    END_FOR;
END_REGION
```

Diese Region berechnet den Zeitunterschied für jedes Taktsignal und schaltet das Signal um, wenn das Zeitintervall verstrichen ist. Sie berücksichtigt einen möglichen Überlauf des TIME-Werts.

### Takt_Signal_Assignment Region

```plc
REGION Takt_Signal_Assignment
    // Weise die internen Taktsignale den Ausgangsstrukturfeldern zu
    FOR #i := 0 TO 9 DO
        CASE #i OF
            0: #Takt_Signals.Takt_10Hz := #Internal_TaktSignals[0];
            1: #Takt_Signals.Takt_5Hz := #Internal_TaktSignals[1];
            2: #Takt_Signals.Takt_2_5Hz := #Internal_TaktSignals[2];
            3: #Takt_Signals.Takt_2Hz := #Internal_TaktSignals[3];
            4: #Takt_Signals.Takt_1_25Hz := #Internal_TaktSignals[4];
            5: #Takt_Signals.Takt_1Hz := #Internal_TaktSignals[5];
            6: #Takt_Signals.Takt_0_625Hz := #Internal_TaktSignals[6];
            7: #Takt_Signals.Takt_0_5Hz := #Internal_TaktSignals[7];
            8: #Takt_Signals.Takt_0_25Hz := #Internal_TaktSignals[8];
            9: #Takt_Signals.Takt_0_2Hz := #Internal_TaktSignals[9];
        END_CASE;
    END_FOR;
END_REGION
```

Diese Region weist die internen Taktsignale der Ausgabenstruktur (`Takt_Signals`) zu, die dann vom Benutzer verwendet werden können.

### Edge_Detection Region

```plc
REGION Edge_Detection
    // Erkenne Flanken (positive Flanke) für jedes Taktsignal
    FOR #i := 0 TO 9 DO
        // Erkenne eine positive Flanke (Taktsignal wechselt von FALSE zu TRUE)
        #Pulse_Trigger[#i] := #Internal_TaktSignals[#i] XOR #Edge_Flag[#i];

        // Aktualisiere das Flanken-Flag für den nächsten Zyklus
        #Edge_Flag[#i] := #Internal_TaktSignals[#i];
    END_FOR;
END_REGION
```

Diese Region erkennt die positive Flanke (steigende Flanke) für jedes Taktsignal. Der `Pulse_Trigger` wird auf TRUE gesetzt, wenn eine positive Flanke erkannt wird.

### P_Trig_Assignment Region

```plc
REGION P_Trig_Assignment
    // Weise die Pulse Trigger Signale den entsprechenden Ausgangsfeldern zu
    FOR #i := 0 TO 9 DO
        CASE #i OF
            0: #Edge_Signals.Takt_10Hz := #Pulse_Trigger[#i];
            1: #Edge_Signals.Takt_5Hz := #Pulse_Trigger[#i];
            2: #Edge_Signals.Takt_2_5Hz := #Pulse_Trigger[#i];
            3: #Edge_Signals.Takt_2Hz := #Pulse_Trigger[#i];
            4: #Edge_Signals.Takt_1_25Hz := #Pulse_Trigger[#i];
            5

: #Edge_Signals.Takt_1Hz := #Pulse_Trigger[#i];
            6: #Edge_Signals.Takt_0_625Hz := #Pulse_Trigger[#i];
            7: #Edge_Signals.Takt_0_5Hz := #Pulse_Trigger[#i];
            8: #Edge_Signals.Takt_0_25Hz := #Pulse_Trigger[#i];
            9: #Edge_Signals.Takt_0_2Hz := #Pulse_Trigger[#i];
        END_CASE;
    END_FOR;
END_REGION
```

Diese Region weist die `Pulse_Trigger`-Werte (erkannten Flanken) der Ausgabenstruktur (`Edge_Signals`) zu, die vom Benutzer genutzt werden können.

---

## Fazit

Der **FB_Taktgenerator** bietet eine einfache und flexible Möglichkeit, Taktsignale zu generieren und positive Flanken für mehrere Frequenzen zu erkennen. Durch die Zuweisung von UDT-Instanzen zu den Ausgangsvariablen können diese Signale einfach in Ihr Projekt integriert werden.