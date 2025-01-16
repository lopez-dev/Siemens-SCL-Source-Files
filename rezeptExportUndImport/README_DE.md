# Dokumentation für den Funktionsbaustein "rezept_ex-import"

## Überblick
Der Funktionsbaustein `rezept_ex-import` dient zur Verwaltung von Import- und Exportprozessen für Rezeptdaten. Er ermöglicht die Steuerung und Statusüberwachung dieser Prozesse, einschließlich Fehlerbehandlung und Alarmverwaltung. Es kommen optimierte Datenstrukturen für Import und Export (`importDataUDT`, `exportDataUDT`) sowie externe Funktionsinstanzen für den Rezeptimport und -export zum Einsatz.

---

## Datentypdefinitionen

### `importDataUDT`
Datenstruktur zur Steuerung und Überwachung des Importprozesses.

| Feld             | Typ   | Beschreibung                               |
|-------------------|-------|-------------------------------------------|
| `req`            | Bool  | Anforderung zur Ausführung des Imports.   |
| `done`           | Bool  | Import erfolgreich abgeschlossen.         |
| `busy`           | Bool  | Import wird gerade ausgeführt.            |
| `error`          | Bool  | Fehler während des Imports.               |
| `status`         | Word  | Statuscode des Importprozesses.           |
| `lastErrorStatus`| Word  | Letzter Fehlerstatus des Imports.          |

### `exportDataUDT`
Datenstruktur zur Steuerung und Überwachung des Exportprozesses.

| Feld             | Typ   | Beschreibung                               |
|-------------------|-------|-------------------------------------------|
| `req`            | Bool  | Anforderung zur Ausführung des Exports.   |
| `done`           | Bool  | Export erfolgreich abgeschlossen.         |
| `busy`           | Bool  | Export wird gerade ausgeführt.            |
| `error`          | Bool  | Fehler während des Exports.               |
| `status`         | Word  | Statuscode des Exportprozesses.           |
| `lastErrorStatus`| Word  | Letzter Fehlerstatus des Exports.          |

---

## Funktionsbaustein `rezept_ex-import`

### Eigenschaften

- **Optimierter Zugriff**: Der Baustein ist auf optimierten Zugriff (`S7_Optimized_Access := 'TRUE'`) ausgelegt.
- **Instanzen von Import- und Export-Funktionen**: Integriert spezialisierte Funktionen zur Abwicklung der Import- und Exportprozesse.
- **Fehler- und Statusverarbeitung**: Unterstützt Fehlerhandling und Alarmverarbeitung über dedizierte Instanzen (`Program_Alarm`).

### Schnittstellen

#### Eingangs-/Ausgangsvariablen (`VAR_IN_OUT`)

| Name            | Typ            | Beschreibung                                      |
|------------------|----------------|--------------------------------------------------|
| `exportDataUDT` | `exportDataUDT`| Datenstruktur für Exportsteuerung und -status.  |
| `importDataUDT` | `importDataUDT`| Datenstruktur für Importsteuerung und -status.  |
| `DB`            | Variant        | Rezept-Datenbaustein.                            |

#### Interne Variablen (`VAR`)

| Name                  | Typ            | Beschreibung                                     |
|-----------------------|----------------|-------------------------------------------------|
| `RecipeExport_Instance` | `RecipeExport` | Instanz für den Rezept-Export.                 |
| `RecipeImport_Instance` | `RecipeImport` | Instanz für den Rezept-Import.                 |
| `Export_Data`         | `Program_Alarm` | Alarmverarbeitung für den Export.              |
| `Import_Data`         | `Program_Alarm` | Alarmverarbeitung für den Import.              |

#### Temporäre Variablen (`VAR_TEMP`)

| Name             | Typ  | Beschreibung                                    |
|-------------------|------|------------------------------------------------|
| `importTextByte` | Byte | Byte-Darstellung des Importstatus.             |
| `exportTextByte` | Byte | Byte-Darstellung des Exportstatus.             |

---

### Logik des Funktionsbausteins

#### Exportprozess (`REGION Export`)

1. **Startbedingungen**:
   - Der Export wird gestartet, wenn `req` aktiv ist und der Prozess nicht `busy` ist.
2. **Instanzaufruf**:
   - Die Instanz `RecipeExport_Instance` führt den Export aus, wobei die Steuer- und Statusvariablen übergeben werden.
3. **Statusaktualisierung**:
   - Nach der Instanziierung wird `req` auf `FALSE` gesetzt.
   - Bei einem Fehler oder Abschluss wird der letzte Fehlerstatus in `lastErrorStatus` gespeichert.
4. **Statusverarbeitung**:
   - Der Statuscode wird von `Word` in `Byte` umgewandelt.
   - Der Alarmstatus wird über die Instanz `Export_Data` verarbeitet.

#### Importprozess (`REGION Import`)

1. **Startbedingungen**:
   - Der Import wird gestartet, wenn `req` aktiv ist und der Prozess nicht `busy` ist.
2. **Instanzaufruf**:
   - Die Instanz `RecipeImport_Instance` führt den Import aus, wobei die Steuer- und Statusvariablen übergeben werden.
3. **Statusaktualisierung**:
   - Nach der Instanziierung wird `req` auf `FALSE` gesetzt.
   - Bei einem Fehler oder Abschluss wird der letzte Fehlerstatus in `lastErrorStatus` gespeichert.
4. **Statusverarbeitung**:
   - Der Statuscode wird von `Word` in `Byte` umgewandelt.
   - Der Alarmstatus wird über die Instanz `Import_Data` verarbeitet.

---

## Nutzungshinweise

- **Rezept-Datenbaustein**: Der Datenbaustein muss kompatibel mit den erwarteten Rezeptformaten sein.
- **Fehlerbehandlung**: Die Fehlercodes und Statuswerte sollten in der Dokumentation der Instanzmodule (`RecipeExport`, `RecipeImport`, `Program_Alarm`) nachgeschlagen werden.
- **Optimierung**: Stellen Sie sicher, dass der optimierte Zugriff in der Ziel-SPS unterstützt wird.

---

## Änderungsprotokoll

| Version | Änderungen                | Datum        |
|---------|---------------------------|--------------|
| 0.1     | Initiale Implementierung  | 16.01.2025   |
