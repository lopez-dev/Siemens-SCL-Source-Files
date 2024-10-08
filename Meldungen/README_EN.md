# Function Block `fb_alarming`

## Overview

The `fb_alarming` function block is responsible for managing alarm conditions by monitoring a set of signals and detecting new errors. It uses pulse detection to identify changes and stores snapshots of alarm states for comparison in subsequent cycles.

---

## Interface

### Input Parameters

| Name   | Type | Description |
|--------|------|-------------|
| `reset` | `Bool` | Reset input to clear alarms and reset states. |

### Output Parameters

| Name      | Type  | Description |
|-----------|-------|-------------|
| `error`   | `Bool` | Indicates if an error is currently active. |
| `new_error` | `Bool` | Indicates if a new error has been detected since the last reset. |
| `state`   | `Byte` | Indicates the current state of the alarms. |

### In/Out Parameters

| Name       | Type           | Description |
|------------|----------------|-------------|
| `Meldungen` | `Array[0..7] of Word` | Array of alarm words being monitored for errors. |

### Internal Variables

| Name            | Type   | Description |
|-----------------|--------|-------------|
| `MeldeSnapshot` | `Array[0..7] of Word` | Stores the snapshot of the previous alarm state for comparison. |
| `reset_puls_pos` | `Bool` | Stores the positive pulse generated by the reset signal. |
| `st_error` | `Bool` | Internal flag for tracking active errors. |
| `st_new_error` | `Bool` | Internal flag for detecting new errors. |
| `R_TRIG_Instance` | `R_TRIG` | Trigger instance for detecting rising edges. |
| `R_TRIG_Instance` | `Array[0..15] of R_TRIG` | Array of rising edge triggers. |
| `F_TRIG_Instance` | `Array[0..15] of F_TRIG` | Array of falling edge triggers. |
| `dummy` | `Bool` | Temporary variable for unused outputs. |
| `i` | `Int` | Loop index for scanning through alarms. |

---

## Logic Structure

### Alarm Detection (`#error` Region)

This region checks each alarm word in the `Meldungen` array to detect if any alarm is active. If any bit in the words is set, the `st_error` flag is set to `TRUE`.

```scl
IF (#Meldungen[0] OR #Meldungen[1] OR ... OR #Meldungen[7]) <> 0 THEN
    #st_error := TRUE;
ELSE
    #st_error := FALSE;
END_IF;
```

### Reset Pulse Detection (`reset pulse` Region)

This region calls the `fc_pulse` function to generate a pulse from the reset signal. It detects a positive edge on the reset input to trigger further actions.

```scl
"fc_pulse"(In_Var := #reset,
           Pulse_pos_Var => #reset_puls_pos,
           Pulse_neg_Var => #dummy,
           Edge_flag_pos := #dummy,
           Edge_flag_neg := #dummy);
```

### Snapshot and Reset (`reset` Region)

If a reset pulse is detected (`reset_puls_pos`), the current state of the `Meldungen` array is saved into `MeldeSnapshot`, and the `st_new_error` flag is reset to `FALSE`.

```scl
IF #reset_puls_pos THEN
    #MeldeSnapshot := #Meldungen;
    #st_new_error := FALSE;
END_IF;
```

### New Error Detection Loop (`Schleife für erkennung von neue Fehler` Region)

This loop scans through each bit in the `Meldungen` array and compares it to the corresponding bit in `MeldeSnapshot` to detect if a new error has occurred. If a rising edge is detected (transition from 0 to 1), `st_new_error` is set to `TRUE`.

```scl
FOR #i := 0 TO 7 DO
    IF #Meldungen[#i].%X0 = 1 AND #MeldeSnapshot[#i].%X0 = 0 THEN
        #st_new_error := TRUE;
    END_IF;
    ...
END_FOR;
```

### State Handling (`state` Region)

The state of the alarms is determined based on the status of `st_error` and `st_new_error`:
- `state = 0`: No error
- `state = 1`: Error, but no new error
- `state = 2`: New error detected
- `state = 3`: Undefined state

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

### Outputs Assignment (`OUTPUTS` Region)

The `error` and `new_error` output variables are assigned the values of the internal flags `st_error` and `st_new_error`, respectively.

```scl
#error := #st_error;
#new_error := #st_new_error;
```

---

## Related Functions

### `fc_pulse`

This function generates positive and negative pulses from a boolean input. It is used in the `reset pulse` region to detect the rising and falling edges of the reset signal.

| Input | Type | Description |
|-------|------|-------------|
| `In_Var` | `Bool` | The input boolean variable used to generate the pulse. |

| Output | Type | Description |
|--------|------|-------------|
| `Pulse_pos_Var` | `Bool` | Output indicating the positive pulse (rising edge). |
| `Pulse_neg_Var` | `Bool` | Output indicating the negative pulse (falling edge). |

| In/Out | Type | Description |
|--------|------|-------------|
| `Edge_flag_pos` | `Bool` | Stores the state for detecting the positive edge. |
| `Edge_flag_neg` | `Bool` | Stores the state for detecting the negative edge. |

---

## Data Types

### `typ_Meld_HMI`

This user-defined data type is used for storing and handling alarm and warning messages for the HMI.

| Name | Type | Description |
|------|------|-------------|
| `ACK` | `Bool` | Acknowledgement signal for clearing alarms. |
| `Alarm` | `Bool` | Signal to display alarm messages. |
| `Warn` | `Bool` | Signal to display warning messages. |
| `Meld_Quitt` | `Array[0..19] of Word` | Array storing the state of alarms and their acknowledgements. |
| `Warnungen` | `Array[0..1] of Word` | Array storing the warning messages. |

---
