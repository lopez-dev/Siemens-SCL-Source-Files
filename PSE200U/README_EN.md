# Function Block `fb_PSE200U`

## Overview

The `fb_PSE200U` function block is responsible for managing and monitoring the status of a PSE200U device. It handles the sequence of operations, checks the statuses of channels, monitors the progression of steps, and identifies errors in real-time.

### Key Features:
- Monitors up to 4 channels for correct functionality.
- Implements a sequential step control mechanism.
- Detects and handles errors during execution.
- Provides detailed status and error outputs.

---

## Interface

### Input Parameters

| Name | Type | Description |
|------|------|-------------|
| `S`  | `Bool` | The status signal from the PSE200U device. This input triggers the start of the sequence when TRUE. |

### Output Parameters

| Name | Type | Description |
|------|------|-------------|
| `CH1` | `Bool` | Output indicating the status of Channel 1 (TRUE if functioning correctly). |
| `CH2` | `Bool` | Output indicating the status of Channel 2 (TRUE if functioning correctly). |
| `CH3` | `Bool` | Output indicating the status of Channel 3 (TRUE if functioning correctly). |
| `CH4` | `Bool` | Output indicating the status of Channel 4 (TRUE if functioning correctly). |
| `error` | `Bool` | Error flag, set to TRUE if any error is detected in the sequence. |
| `status` | `Int` | Status output providing a detailed error or operation state code. |

---

## Internal Variables

### Timers

| Name        | Type      | Description |
|-------------|-----------|-------------|
| `TIME`      | `TON_TIME` | The main timer controlling the sequence timing. |
| `CHECK`     | `TON_TIME` | Timer used to validate each step in the sequence. |
| `ERROR_TON` | `TON_TIME` | Timer that tracks inactivity to detect errors. |

### State Variables

| Name           | Type         | Description |
|----------------|--------------|-------------|
| `state`        | `Int`        | Current state of the operational sequence. |
| `StepStatus`   | `Array[0..9] of Bool` | Boolean array storing the status of each step in the sequence. |
| `ChannelStatus` | `Array[1..4] of Bool` | Boolean array storing the status of each channel (TRUE if functioning correctly). |
| `saved_state`  | `Int`        | Stores the last known state to detect changes. |

### Temporary Variables

| Name          | Type | Description |
|---------------|------|-------------|
| `i`           | `Int` | Index variable for loop operations. |
| `status_tmp`  | `Int` | Temporary variable to store status before assignment to the output. |

### Constants

| Name          | Type  | Default Value | Description |
|---------------|-------|---------------|-------------|
| `StartTimeTrig` | `Time` | `T#500MS` | The delay time before the sequence starts. |
| `deviation`     | `Time` | `T#10MS`  | The allowable time deviation for timing checks. |
| `MaxTime`       | `Time` | `T#2S_750MS` | Maximum time allowed for the entire sequence to complete. |
| `T_Interval`    | `Time` | `T#250MS` | The interval time between steps and channel checks. |

---

## Logic Structure

### Main Timer (`Start main Timer` Region)
This section initializes the main timer when the sequence starts (`state > 0`). The timer counts the total duration allowed for the sequence, ensuring it doesn't exceed the predefined maximum time (`MaxTime`).

### Sequence Control (`Check Sequence` Region)
This region controls the operational steps. The sequence starts by initializing the steps (state 0), then alternates between pauses and channel checks (states 1 to 9). Each step's status is recorded, and the appropriate channels are updated.

- **Initialization (State 0):** All steps are reset and the sequence begins.
- **States 1 to 9:** Alternating between pauses and checking the status of channels, with the appropriate timers applied for each step.
- **Completion (State 10):** Marks the sequence as complete and resets the state.

### Error Detection (`Error Handling` Region)
This region detects errors based on the inactivity or abnormal execution of the sequence. Errors are classified with status codes:

- `0`: No error.
- `1`: Channel error.
- `2`: Pause error.
- `3`: Sequence not started.
- `4`: Sequence stopped.

Timers are used to monitor the sequence progress. If the state does not change within the allowed time, an error is triggered. 

### Output Assignment (`Assign Outputs` Region)
The final region maps the status of the channels to the corresponding outputs (`CH1`, `CH2`, `CH3`, `CH4`) and sets the `error` flag if any issue is detected. The `status` output reflects the specific error or operation state code.

---

## Error Status Codes

The following error status codes are used to diagnose issues in the sequence:

| Status Code | Description |
|-------------|-------------|
| `0`         | No error. |
| `1`         | Channel error (a channel did not complete successfully). |
| `2`         | Pause error (a pause did not complete as expected). |
| `3`         | Sequence did not start. |
| `4`         | Sequence was stopped unexpectedly. |

---

## Sequence Flow

1. **Initialization (State 0):** The sequence begins by resetting all steps.
2. **Intermediate States (1-9):** Alternating between checking pauses and channels.
   - Odd states represent pauses.
   - Even states check the corresponding channel status.
3. **Completion (State 10):** The sequence finishes, error detection occurs, and the state is reset.
4. **Error Handling:** Monitors for state changes and triggers error codes if abnormalities occur.

