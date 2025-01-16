# Backup und Restore Funktionsblock Dokumentation

## Übersicht
Der Funktionsblock `Backup` wurde entwickelt, um Datenblöcke (DBs) in einer SPS zu sichern und wiederherzustellen. Die Implementierung enthält zwei Hauptbereiche: den Backup-Bereich (zum Speichern von Daten) und den Restore-Bereich (zum Wiederherstellen von Daten). Es werden optimierte Datenzugriffsmechanismen verwendet.

## Hauptbestandteile

### 1. Datentypen
#### `backupMainHeaderUDT`
Enthält die Steuerparameter für den gesamten Backup- und Restore-Prozess:
- **`save` (Bool):** Startet den Backup-Prozess, wenn auf `TRUE` gesetzt.
- **`restore` (Bool):** Startet den Restore-Prozess, wenn auf `TRUE` gesetzt.
- **`wr_counter` (Int):** Zähler für die Anzahl der abgeschlossenen Schreiboperationen.
- **`rd_counter` (Int):** Zähler für die Anzahl der abgeschlossenen Leseoperationen.
- **`last_BLK` (Int):** Nummer des letzten Blocks im Prozess.

#### `backupBLKHeaderUDT`
Steuert die Operationen für einzelne Datenblöcke:
- **`save` (Bool):** Gibt an, ob der aktuelle Block gespeichert werden soll.
- **`restore` (Bool):** Gibt an, ob der aktuelle Block wiederhergestellt werden soll.
- **`write` (Bool):** Startet den Schreibvorgang.
- **`wr_busy` (Bool):** Zeigt an, ob ein Schreibvorgang läuft.
- **`wr_process` (Bool):** Markiert den Schreibprozess als aktiv.
- **`wr_done` (Bool):** Markiert den Schreibvorgang als abgeschlossen.
- **`wr_ret` (Int):** Rückgabewert der Schreiboperation (z. B. Fehlercode).
- **`wr_ret_mem` (Int):** Zwischenspeicherung des Rückgabewertes der letzten Operation.
- **`read` (Bool):** Startet den Lesevorgang.
- **`rd_busy` (Bool):** Zeigt an, ob ein Lesevorgang läuft.
- **`rd_process` (Bool):** Markiert den Leseprozess als aktiv.
- **`rd_done` (Bool):** Markiert den Lesevorgang als abgeschlossen.
- **`rd_ret` (Int):** Rückgabewert der Leseoperation (z. B. Fehlercode).
- **`rd_ret_mem` (Int):** Zwischenspeicherung des Rückgabewertes der letzten Operation.

### 2. Funktionsblock `Backup`

#### Eingabevariablen (`VAR_INPUT`)
- **`Nr` (Int):** Identifiziert den aktuellen Block im Prozess.
- **`timestamp` (LDT):** Zeitstempel für die Sicherungs- oder Wiederherstellungsoperation.

#### Ein-/Ausgabevariablen (`VAR_IN_OUT`)
- **`main_header` (backupMainHeaderUDT):** Enthält die Hauptsteuerparameter.
- **`BLK_header` (backupBLKHeaderUDT):** Enthält die Steuerparameter für den aktuellen Block.
- **`DB_BLK` (Variant):** Referenz auf den Datenblock, der gesichert oder wiederhergestellt werden soll.

#### Temporäre Variablen (`VAR_TEMP`)
- **`SD1_wr` (UInt):** Zwischenspeicherung des Schreibstatus.
- **`SD1_rd` (UInt):** Zwischenspeicherung des Lesestatus.
- **`wr` (UInt):** Parameter für Schreiboperationen.
- **`rd` (UInt):** Parameter für Leseoperationen.

### Backup-Prozess (`REGION backup`)
1. **Initialisierung:**
   - Der Schreibzähler (`wr_counter`) wird initialisiert, falls er auf `0` steht.
   - Der Prozess wird nur gestartet, wenn keine Wiederherstellung (`restore`) aktiv ist.

2. **Schreibvorgang:**
   - Das Flag `save` wird aktiviert, wenn die Bedingungen stimmen.
   - Sobald `save` aktiv ist, wird der Schreibprozess gestartet (`wr_process`) und `write` aktiviert.
   - Die Daten werden mit der Funktion `WRIT_DBL` in den Ziel-Datenblock (`DB_BLK`) geschrieben.

3. **Abschluss:**
   - Nach erfolgreichem Abschluss wird der Zähler (`wr_counter`) erhöht und der Prozess beendet.
   - Der Status wird zurückgesetzt, um den nächsten Block vorzubereiten.

### Restore-Prozess (`REGION restore`)
1. **Initialisierung:**
   - Der Restore-Zähler (`rd_counter`) wird initialisiert, falls er auf `0` steht.
   - Der Prozess wird nur gestartet, wenn der aktuelle Block (`Nr`) wiederhergestellt werden soll.

2. **Lesevorgang:**
   - Das Flag `restore` wird aktiviert, wenn die Bedingungen stimmen.
   - Sobald `restore` aktiv ist, wird der Leseprozess gestartet (`rd_process`) und `read` aktiviert.
   - Die Daten werden mit der Funktion `READ_DBL` aus dem Quell-Datenblock (`DB_BLK`) gelesen.

3. **Abschluss:**
   - Nach erfolgreichem Abschluss wird der Zähler (`rd_counter`) erhöht und der Prozess beendet.
   - Der Status wird zurückgesetzt, um den nächsten Block vorzubereiten.

## Wichtige Funktionen
- **`WRIT_DBL`:** Schreibt die Daten in einen Ziel-Datenblock.
- **`READ_DBL`:** Liest die Daten aus einem Quell-Datenblock.
- **Alarmausgabe:** Verwendet `Backup_Return` und `Restore_Return`, um den Status der Operationen zu melden.

## Hinweise
- Der Funktionsblock verwendet optimierte Datenzugriffe (`S7_Optimized_Access := 'TRUE'`).
- Die Rückgabewerte und Statussignale bieten eine einfache Diagnose und Überwachung des Prozesses.
- Der Prozess ist modular aufgebaut und kann leicht angepasst oder erweitert werden.

## Verwendung
1. Setzen Sie die Variablen `save` oder `restore` im `main_header` auf `TRUE`, um den Backup- oder Restore-Prozess zu starten.
2. Stellen Sie sicher, dass der Datenblock `DB_BLK` korrekt referenziert wird.
3. Überwachen Sie die Statussignale (`wr_done`, `rd_done`) und Rückgabewerte (`wr_ret`, `rd_ret`), um den Fortschritt zu verfolgen.
