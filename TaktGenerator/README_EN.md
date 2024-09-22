# FB_Taktgenerator Documentation

## Overview

The **FB_Taktgenerator** is a function block that generates various frequency-based signals (Takt signals) and detects the rising edges (positive flanks) for each frequency. The frequencies range from 10Hz to 0.2Hz. It can be used in any part of the PLC program, and the outputs (Takt signals and Edge signals) can be assigned to user-defined UDTs in the respective DBs.

### Input and Output Description:

- **Takt_Signals (`udt_tg_takt`)**: This structure contains the output signals for different frequencies (Takt signals).
- **Edge_Signals (`udt_tg_takt`)**: This structure contains the rising edge detection signals (Edge signals).

### Internal Variables:

- **Internal_TaktSignals**: Stores the state (True/False) of the internal Takt signals.
- **Pulse_Trigger**: Stores the pulse triggers generated at the start of each Takt signal.
- **Edge_Flag**: Keeps track of the signal state for edge detection.
- **StartTime**: Stores the start time for each Takt signal to calculate the interval.
- **Takt_Intervals**: Stores the predefined intervals for each Takt signal.

### Constants:

- **MAX_TIME**: Maximum time value to handle time overflow.
- **const_100MS** - **const_5S**: Constants defining the time intervals for each Takt signal.

---

## Usage

To use the **FB_Taktgenerator**, follow these steps:

1. **Define UDTs**: Create an instance of the `udt_tg_takt` in your data block (DB) where you wish to store the output signals.
   
2. **Assign the UDTs to the Output Variables**: Assign the UDT instances to the `Takt_Signals` and `Edge_Signals` outputs of the **FB_Taktgenerator**.

3. **Call the FB**: The **FB_Taktgenerator** can be called from anywhere in the program, and it will generate Takt signals based on the system clock.

### Example

1. **Define a UDT Instance in a DB:**

```plc
DATA_BLOCK "DB_TaktOutputs"
    udt_tg_takt Takt_Output;
    udt_tg_takt Edge_Output;
END_DATA_BLOCK
```

2. **Assign the UDTs in the FB call:**

```plc
CALL "FB_Taktgenerator" (
    Takt_Signals := "DB_TaktOutputs".Takt_Output,
    Edge_Signals := "DB_TaktOutputs".Edge_Output
);
```

---

## Detailed Functionality

### Time_Read Region

```plc
REGION Time_Read
    // Read the current system time and convert it to the TIME format
    #ret := RD_SYS_T(#CurrentTime_DTL);
    #CurrentTime := TOD_TO_TIME(IN := DTL_TO_TOD(IN := #CurrentTime_DTL));
END_REGION
```

This region reads the current system time and converts it into the TIME format to calculate time intervals for the Takt signals.

### Interval_Assignment Region

```plc
REGION Interval_Assignment
    // Assign the predefined Takt intervals to the appropriate array elements
    FOR #i := 0 TO 9 DO
        CASE #i OF
            0: #Takt_Intervals[#i] := #const_100MS;
            1: #Takt_Intervals[#i] := #const_200MS;
            2: #Takt_Intervals[#i] := #const_400MS;
            3: #Takt_Intervals[#i] := #const_500MS;
            4: #Takt_Intervals[#i] := #const_800MS;
            5: #Takt_Intervals[#i] := #const_1S;
            6: #Takt_Intervals[#i] := #const_1S_600MS;
            7: #Takt_Intervals[#i] := #const_2S;
            8: #Takt_Intervals[#i] := #const_4S;
            9: #Takt_Intervals[#i] := #const_5S;
        END_CASE;
    END_FOR;
END_REGION
```

This region assigns the time intervals (e.g., 100ms, 200ms) to the `Takt_Intervals` array, which will be used to control the timing of each Takt signal.

### Takt_Signal_Update Region

