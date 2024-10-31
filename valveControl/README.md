# ValveControl_2_way

**Version**: 1.0  
**Autor**: Daniel Caceres Lopez  
**Titel**: Ventilsteuerung 2 Wege  
**Zweck**: Steuerung eines 2-Wege-Ventils mit Statusüberwachung und Fehlerausgabe

### Status : 

---

## Inhaltsverzeichnis
1. [Beschreibung](#beschreibung)
2. [Schnittstellen](#schnittstellen)
   - [Eingangsvariablen](#eingangsvariablen)
   - [Ausgangsvariablen](#ausgangsvariablen)
3. [Temporäre Variablen](#temporäre-variablen)
4. [Interne Variablen](#interne-variablen)
5. [Funktionsbeschreibung](#funktionsbeschreibung)
6. [Fehlerbehandlung](#fehlerbehandlung)
7. [Versionshinweise](#versionshinweise)

---

## Beschreibung

Der Funktionsbaustein `ValveControl_2_way` steuert ein 2-Wege-Ventil, welches geöffnet und geschlossen werden kann. Der Baustein überwacht den aktuellen Status des Ventils und generiert Fehlermeldungen, falls unerwartete Zustände auftreten. Das Ziel ist eine sichere und zuverlässige Steuerung, die auf den Zustand des Ventils und den Not-Aus-Status reagiert und klare Fehlerbeschreibungen liefert.

## Schnittstellen

### Eingangsvariablen

| Name         | Typ  | Beschreibung                                                  |
|--------------|------|---------------------------------------------------------------|
| `open`       | BOOL | Steuerungssignal zum Öffnen des Ventils                       |
| `close`      | BOOL | Steuerungssignal zum Schließen des Ventils                    |
| `statusOpen` | BOOL | Rückmeldung, dass das Ventil geöffnet ist                     |
| `statusClose`| BOOL | Rückmeldung, dass das Ventil geschlossen ist                  |
| `estopOk`    | BOOL | Not-Aus-Status (TRUE: nicht ausgelöst, FALSE: ausgelöst)      |

### Ausgangsvariablen

| Name        | Typ   | Beschreibung                                         |
|-------------|-------|------------------------------------------------------|
| `y_open`    | BOOL  | Ausgangssignal zum Öffnen des Ventils                |
| `y_close`   | BOOL  | Ausgangssignal zum Schließen des Ventils             |
| `error`     | INT   | Fehlercode zur Fehlererkennung                       |
| `errorText` | STRING| Fehlerbeschreibung als Textausgabe                   |

## Temporäre Variablen

| Name       | Typ  | Beschreibung                                         |
|------------|------|------------------------------------------------------|
| `t_open`   | BOOL | Temporäres Signal für das Öffnen des Ventils         |
| `t_close`  | BOOL | Temporäres Signal für das Schließen des Ventils      |

## Interne Variablen

| Name        | Typ | Beschreibung                                                           |
|-------------|-----|------------------------------------------------------------------------|
| `tonOpen`   | TON | Zeitbaustein zur Überwachung des Öffnungsstatus des Ventils           |
| `tonClose`  | TON | Zeitbaustein zur Überwachung des Schließstatus des Ventils            |

## Funktionsbeschreibung

1. **Initialisierung**: Alle Ausgänge werden initial auf `FALSE` gesetzt, und der Fehlercode (`error`) wird auf `0` gesetzt, was "Kein Fehler" bedeutet.

2. **Steuerung des Ventils**:  
   - Wenn der Not-Aus (`estopOk`) aktiv ist (`TRUE`), wird das Ventil über die Eingänge `open` und `close` gesteuert:
     - **Öffnen**: Wenn `open = TRUE` und `close = FALSE`, wird das temporäre Signal `t_open` gesetzt, während `t_close` auf `FALSE` bleibt.
     - **Schließen**: Wenn `close = TRUE` und `open = FALSE`, wird `t_close` gesetzt und `t_open` deaktiviert.
     - Wenn beide oder keiner der Eingänge (`open` und `close`) aktiv sind, bleiben beide temporären Signale (`t_open`, `t_close`) auf `FALSE`.
   - Wenn der Not-Aus nicht aktiv ist (`estopOk = FALSE`), werden `t_open` und `t_close` auf `FALSE` gesetzt, und es wird der Fehlercode `1` für "Not-Aus aktiv" ausgegeben.

3. **Setzen der Ausgangssignale**: Die Ausgangssignale `y_open` und `y_close` werden entsprechend der temporären Signale `t_open` und `t_close` gesetzt.

4. **Statusüberwachung**:
   - Der `tonOpen`-Zeitbaustein überwacht, ob das Ventil in der angegebenen Zeitspanne öffnet. Ist dies nicht der Fall, wird Fehlercode `2` ("Ventil öffnet nicht") ausgegeben.
   - Der `tonClose`-Zeitbaustein überwacht, ob das Ventil in der angegebenen Zeitspanne schließt. Wenn dies fehlschlägt, wird Fehlercode `3` ("Ventil schließt nicht") gesetzt.
   - Wird kein Status (`statusClose` oder `statusOpen`) zurückgemeldet, wird der Fehlercode `4` ("Ventilstatus unbekannt") ausgegeben.
   - Sind `open` und `close` gleichzeitig aktiv, wird der Fehlercode `5` ("Öffnen und Schließen gleichzeitig") gesetzt.

5. **Fehlerausgabe in Textform**: Der Fehlercode wird über den Ausgang `errorText` als lesbare Textmeldung ausgegeben, abhängig vom aktuellen Fehlercode.

## Fehlerbehandlung

Die folgende Tabelle beschreibt die möglichen Fehlerzustände und ihre Ursachen:

| Fehlercode | Beschreibung                          | Ursache                                         |
|------------|--------------------------------------|-------------------------------------------------|
| 1          | `Not-Aus aktiv`                      | Not-Aus ist ausgelöst                           |
| 2          | `Ventil öffnet nicht`                | Ventil öffnet nicht innerhalb der Zeitgrenze    |
| 3          | `Ventil schließt nicht`              | Ventil schließt nicht innerhalb der Zeitgrenze  |
| 4          | `Ventilstatus unbekannt`             | Status unbekannt - weder `statusOpen` noch `statusClose` aktiv |
| 5          | `Öffnen und Schließen gleichzeitig`  | Beide Befehle (`open` und `close`) sind aktiv   |
| 0          | `Kein Fehler`                        | Normalzustand                                   |

## Versionshinweise

- **Version 1.0**: Initiale Implementierung des 2-Wege-Ventilsteuerungsbausteins mit Fehlerbehandlung und Statusüberwachung.

