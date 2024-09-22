### Dokumentation: Data Logging in einer SPS

#### Einleitung
Die Funktionalität, die in diesem Code implementiert ist, bezieht sich auf ein Datenprotokollierungssystem (Data Logging) für eine speicherprogrammierbare Steuerung (SPS). Das System ermöglicht das Erstellen, Schreiben, Schließen, Öffnen und Löschen von Protokolldateien. Ziel ist es, Daten in der SPS zu sammeln, zu speichern und zu verarbeiten. Hier werden die Hauptkomponenten und Funktionen beschrieben.

---

### 1. **Struktur: `typ_DataLogging_DB`**

Diese Struktur ist das Herzstück des Systems. Sie definiert alle Parameter und Steuerungsvariablen, die zur Verwaltung eines Data Logs erforderlich sind.

**Felder:**
- **LogEntryIndex**: Der Index des aktuellen Log-Eintrags.
- **nextPosInStack**: Die nächste Position im Stack für die Speicherung neuer Log-Daten.
- **logID**: Die eindeutige ID für ein Log.
- **name**: Der Name des Logs, das dynamisch während der Protokollerstellung generiert wird.
- **newName**: Standardname für neue Log-Dateien.

**dataLogEntries**: Ein Array, das bis zu 8 Log-Einträge speichert. Jeder Eintrag enthält:
  - **name**: Der Name des Logs.
  - **ID**: Eine eindeutige ID für jeden Log-Eintrag.
  - **DLclosed**: Ein Statusbit, das anzeigt, ob das Log geschlossen ist.

#### Datenprotokoll-Operationen:

Die Struktur enthält spezifische Unterstrukturen, die verschiedene Aktionen wie Erstellen, Schreiben, Schließen und Öffnen von Logs steuern.

1. **DLcreate**: Steuert die Erstellung eines neuen Logs.
2. **DLdelete**: Ermöglicht das Löschen eines vorhandenen Logs.
3. **DLwrite**: Verarbeitet das Schreiben neuer Daten in ein Log.
4. **DLnewfile**: Erstellt eine neue Log-Datei basierend auf den Daten eines vorhandenen Logs.
5. **DLclose**: Schließt ein Log nach Abschluss der Schreibvorgänge.
6. **DLopen**: Öffnet ein vorhandenes Log zur Bearbeitung.

#### Parameter:

Die Parameter in der Struktur definieren die Rahmenbedingungen für die Protokollierung, darunter:
- **Logging**: Ein Array von Booleans, das angibt, ob das Logging aktiv ist.
- **BufferSize**: Die Größe des Puffers für die Datenspeicherung.
- **newRECORDS**: Die Anzahl der neuen Protokolleinträge.
- **logHeader**: Der Header des Logs.
- **maxPosEntry**: Die maximale Anzahl von Einträgen.
- **Buffer**: Ein Array zur Zwischenspeicherung von Daten, bevor sie ins Log geschrieben werden.

---

### 2. **Funktionen**

#### 2.1 **`fc_LogMsg_InputBuffer`**
Diese Funktion verwaltet die Eingabe neuer Meldungen in den Protokollpuffer. Sie verschiebt die ältesten Einträge im Puffer, um Platz für neue Meldungen zu schaffen. Sobald eine neue Meldung eingegeben wird, wird sie an die erste Stelle im Puffer geschrieben. Die Funktion unterstützt auch das Melden von Pufferüberläufen.

#### 2.2 **`fc_LogMsg_NextEntry`**
Diese Funktion erhöht den Index des aktuellen Log-Eintrags, sodass das nächste verfügbare Log bearbeitet werden kann. Wenn der Index die maximale Anzahl von Einträgen überschreitet, wird er auf null zurückgesetzt.

#### 2.3 **`fc_LogMsg_CallEntry`**
Diese Funktion ruft den Namen und die ID des aktuellen Log-Eintrags ab, basierend auf dem aktuellen Index. Dadurch können Daten gezielt aus einem bestimmten Log-Eintrag extrahiert werden.

---

### 3. **Funktionsbaustein: `FB_LogMsg`**

Dieser Funktionsbaustein koordiniert alle Log-Aktionen und -Operationen. Die internen Instanzen wie **DataLogCreate**, **DataLogWrite**, **DataLogOpen**, **DataLogClose** usw. steuern die Hauptaufgaben.

#### Regions (Teile des Funktionsbausteins):
1. **Name Generator**: Generiert dynamisch neue Log-Namen basierend auf einem vordefinierten Präfix und der aktuellen Position im Stack.
2. **Buffer Manager**: Verarbeitet die Verwaltung der Daten im Puffer und überprüft, ob eine Pufferüberfüllung vorliegt.
3. **DataLogCreate**: Verwaltet die Erstellung neuer Logs und behandelt eventuelle Fehler während des Erstellungsprozesses.
4. **DataLogWrite**: Behandelt das Schreiben von Daten in die erstellten Logs. Wenn ein Log voll ist, wird ein neues Log erstellt.
5. **DataLogClose**: Schließt das aktuelle Log und markiert es als abgeschlossen.
6. **DataLogOpen**: Öffnet ein bestehendes Log zur weiteren Bearbeitung.
7. **DataLogNewFile**: Erstellt eine neue Log-Datei aus einer vorhandenen Datei.
8. **DataLogDelete**: Löscht ein vorhandenes Log und stellt sicher, dass alle zugehörigen Daten und Statusinformationen entfernt werden.

---

### 4. **Verwendung und Empfehlungen**

#### Erstellen eines Logs:
Um ein neues Log zu erstellen, muss der Parameter `DLcreate.execute` auf `TRUE` gesetzt werden. Dadurch wird ein neues Log erstellt, und alle relevanten Parameter wie **name**, **ID** und **Header** werden festgelegt.

#### Schreiben in ein Log:
Wenn neue Daten in ein Log geschrieben werden sollen, wird der Parameter `DLwrite.execute` auf `TRUE` gesetzt. Die Funktion übernimmt die Daten aus dem Puffer und schreibt sie in das Protokoll.

#### Schließen eines Logs:
Nach Abschluss der Schreibvorgänge muss das Log geschlossen werden. Dies wird durch Setzen des Parameters `DLclose.execute` auf `TRUE` erreicht.

#### Löschen eines Logs:
Zum Löschen eines Logs wird `DLdelete.execute` auf `TRUE` gesetzt, und das entsprechende Log wird gelöscht.

---

### 5. **Fehlerbehandlung**

Die Struktur enthält verschiedene **Statusmeldungen** wie `memStatusMsg`, die verwendet werden können, um den Status von Aktionen zu überwachen und Fehler zu erkennen. Es ist wichtig, den Status nach jeder Aktion zu überprüfen, um sicherzustellen, dass keine Fehler aufgetreten sind. Typische Fehlercodes können Probleme wie unzureichenden Speicherplatz, Zugriffsprobleme oder unzulässige Dateinamen anzeigen.