```plc
REGION Takt_Signal_Update
    // Iterate over all frequencies and update each Takt signal based on the time difference
    FOR #i := 0 TO 9 DO
        IF (#CurrentTime >= #StartTime[#i]) THEN
            #TimeDifference := #CurrentTime - #StartTime[#i];
        ELSE
            #TimeDifference := (#MAX_TIME - #StartTime[#i]) + #CurrentTime;  // Handle time overflow
        END_IF;

        // Toggle the Takt signal if the time difference exceeds the interval
        IF (#TimeDifference >= #Takt_Intervals[#i]) THEN
            #Internal_TaktSignals[#i] := NOT #Internal_TaktSignals[#i];
            #StartTime[#i] := #CurrentTime;
        END_IF;
    END_FOR;
END_REGION
```

This region calculates the time difference for each Takt signal and toggles the signal when the time interval has passed. It handles the potential overflow of the TIME value.

### Takt_Signal_Assignment Region

```plc
REGION Takt_Signal_Assignment
    // Assign the internal Takt signals to the output struct fields
    FOR #i := 0 TO 9 DO
        CASE #i OF
            0: #Takt_Signals.Takt_10Hz := #Internal_TaktSignals[0];
            1: #Takt_Signals.Takt_5Hz := #Internal_TaktSignals[1];
            2: #Takt_Signals.Takt_2_5Hz := #Internal_TaktSignals[2];
            3: #Takt_Signals.Takt_2Hz := #Internal_TaktSignals[3];
            4: #Takt_Signals.Takt_1_25Hz := #Internal_TaktSignals[4];
            5: #Takt_Signals.Takt_1Hz := #Internal_TaktSignals[5];
            6: #Takt_Signals.Takt_0_625Hz := #Internal_TaktSignals[6];
            7: #Takt_Signals.Takt_0_5Hz := #Internal_TaktSignals[7];
            8: #Takt_Signals.Takt_0_25Hz := #Internal_TaktSignals[8];
            9: #Takt_Signals.Takt_0_2Hz := #Internal_TaktSignals[9];
        END_CASE;
    END_FOR;
END_REGION
```

This region assigns the internal Takt signals to the output structure (`Takt_Signals`), which can then be accessed by the user.

### Edge_Detection Region

```plc
REGION Edge_Detection
    // Detect edges (positive flank) for each Takt signal
    FOR #i := 0 TO 9 DO
        // Detect a positive edge (Takt signal changes from FALSE to TRUE)
        #Pulse_Trigger[#i] := #Internal_TaktSignals[#i] XOR #Edge_Flag[#i];

        // Update the edge flag for the next cycle
        #Edge_Flag[#i] := #Internal_TaktSignals[#i];
    END_FOR;
END_REGION
```

This region detects the positive edge (rising flank) for each Takt signal. The `Pulse_Trigger` will be TRUE if a positive edge is detected.

### P_Trig_Assignment Region

```plc
REGION P_Trig_Assignment
    // Assign the Pulse Trigger signals to the appropriate output fields
    FOR #i := 0 TO 9 DO
        CASE #i OF
            0: #Edge_Signals.Takt_10Hz := #Pulse_Trigger[#i];
            1: #Edge_Signals.Takt_5Hz := #Pulse_Trigger[#i];
            2: #Edge_Signals.Takt_2_5Hz := #Pulse_Trigger[#i];
            3: #Edge_Signals.Takt_2Hz := #Pulse_Trigger[#i];
            4: #Edge_Signals.Takt_1_25Hz := #Pulse_Trigger[#i];
            5: #Edge_Signals.Takt_1Hz := #Pulse_Trigger[#i];
            6: #Edge_Signals.Takt_0_625Hz := #Pulse_Trigger[#i];
            7: #Edge_Signals.Takt_0_5Hz := #Pulse_Trigger[#i];
            8: #Edge_Signals.Takt_0_25Hz := #Pulse_Trigger[#i];
            9: #Edge_Signals.Takt_0_2Hz := #Pulse_Trigger[#i];
        END_CASE;
    END_FOR;
END_REGION
```

This region assigns the `Pulse_Trigger` values (detected edges) to the output structure (`Edge_Signals`), which can be accessed by the user.

---

## Conclusion

The **FB_Taktgenerator** provides an easy and flexible way to generate Takt

 signals and detect positive edges for multiple frequencies. By assigning UDT instances to the output variables, you can easily integrate these signals into your project.

