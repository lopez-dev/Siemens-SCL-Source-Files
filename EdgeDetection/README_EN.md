# Function: EdgeDetectionNeg

## Description:
This function detects a **falling edge** (negative edge) from an input signal. It outputs a boolean indicating the detection of the falling edge.

### Inputs:
- **InputSignal (Bool):** The signal from which the edge is detected.

### In/Out Variables:
- **NegEdgeOut (Bool):** Output variable indicating the detection of a falling edge (negative edge).
- **NegEdgeMem (Bool):** Memory flag that stores the previous state of the `InputSignal`. This is used to detect the falling edge.

### Logic:
1. The function detects a falling edge when the `InputSignal` transitions from **TRUE to FALSE**.
2. If a falling edge is detected, the `NegEdgeOut` output is set to TRUE.
3. The previous state of the `InputSignal` is stored in the `NegEdgeMem` to detect future edges.

---

# Function: EdgeDetectionPos

## Description:
This function detects a **rising edge** (positive edge) from an input signal. It outputs a boolean indicating the detection of the rising edge.

### Inputs:
- **InputSignal (Bool):** The signal from which the edge is detected.

### In/Out Variables:
- **PosEdgeOut (Bool):** Output variable indicating the detection of a rising edge (positive edge).
- **PosEdgeMem (Bool):** Memory flag that stores the previous state of the `InputSignal`. This is used to detect the rising edge.

### Logic:
1. The function detects a rising edge when the `InputSignal` transitions from **FALSE to TRUE**.
2. If a rising edge is detected, the `PosEdgeOut` output is set to TRUE.
3. The previous state of the `InputSignal` is stored in the `PosEdgeMem` to detect future edges.
