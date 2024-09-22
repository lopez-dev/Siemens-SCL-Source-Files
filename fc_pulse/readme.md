# Dokumentation zur Funktion `fc_pulse`

## Übersicht

Die Funktion `fc_pulse` wird verwendet, um sowohl positive als auch negative Flanken von einem Bool-Eingangssignal zu erkennen. Diese Funktion nutzt einen benutzerdefinierten Datentyp `type_fc_pulse`, um die Flankeninformationen zu speichern und zu verarbeiten.

## Typdefinition: `type_fc_pulse`

### Beschreibung

Der Datentyp `type_fc_pulse` ist eine Struktur, die Variablen für die Erkennung und Speicherung von Flankeninformationen enthält. Dieser Datentyp ermöglicht es, sowohl positive als auch negative Flanken eines Signals zu erkennen und zu speichern.

### Struktur

- `Pulse_pos_Var` (Bool): Ausgangsvariable der positiven Flanke. Diese Variable wird gesetzt, wenn eine positive Flanke erkannt wird.
- `Pulse_neg_Var` (Bool): Ausgangsvariable der negativen Flanke. Diese Variable wird gesetzt, wenn eine negative Flanke erkannt wird.
- `Edge_flag_pos` (Bool): Flankenmerker für die positive Flanke. Diese Variable dient als Speicher, um den Zustand des Eingangssignals zu speichern und wird bei jedem Funktionsaufruf aktualisiert.
- `Edge_flag_neg` (Bool): Flankenmerker für die negative Flanke. Ähnlich wie `Edge_flag_pos`, dient diese Variable als Speicher für die Erkennung der negativen Flanke.

### Code

```scl
TYPE "type_fc_pulse"
TITLE = type_fc_pulse
VERSION : 0.1
//Variablen für die Funktion fc_pulse
   STRUCT
      Pulse_pos_Var : Bool;   // Ausgangsvariable der positiven Flanke                 //Output variable of the positive pulse
      Pulse_neg_Var : Bool;   // Ausgangsvariable der negativen Flanke                 //Output variable of the negative pulse
      Edge_flag_pos : Bool;   // Flankenmerker der pos. Flanke. Übergabe im Aufruf.    //Memory flag of the positive pulse. Assignment by call of the FC_Pulse
      Edge_flag_neg : Bool;   // Flankenmerker der neg. Flanke. Übergabe im Aufruf.    //Memory flag of the negative pulse. Assignment by call of the FC_Pulse
   END_STRUCT;

END_TYPE

FUNCTION "fc_pulse" : Void
{ S7_Optimized_Access := 'TRUE' }
VERSION : 0.1
   VAR_INPUT 
      In_Var : Bool;   // Variable von der die Flanke gebildet wird 
   END_VAR

   VAR_IN_OUT 
      type_fc_pulse_Var : "type_fc_pulse";
   END_VAR


BEGIN
	//Bildung der positiven Flanke  //get the positive pulse
	#type_fc_pulse_Var.Pulse_pos_Var := #In_Var AND NOT #type_fc_pulse_Var.Edge_flag_pos;
	#type_fc_pulse_Var.Edge_flag_pos := #In_Var;
	
	//Bildung der negativen Flanke  //get the negative pulse
	#type_fc_pulse_Var.Pulse_neg_Var := NOT #In_Var AND #type_fc_pulse_Var.Edge_flag_neg;
	#type_fc_pulse_Var.Edge_flag_neg := #In_Var;
END_FUNCTION

```
