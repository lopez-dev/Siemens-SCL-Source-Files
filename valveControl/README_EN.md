# Valve Control Documentation

This document describes the structure and functionality of the valve control logic in SCL. The implementation includes the `udt_valveStruct` structure for defining valve data and the `fb_valveControl` function block for managing valve states. Additional functions, `fc_valveClose` and `fc_valveOpen`, provide basic mechanisms for initializing the valve's closing and opening states.

- ✅ **Completed**: This project is fully developed and tested.

## Table of Contents

1. [Structure `udt_valveStruct`](#udt_valvestruct)
2. [Function Block `fb_valveControl`](#fb_valvecontrol)
3. [Helper Functions](#helper-functions)
   - [Function `fc_valveClose`](#fc_valveclose)
   - [Function `fc_valveOpen`](#fc_valveopen)

---

### `udt_valveStruct`

The `udt_valveStruct` structure defines the properties and states of a valve.

#### Fields:

- **state** : `Int`  
  Current state of the valve.
  - `0`: closed
  - `1`: opening
  - `2`: open
  - `3`: closing

- **typeOfValve** : `Int`  
  Type of valve.
  - `0`: single-solenoid (default)
  - `1`: double-solenoid

- **open** : `Bool`  
  Input to open the valve (TRUE initiates opening).

- **close** : `Bool`  
  Input to close the valve (TRUE initiates closing).

- **fb_opened** : `Bool`  
  Feedback signal indicating the valve is fully open.

- **fb_closed** : `Bool`  
  Feedback signal indicating the valve is fully closed.

- **y_open** : `Bool`  
  Output to control the valve to open.

- **y_close** : `Bool`  
  Output to control the valve to close.

- **errorState** : `UInt`  
  Error state code:
  - `0`: No error
  - `1`: Valve not opening
  - `2`: Valve not closing
  - `3`: Open and close input activated simultaneously
  - `4`: Valve status unknown
  - `5`: Open and close feedback active simultaneously
  - `6`: Closed, but `fb_opened` active
  - `7`: Open, but `fb_closed` active

- **errorText** : `String[60]`  
  Text description of the current error state.

- **openTime** : `Time`  
  Timeout duration for opening the valve. `T#0ms` means off.

- **closeTime** : `Time`  
  Timeout duration for closing the valve. `T#0ms` means off.

- **unknowTime** : `Time`  
  Timeout duration for detecting an unknown valve state. `T#0ms` means off.

- **timerOpen** : `TON_TIME`  
  Timer to monitor the open valve operation.

- **timerClose** : `TON_TIME`  
  Timer to monitor the close valve operation.

- **timerUnknown** : `TON_TIME`  
  Timer to monitor the unknown valve state.

---

### `fb_valveControl`

The function block `fb_valveControl` controls the valve's behavior based on its inputs and state feedback.

#### Variables:

- **valve** : `udt_valveStruct`  
  The valve structure passed as an In-Out parameter.

#### State Constants:

- `STATE_CLOSED` : `Int := 0`  
  State constant for "closed".

- `STATE_OPENING` : `Int := 1`  
  State constant for "opening".

- `STATE_OPENED` : `Int := 2`  
  State constant for "fully open".

- `STATE_CLOSING` : `Int := 3`  
  State constant for "closing".

#### Regions:

- **States legend**  
  Describes the different states of the valve.

- **typeOfValve**  
  Logic for checking the valve type and applying specific behaviors:
  - Single-solenoid: Automatically closes if "open" is not active.
  - Double-solenoid: No specific logic implemented yet, placeholder for future expansion.

- **Valve Control Logic**  
  Controls the valve behavior based on its current state and input commands. Transitions between states are defined based on the valve's input and feedback signals.

- **Valve Outputs**  
  Controls the output signals `y_open` and `y_close` based on the valve's state.

- **Error handling**  
  Detects and assigns error states based on timers, input conflicts, and feedback signals.

- **Error text**  
  Assigns descriptive error messages based on the error state.

---

### Helper Functions

#### `fc_valveClose`

The `fc_valveClose` function checks if the valve is not already in the "closed" or "closing" state and, if necessary, sets the state to `STATE_CLOSING`.

```scl
FUNCTION "fc_valveClose" : Void
    { S7_Optimized_Access := 'TRUE' }
    VERSION : 0.1
VAR_IN_OUT
    valveData : "udt_valveStruct";
END_VAR

BEGIN
    IF #valveData.state <> 0 AND #valveData.state <> 3 THEN
        #valveData.state := 3;
    END_IF;
END_FUNCTION
```

#### `fc_valveOpen`

The `fc_valveOpen` function checks if the valve is not already in the "opening" or "open" state and, if necessary, sets the state to `STATE_OPENING`.

```scl
FUNCTION "fc_valveOpen" : Void
    { S7_Optimized_Access := 'TRUE' }
    VERSION : 0.1
VAR_IN_OUT
    valveData : "udt_valveStruct";
END_VAR

BEGIN
    IF #valveData.state <> 1 AND #valveData.state <> 2 THEN
        #valveData.state := 1;
    END_IF;
END_FUNCTION
```

---

### Change Log

- **Version 0.1**  
  - Initial implementation of the valve structure, control function block, and basic helper functions.

---