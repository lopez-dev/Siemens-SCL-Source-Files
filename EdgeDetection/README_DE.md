# Funktion: EdgeDetectionNeg

## Beschreibung:
Diese Funktion erkennt eine **fallende Flanke** (negative Flanke) von einem Eingangssignal. Sie gibt ein boolesches Signal aus, das die Erkennung der fallenden Flanke anzeigt.

### Eingänge:
- **InputSignal (Bool):** Das Signal, von dem die Flanke erkannt wird.

### In/Out Variablen:
- **NegEdgeOut (Bool):** Ausgabewert, der die Erkennung einer fallenden Flanke (negativen Flanke) anzeigt.
- **NegEdgeMem (Bool):** Speicherflag, das den vorherigen Zustand des `InputSignal` speichert. Dies wird verwendet, um die fallende Flanke zu erkennen.

### Logik:
1. Die Funktion erkennt eine fallende Flanke, wenn das `InputSignal` von **TRUE zu FALSE** wechselt.
2. Wenn eine fallende Flanke erkannt wird, wird `NegEdgeOut` auf TRUE gesetzt.
3. Der vorherige Zustand des `InputSignal` wird im `NegEdgeMem` gespeichert, um zukünftige Flanken zu erkennen.

---

# Funktion: EdgeDetectionPos

## Beschreibung:
Diese Funktion erkennt eine **steigende Flanke** (positive Flanke) von einem Eingangssignal. Sie gibt ein boolesches Signal aus, das die Erkennung der steigenden Flanke anzeigt.

### Eingänge:
- **InputSignal (Bool):** Das Signal, von dem die Flanke erkannt wird.

### In/Out Variablen:
- **PosEdgeOut (Bool):** Ausgabewert, der die Erkennung einer steigenden Flanke (positiven Flanke) anzeigt.
- **PosEdgeMem (Bool):** Speicherflag, das den vorherigen Zustand des `InputSignal` speichert. Dies wird verwendet, um die steigende Flanke zu erkennen.

### Logik:
1. Die Funktion erkennt eine steigende Flanke, wenn das `InputSignal` von **FALSE zu TRUE** wechselt.
2. Wenn eine steigende Flanke erkannt wird, wird `PosEdgeOut` auf TRUE gesetzt.
3. Der vorherige Zustand des `InputSignal` wird im `PosEdgeMem` gespeichert, um zukünftige Flanken zu erkennen.
