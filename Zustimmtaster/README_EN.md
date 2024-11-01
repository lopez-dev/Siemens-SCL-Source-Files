# Zustimmtaster_SSP Function Block Documentation

## General Information
- **Title**: Zustimmtaster_SSP
- **Family**: Approval
- **Author**: Lopez

## Overview
The `Zustimmtaster_SSP` function block controls a safety approval switch and its associated output logic, which is activated depending on various input conditions and the current state of the switch. It manages the control of LED indicators, outputs, and a BCD-based number calculation.

---

## Interfaces

### Input Variables (`var_input`)
| Variable         | Type    | Description                                              |
|------------------|---------|----------------------------------------------------------|
| `not_halt_OK`    | `bool`  | Emergency stop OK signal (active if emergency stop is OK). |
| `drei_pos_sw`    | `bool`  | Three-position switch (approval switch).                 |
| `activity_sensor`| `bool`  | Activity sensor on wall mount.                           |
| `Bit_0`          | `bool`  | Bit 0 for BCD counting.                                  |
| `Bit_1`          | `bool`  | Bit 1 for BCD counting.                                  |
| `Bit_2`          | `bool`  | Bit 2 for BCD counting.                                  |
| `Bit_3`          | `bool`  | Bit 3 for BCD counting.                                  |
| `SW_Right`       | `bool`  | Switch for right movement.                               |
| `SW_Left`        | `bool`  | Switch for left movement.                                |
| `LED_Takt`       | `bool`  | Clock for LED blinking.                                  |
| `BCD_offset`     | `int`   | Offset value for BCD calculation.                        |

### Output Variables (`var_output`)
| Variable         | Type                     | Description                                         |
|------------------|--------------------------|-----------------------------------------------------|
| `LED_Red`        | `bool`                   | Red LED indicator (status signal).                  |
| `LED_Green`      | `bool`                   | Green LED indicator (status signal).                |
| `Out_Right`      | `bool`                   | Output signal for right movement.                   |
| `Out_Left`       | `bool`                   | Output signal for left movement.                    |
| `BCD_Nr`         | `int`                    | Current BCD number (counter).                       |
| `OutputLeft`     | `ARRAY[1..16] OF BOOL`   | Individual outputs for left movement.               |
| `OutputRight`    | `ARRAY[1..16] OF BOOL`   | Individual outputs for right movement.              |

### Internal Variables (`var`)
| Variable             | Type    | Description                                         |
|----------------------|---------|-----------------------------------------------------|
| `Freigabe`           | `bool`  | Control release signal to activate outputs.         |
| `Edge_flag_pos_3SW`  | `bool`  | Edge flag for the three-position switch.            |
| `Edge_flag_pos_LeftSW` | `bool` | Edge flag for the left switch.                    |
| `Edge_flag_pos_RightSW` | `bool` | Edge flag for the right switch.                  |
| `BCD_is_diff`        | `bool`  | Marker for BCD number change.                       |
| `BCD_Nr_cache`       | `int`   | Cache for the previous BCD number.                  |
| `Bit_Wert_0`         | `int`   | Value of BCD bit 0.                                 |
| `Bit_Wert_1`         | `int`   | Value of BCD bit 1.                                 |
| `Bit_Wert_2`         | `int`   | Value of BCD bit 2.                                 |
| `Bit_Wert_3`         | `int`   | Value of BCD bit 3.                                 |
| `Constant_bit0`      | `int`   | Constant for BCD value of bit 0.                    |
| `Constant_bit1`      | `int`   | Constant for BCD value of bit 1.                    |
| `Constant_bit2`      | `int`   | Constant for BCD value of bit 2.                    |
| `Constant_bit3`      | `int`   | Constant for BCD value of bit 3.                    |
| `i`                  | `int`   | Loop index for individual outputs.                  |

### Temporary Variables (`var_temp`)
| Variable               | Type    | Description                                           |
|------------------------|---------|-------------------------------------------------------|
| `BCD_Result`           | `int`   | Calculated BCD value from the bit inputs and offset.  |
| `Pulse_GrundStellung`  | `bool`  | Pulse for the home position of the approval switch.   |
| `Pulse_Left`           | `bool`  | Pulse for the left switch.                            |
| `Pulse_Right`          | `bool`  | Pulse for the right switch.                           |

---

## Function Description

### LED Indicators
The LEDs are controlled based on the input signals:
- **LED_Red** is active when `not_halt_OK` is inactive.
- **LED_Green** blinks or remains on depending on `activity_sensor`, `drei_pos_sw`, `LED_Takt`, and `not_halt_OK`.

### BCD Calculation
The BCD calculation is performed by summing up bits `Bit_0` to `Bit_3` with the `BCD_offset`:
1. **Bit Values**: Each bit (`Bit_0` to `Bit_3`) is converted to a value based on its constant.
2. **Summing**: The sum of the bit values with `BCD_offset` produces the current BCD value (`BCD_Result`).
3. **Change Detection**: A change (`BCD_is_diff`) is detected if `BCD_Result` differs from `BCD_Nr_cache`.

### Release Control
The release signal (`Freigabe`) is reset when the BCD value changes or the approval switch (`drei_pos_sw`) is released. It reactivates on a positive edge of the approval switch (`Pulse_GrundStellung`).

### Output Control
- **Left**: The left output (`Out_Left`) is activated on a positive pulse from `SW_Left`, with the release signal and `not_halt_OK` active, provided `SW_Right` is not active.
- **Right**: The right output (`Out_Right`) is activated on a positive pulse from `SW_Right`, with the release signal and `not_halt_OK` active, provided `SW_Left` is not active.

### Individual Outputs
Each individual output in `OutputLeft` and `OutputRight` is activated if the current BCD number (`BCD_Result`) matches the output's index and the respective main output (`Out_Left` or `Out_Right`) is active.

---

## Workflow

1. **Initialization**: Variables and bit values are calculated based on the inputs.
2. **LED Control**: Indicators are set based on `not_halt_OK` and other parameters.
3. **Release Check**: The release signal is set or reset based on the state of the approval switch and the other switches.
4. **Output Control**: The `Out_Left` and `Out_Right` outputs, as well as the individual outputs, are activated or deactivated based on `BCD_Result` and the main outputs.

---

## Notes
- The release signal is only set when safety and condition signals are active and on a positive edge of the approval switch.
- LED and output control are based on synchronized edge detection and state checks to prevent incorrect activations.

--- 

## Changelog
- **Version 1.0** - Initial release by author Lopez
