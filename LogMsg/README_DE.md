# Dokumentation des SCL-Codes für Siemens TIA Portal

## Einleitung

Diese Dokumentation beschreibt den SCL-Code für das Siemens TIA Portal, der Funktionen für das Data Logging in einer SPS-Anwendung bereitstellt. Der Code ermöglicht das Erstellen, Schreiben, Schließen, Öffnen und Löschen von Datenprotokollen und verwaltet Meldungen über einen Puffer.

## Inhaltsverzeichnis

- [Datentypdefinition (`typ_DataLogging_DB`)](#datentypdefinition-typ_datalogging_db)
- [Datenbaustein (`DB_LogMsg`)](#datenbaustein-db_logmsg)
- [Funktionen](#funktionen)
  - [`fc_LogMsg_InputBuffer`](#funktion-fc_logmsg_inputbuffer)
  - [`fc_LogMsg_CallEntry`](#funktion-fc_logmsg_callentry)
  - [`fc_LogMsg_NextEntry`](#funktion-fc_logmsg_nextentry)
  - [`fc_LogMsg_inputBool`](#funktion-fc_logmsg_inputbool)
- [Funktionsbaustein (`FB_LogMsg`)](#funktionsbaustein-fb_logmsg)
- [Hinweise zur Verwendung](#hinweise-zur-verwendung)
- [Schlussfolgerung](#schlussfolgerung)

## Datentypdefinition (`typ_DataLogging_DB`)

Der Datentyp `typ_DataLogging_DB` definiert die Struktur für das Data Logging. Er enthält alle notwendigen Variablen und Strukturen zur Verwaltung von Log-Einträgen und -Operationen.

### Strukturdefinition

```scl
TYPE "typ_DataLogging_DB"
VERSION : 0.1
STRUCT
    LogEntryIndex : UInt;   // Aktuelle Log-Eintrag-Position
    name : String := '';    // Name des Log-Eintrags
    nextPosInStack : UInt := 1;  // Nächster Log-Datei-Index
    newName : String := 'DataLog_';  // Name für neue Log-Dateien
    logID : DWord;  // ID des Log-Eintrags
    TurnAllOn : Bool;  // Aktiviert alle Logging-Tags
    TurnAlloff : Bool;  // Deaktiviert alle Logging-Tags
    reset : Bool;  // Reset-Befehl zum Quittieren von Fehlern und Zurücksetzen des Fehlerzählers
    deleteAll : Bool;  // Löscht alle Log-Einträge
    dataLogEntries : Array[0..7] of Struct
        name : String;  // Name des Log-Eintrags
        ID : DWord;     // ID des Log-Eintrags
        DLclosed : Bool;  // Gibt an, ob der Log-Eintrag geschlossen ist
    END_STRUCT;
    DLcreate : Struct
        execute : Bool;  // Ausführen
        done : Bool;     // Abgeschlossen
        busy : Bool;     // Läuft
        error : Bool;    // Fehler
        status : Word;   // Aktueller Status
        memStatusMsg : String[200];  // Statusmeldung in Textform
        memStatus : Word;  // Zuletzt gespeicherte Statusmeldung
        dlogCreated : Bool;  // Log-Eintrag erfolgreich erstellt
    END_STRUCT;
    // Weitere Strukturen: DLdelete, DLwrite, DLnewfile, DLclose, DLopen
    // Parameterstruktur für Logging-Einstellungen
    // Buffer für Meldungen
    // Statistik-Variablen
    // Aktueller zu bearbeitender Log-Datensatz (myData)
END_STRUCT;
END_TYPE
```

### Beschreibung der Hauptvariablen

- **LogEntryIndex**: Hält den Index des aktuellen Log-Eintrags.
- **name**: Name des aktuellen Log-Eintrags.
- **nextPosInStack**: Index für die nächste Log-Datei im Stack.
- **newName**: Generierter Name für neue Log-Dateien.
- **logID**: ID des aktuellen Log-Eintrags.
- **TurnAllOn** / **TurnAlloff**: Steuert die Aktivierung oder Deaktivierung aller Logging-Tags.
- **reset**: Setzt Fehlerzustände zurück und quittiert Fehlermeldungen.
- **deleteAll**: Befehl zum Löschen aller Log-Einträge.
- **dataLogEntries**: Array, das Informationen zu jedem Log-Eintrag speichert.
- **DLcreate** bis **DLopen**: Strukturen zur Verwaltung verschiedener Data Logging-Operationen wie Erstellen, Löschen, Schreiben usw.
- **Parameter**: Enthält Einstellungen für das Logging, wie z.B. maximale Anzahl von Fehlern, Puffergröße, Logging-Tags usw.
- **Buffer**: Puffer zur Speicherung von Meldungen, die geloggt werden sollen.
- **Statistik**: Hält statistische Informationen wie maximale Pufferbelegung und Fehlerzähler.
- **myData**: Aktueller Datensatz, der in das Log geschrieben wird.

## Datenbaustein (`DB_LogMsg`)

Der Datenbaustein `DB_LogMsg` ist eine Instanz des Typs `typ_DataLogging_DB` und dient zur Speicherung der Laufzeitdaten und Parameter für das Data Logging.

### Initialisierung

```scl
DATA_BLOCK "DB_LogMsg"
{ S7_Optimized_Access := 'TRUE' }
VERSION : 0.1
"typ_DataLogging_DB"

BEGIN
    Parameter.LoggingTag[0].On := TRUE;
    Parameter.LoggingTag[0].Tag := 'System';
    Parameter.LoggingTag[1].On := TRUE;
    Parameter.LoggingTag[1].Tag := 'Infeed';
    Parameter.LoggingTag[2].On := TRUE;
    Parameter.LoggingTag[2].Tag := 'Outfeed';
    Parameter.LoggingTag[3].On := TRUE;
    Parameter.LoggingTag[3].Tag := 'Loader';
    Parameter.LoggingTag[4].On := TRUE;
    Parameter.LoggingTag[4].Tag := 'Unloader';
    Parameter.LoggingTag[5].On := TRUE;
    Parameter.LoggingTag[5].Tag := 'Deck';
    Parameter.LoggingTag[6].On := TRUE;
    Parameter.maxPosEntry := 8;
    Parameter.newRECORDS := 2000;
    Parameter.NeNamePrefix := 'DataLog_';
END_DATA_BLOCK
```

### Beschreibung

- **Parameter.LoggingTag**: Array zur Konfiguration der Logging-Tags. Jedes Element aktiviert das Logging für einen spezifischen Bereich (z.B. 'System', 'Infeed', 'Outfeed' usw.).
- **Parameter.maxPosEntry**: Maximale Anzahl von Log-Einträgen im Stack.
- **Parameter.newRECORDS**: Maximale Anzahl von Datensätzen für neue Log-Dateien.
- **Parameter.NeNamePrefix**: Präfix für neue Log-Dateien.

## Funktionen

### Funktion `fc_LogMsg_InputBuffer`

Diese Funktion fügt neue Meldungen in den Meldungspuffer ein und verwaltet das Verschieben bestehender Meldungen.

#### Deklaration

```scl
FUNCTION "fc_LogMsg_InputBuffer" : Void
{ S7_Optimized_Access := 'TRUE' }
VERSION : 0.1
VAR_INPUT 
    Logging_i : Int;
    Text : String;
END_VAR

VAR_TEMP 
    i : Int;
END_VAR
```

#### Beschreibung

- **Logging_i**: Index des Logging-Tags.
- **Text**: Der zu loggende Meldungstext.
- **i**: Temporäre Variable für Schleifendurchläufe.

#### Funktionsweise

- Überprüft, ob der gegebene `Logging_i` innerhalb der maximalen Anzahl von Logging-Tags liegt und ob das Logging für diesen Tag aktiviert ist.
- Verschiebt alle bestehenden Meldungen im Puffer um eine Position nach unten.
- Fügt die neue Meldung an der ersten Position des Puffers ein.
- Setzt die `unwritten`-Flag der neuen Meldung auf `TRUE`, um anzuzeigen, dass sie noch nicht in das Log geschrieben wurde.

#### Codeauszug

```scl
IF #Logging_i <= "DB_LogMsg".Parameter.MaxLoggingTags -1 THEN
    IF "DB_LogMsg".Parameter.LoggingTag[#Logging_i].On THEN
        FOR #i := "DB_LogMsg".Parameter.BufferSize - 1 TO 1 BY -1 DO
            "DB_LogMsg".Buffer[#i] := "DB_LogMsg".Buffer[#i - 1];
        END_FOR;
        
        "DB_LogMsg".Buffer[0].id := "DB_LogMsg".Parameter.NextID;
        "DB_LogMsg".Parameter.NextID := "DB_LogMsg".Parameter.NextID + 1;
        "DB_LogMsg".Buffer[0].Text := #Text;
        "DB_LogMsg".Buffer[0].tag := "DB_LogMsg".Parameter.LoggingTag[#Logging_i].Tag;
        "DB_LogMsg".Buffer[0].unwritten := TRUE;
    END_IF;
END_IF;
```

### Funktion `fc_LogMsg_CallEntry`

Diese Funktion aktualisiert den Namen und die ID des aktuellen Log-Eintrags basierend auf dem ausgewählten `LogEntryIndex`.

#### Deklaration

```scl
FUNCTION "fc_LogMsg_CallEntry" : Void
{ S7_Optimized_Access := 'TRUE' }
VERSION : 0.1
VAR_TEMP 
    tempEntry : UInt;
END_VAR
```

#### Funktionsweise

- Setzt den `name` und die `logID` aus dem aktuell ausgewählten Log-Eintrag.

#### Codeauszug

```scl
"DB_LogMsg".name := "DB_LogMsg".dataLogEntries["DB_LogMsg".LogEntryIndex].name;
"DB_LogMsg".logID := "DB_LogMsg".dataLogEntries["DB_LogMsg".LogEntryIndex].ID;
```

### Funktion `fc_LogMsg_NextEntry`

Diese Funktion berechnet den nächsten `LogEntryIndex` und aktualisiert `nextPosInStack`, um stets einen Schritt vor dem aktuellen Index zu sein.

#### Deklaration

```scl
FUNCTION "fc_LogMsg_NextEntry" : Void
{ S7_Optimized_Access := 'TRUE' }
VERSION : 0.1
VAR_TEMP 
    nextIndex : UInt;
END_VAR
```

#### Funktionsweise

- Erhöht den `LogEntryIndex` um 1 und setzt ihn auf 0 zurück, wenn das Maximum erreicht ist.
- Aktualisiert `nextPosInStack` entsprechend.

#### Codeauszug

```scl
nextIndex := "DB_LogMsg".LogEntryIndex + 1;

IF nextIndex > ("DB_LogMsg".Parameter.maxPosEntry - 1) THEN
    nextIndex := 0;
END_IF;

"DB_LogMsg".LogEntryIndex := nextIndex;

"DB_LogMsg".nextPosInStack := "DB_LogMsg".LogEntryIndex + 1;

IF "DB_LogMsg".nextPosInStack > ("DB_LogMsg".Parameter.maxPosEntry -1) THEN
    "DB_LogMsg".nextPosInStack := 0;
END_IF;
```

### Funktion `fc_LogMsg_inputBool`

Diese Funktion überwacht Zustandsänderungen von Booleschen Eingängen und loggt diese.

#### Deklaration

```scl
FUNCTION "fc_LogMsg_inputBool" : Void
{ S7_Optimized_Access := 'TRUE' }
VERSION : 0.1
VAR_INPUT 
    InputState : Bool;
    ActualState : Bool;
    Text : String;
    LogId : Int;
END_VAR
```

#### Beschreibung

- **InputState**: Aktueller Zustand des Eingangs.
- **ActualState**: Vorheriger Zustand des Eingangs.
- **Text**: Text, der geloggt werden soll.
- **LogId**: ID des Logging-Tags.

#### Funktionsweise

- Wenn der Eingang von `FALSE` auf `TRUE` wechselt, wird eine Meldung mit `Text TRUE` geloggt.
- Wenn der Eingang von `TRUE` auf `FALSE` wechselt, wird eine Meldung mit `Text FALSE` geloggt.

#### Codeauszug

```scl
IF #InputState AND NOT #ActualState THEN
    "fc_LogMsg_InputBuffer"(Logging_i := #LogId,
                            Text := CONCAT(IN1:= #Text, IN2:=' TRUE'));
ELSIF NOT #InputState AND #ActualState THEN
    "fc_LogMsg_InputBuffer"(Logging_i := #LogId,
                            Text := CONCAT(IN1:= #Text, IN2:=' FALSE'));
END_IF;
```

## Funktionsbaustein (`FB_LogMsg`)

Der Funktionsbaustein `FB_LogMsg` ist das Herzstück des Data Logging-Systems und verwaltet alle Operationen wie Erstellen, Schreiben, Schließen und Löschen von Datenprotokollen.

### Deklaration

```scl
FUNCTION_BLOCK "FB_LogMsg"
{ S7_Optimized_Access := 'TRUE' }
VERSION : 0.1
VAR 
    DataLogCreate_Instance : DataLogCreate;
    DataLogClose_Instance : DataLogClose;
    DataLogWrite_Instance : DataLogWrite;
    DataLogOpen_Instance : DataLogOpen;
    DataLogNewFile_Instance : DataLogNewFile;
    DataLogDelete_Instance : DataLogDelete;
    LoggingFlag : Bool;
    Error_flag : Bool;
    LoggingOn : Bool;
    Deleted : Array[0..7] of Bool;
    DeleteState : UInt;
    i2 : Int;
END_VAR

VAR_TEMP 
    i : Int;
    AllTagOff : Bool;
    LEnr : Array[0..7] of DWord;
END_VAR

VAR CONSTANT 
    EMPTYSTRING : String;
    LENr0 : DWord := 16#7200_1000;
    LENr1 : DWord := 16#7200_1002;
    // Weitere Konstanten bis LENr7
END_VAR
```

### Hauptbereiche des Funktionsbausteins

#### 1. **Info**

- Enthält allgemeine Informationen und Hinweise zur Verwendung des Loggings.
- Wichtig für die korrekte Konfiguration und Nutzung des Systems.

#### 2. **Reset Status**

- Setzt alle gespeicherten Statusmeldungen zurück, wenn ein neuer Befehl eingegeben wird.

#### 3. **Buffer Manager**

- Verwaltert den Meldungspuffer.
- Überprüft, ob der Puffer voll ist, und generiert Fehlermeldungen bei Überlauf.
- Verwaltet das automatische Stoppen des Loggings bei zu vielen Fehlern.
- Überträgt Meldungen aus dem Puffer in das Datenprotokoll.

#### 4. **DataLogCreate**

- Verwaltet die Erstellung neuer Datenprotokolle.
- Speichert Name und ID des erstellten Protokolls.
- Behandelt mögliche Fehler und speichert entsprechende Statusmeldungen.

#### 5. **DataLogClose**

- Schließt ein offenes Datenprotokoll.
- Aktualisiert den Status des Log-Eintrags nach dem Schließen.

#### 6. **DataLogOpen**

- Öffnet ein bestehendes Datenprotokoll für Schreiboperationen.
- Behandelt Fälle, in denen das Protokoll bereits geöffnet ist.

#### 7. **DataLogWrite**

- Schreibt Daten aus dem Puffer in das Datenprotokoll.
- Überwacht den Schreibstatus und behandelt Fehler.

#### 8. **DataLogNewFile**

- Erstellt ein neues Datenprotokoll, wenn das aktuelle voll ist oder ein neuer Abschnitt benötigt wird.
- Aktualisiert den Log-Eintrags-Index entsprechend.

#### 9. **DataLogDelete**

- Löscht bestehende Datenprotokolle.
- Unterstützt das Löschen aller Protokolle über einen definierten Ablauf.

#### 10. **Reset**

- Setzt Fehlerzustände und Statistikvariablen zurück.

#### 11. **All ON/OFF**

- Ermöglicht das Aktivieren oder Deaktivieren aller Logging-Tags über die Variablen `TurnAllOn` und `TurnAlloff`.

### Funktionsweise

Der Funktionsbaustein arbeitet zyklisch und führt abhängig von den gesetzten Flags und Befehlen die entsprechenden Operationen aus. Er überwacht kontinuierlich den Status der Logging-Operationen und reagiert auf Ereignisse wie Pufferüberlauf, Fehlerzustände oder Benutzerbefehle.

### Statusmeldungen

- Jeder Data Logging-Operation werden Statuscodes zugeordnet.
- Die Statuscodes werden in lesbare Textmeldungen umgewandelt und in `memStatusMsg` gespeichert.
- Dies erleichtert die Diagnose von Problemen und die Überwachung des Systems.

### Beispiel für Statusbehandlung in `DataLogCreate`

```scl
CASE "DB_LogMsg".DLcreate.memStatus OF
    0:
        "DB_LogMsg".DLcreate.memStatusMsg := 'Keine Fehler.';
    7000:
        "DB_LogMsg".DLcreate.memStatusMsg := 'Keine Auftragsbearbeitung aktiv.';
    32915:
        "DB_LogMsg".DLcreate.memStatusMsg := 'Data Log existiert bereits.';
    ELSE
        "DB_LogMsg".DLcreate.memStatusMsg := 'Unbekannter Fehlercode';
END_CASE;
```

## Hinweise zur Verwendung

- **CPU-Konfiguration**:
  - Aktivieren Sie den Webserver der CPU unter PROFINET-Schnittstelle > Zugriff auf den Webserver.
  - Aktivieren Sie den Webserver in den CPU-Eigenschaften.
  - Erstellen Sie einen Benutzer mit Admin-Rechten (alle Lese-, Schreib- und Löschrechte).
- **Wichtige Hinweise**:
  - Wählen Sie immer zuerst den aufzurufenden Datenprotokolleintrag aus, bevor Sie eine Data Logging-Anweisung ausführen.
  - Warten Sie stets, bis eine Data Logging-Operation abgeschlossen ist (`DONE` ist `TRUE`), bevor Sie die nächste starten.
- **Fehlerbehandlung**:
  - Überwachen Sie die `memStatus` und `memStatusMsg`, um den Status der Operationen zu überprüfen.
  - Nutzen Sie die `reset`-Variable, um Fehlerzustände zurückzusetzen.
- **Automatische Funktionen**:
  - `Auto_F_Stopp`: Stoppt das Logging automatisch, wenn die maximale Anzahl von Fehlern erreicht wurde.
  - `Auto_open`: Öffnet automatisch die Datei beim Schreiben.
  - `Auto_NewFile`: Erstellt automatisch eine neue Log-Datei, wenn die aktuelle voll ist.

## Schlussfolgerung

Dieser SCL-Code bietet eine umfassende Lösung für das Data Logging in SPS-Anwendungen mit dem Siemens TIA Portal. Durch die modularen Strukturen und Funktionen können Sie das Logging flexibel an Ihre Anforderungen anpassen und effektiv verwalten. Die Implementierung berücksichtigt Fehlerbehandlung, automatische Steuerung und bietet umfangreiche Diagnosemöglichkeiten über Statusmeldungen.

**Hinweis**: Passen Sie die Parameter und Einstellungen entsprechend Ihren spezifischen Anforderungen und der Konfiguration Ihrer SPS an.