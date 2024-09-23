# Documentation of Timer Management in SCL

## Table of Contents

1. [Introduction](#introduction)
2. [UDT Structure](#udt-structure)
3. [Data Block (TimerDatabase)](#data-block-timerdatabase)
4. [Functions](#functions)
   - [TimerReset](#function-timerreset)
   - [TimerStart](#function-timerstart)
   - [TimerTOF](#function-timertof)
   - [TimerTON](#function-timerton)
   - [TimerPause](#function-timerpause)
   - [TimerResume](#function-timerresume)
   - [TimerHandling](#function-timerhandling)
   - [TimerElapsed](#function-timerelapsed)
5. [Summary](#summary)

## Introduction

This documentation outlines the implementation and functionality of a timer system designed using **Structured Control Language (SCL)** for PLCs. The system enables the management of multiple timers and their states in an industrial automation context. Various functions such as starting, pausing, resetting, and monitoring timers are detailed.

## UDT Structure

The user-defined data type (UDT) `udt_timer` is used to store the information related to each timer.

```scl
TYPE "udt_timer"
VERSION : 0.1
   STRUCT
      timerState : Int;      // Current state of the timer: 0=Off, 1=Running, 2=Paused, 3=Elapsed
      startTime : Time;      // Time when the timer started
      setTime : Time;        // Pre-defined duration for which the timer should run (e.g., 2500 ms)
      elapsedTime : Time;    // Time that has passed since the timer started
   END_STRUCT;
END_TYPE
```

### Field Descriptions:
- **timerState**: Indicates the current state of the timer.
- **startTime**: The time at which the timer was started.
- **setTime**: The predefined duration for the timer to run.
- **elapsedTime**: The time that has passed since the timer was started.

## Data Block TimerDatabase

The data block `TimerDatabase` contains all timer data structures along with additional management variables.

```scl
DATA_BLOCK "TimerDatabase"
{ S7_Optimized_Access := 'TRUE' }
VERSION : 0.1
NON_RETAIN
   VAR 
      timersTime : Time;        // Current system time used for timers
      timersAmount : Int;       // Total number of timers in the array
      timersPerBatch : Int;     // Number of timers processed per cycle/batch
      timersBatchCounter : Int; // Counter to track which batch of timers is being updated
      timers { S7_SetPoint := 'False'} : Array[0..599] of "udt_timer"; // Array holding the timer data structures
   END_VAR
END_DATA_BLOCK
```

### Variable Descriptions:
- **timersTime**: The system time used for timer calculations.
- **timersAmount**: The total number of timers in the system (max 600).
- **timersPerBatch**: Number of timers processed per cycle.
- **timersBatchCounter**: Counter for tracking batches.
- **timers**: Array of timer data structures (up to 600 timers).

## Functions

### Function TimerReset

This function resets the state of a specified timer to the initial state (0=Off).

```scl
FUNCTION "TimerReset" : Void
{ S7_Optimized_Access := 'TRUE' }
VERSION : 0.1
   VAR_INPUT 
      Timer_Nr : Int; // Index of the timer to be reset
   END_VAR
BEGIN
   "TimerDatabase".timers[#Timer_Nr].timerState := 0;
END_FUNCTION
```

### Function TimerStart

Starts a timer by setting its start time to the current system time.

```scl
FUNCTION "TimerStart" : Void
{ S7_Optimized_Access := 'TRUE' }
VERSION : 0.1
   VAR_INPUT 
      Timer_Nr : Int; // Index of the timer to start
   END_VAR
BEGIN
   IF "TimerDatabase".timers[#Timer_Nr].timerState = 0 THEN
      "TimerDatabase".timers[#Timer_Nr].startTime := "TimerDatabase".timersTime;
      "TimerDatabase".timers[#Timer_Nr].timerState := 1; // Timer is running
   END_IF;
END_FUNCTION
```

### Function TimerTOF

A **Timer Off-Delay** (TOF) starts the timer when the input condition is true and resets it when the condition is false.

```scl
FUNCTION "TimerTOF" : Void
{ S7_Optimized_Access := 'TRUE' }
VERSION : 0.1
   VAR_INPUT 
      TimerNr : Int; // Index of the timer
      IN : Bool;     // Input condition to start/reset the timer
   END_VAR
   VAR_OUTPUT 
      OUT : Bool;    // Output indicating whether the timer has elapsed
   END_VAR
BEGIN
   IF #IN AND ("TimerDatabase".timers[#TimerNr].timerState <> 2) THEN
      "TimerReset"(Timer_Nr := #TimerNr);
      "TimerStart"(Timer_Nr := #TimerNr);
      "TimerDatabase".timers[#TimerNr].timerState := 2;
   ELSIF NOT #IN AND ("TimerDatabase".timers[#TimerNr].timerState = 2) THEN
      "TimerReset"(Timer_Nr := #TimerNr);
      "TimerStart"(Timer_Nr := #TimerNr);
   END_IF;
   #OUT := #IN OR "TimerDatabase".timers[#TimerNr].timerState = 1;
END_FUNCTION
```

### Function TimerTON

A **Timer On-Delay** (TON) starts the timer when the input condition is true and resets it when the condition is false.

```scl
FUNCTION "TimerTON" : Void
{ S7_Optimized_Access := 'TRUE' }
VERSION : 0.1
   VAR_INPUT 
      TimerNr : Int; // Index of the timer
      IN : Bool;     // Input condition to start/reset the timer
   END_VAR
   VAR_OUTPUT 
      OUT : Bool;    // Output indicating whether the timer has elapsed
   END_VAR
BEGIN
   IF #IN AND ("TimerDatabase".timers[#TimerNr].timerState = 0) THEN
      "TimerReset"(Timer_Nr := #TimerNr);
      "TimerStart"(Timer_Nr := #TimerNr);
   ELSIF NOT #IN AND NOT ("TimerDatabase".timers[#TimerNr].timerState = 0) THEN
      "TimerReset"(Timer_Nr := #TimerNr);
   END_IF;
   #OUT := "TimerDatabase".timers[#TimerNr].timerState = 3;
END_FUNCTION
```

### Function TimerPause

Pauses a running timer.

```scl
FUNCTION "TimerPause" : Void
{ S7_Optimized_Access := 'TRUE' }
VERSION : 0.1
   VAR_INPUT 
      Timer_nr : Int; // Index of the timer to be paused
   END_VAR
BEGIN
   IF "TimerDatabase".timers[#Timer_nr].timerState = 1 THEN
      "TimerDatabase".timers[#Timer_nr].timerState := 2; // Timer paused
   END_IF;
END_FUNCTION
```

### Function TimerResume

Resumes a paused timer.

```scl
FUNCTION "TimerResume" : Void
{ S7_Optimized_Access := 'TRUE' }
VERSION : 0.1
   VAR_INPUT 
      Timer_nr : Int; // Index of the timer to resume
   END_VAR
BEGIN
   IF "TimerDatabase".timers[#Timer_nr].timerState = 2 THEN
      "TimerDatabase".timers[#Timer_nr].startTime := "TimerDatabase".timersTime - "TimerDatabase".timers[#Timer_nr].elapsedTime;
      "TimerDatabase".timers[#Timer_nr].timerState := 1; // Timer is running again
   END_IF;
END_FUNCTION
```

### Function TimerHandling

Processes timers in batches, updates elapsed time, and checks if timers have elapsed.

```scl
FUNCTION "TimerHandling" : Void
{ S7_Optimized_Access := 'TRUE' }
VERSION : 0.1
   VAR_TEMP 
      ret : Int;
      CurrentTime_DTL : DTL; // Current system time
      nStartTimerIndex : Int;
      nEndTimerIndex : Int;
      iTimerIndex : Int;
      nTimersPerBatch : Int;
      timersArraySize : Int;
   END_VAR
BEGIN
   // Read the current system time and convert it to TIME
   #ret := RD_LOC_T(#CurrentTime_DTL);
   "TimerDatabase".timersTime := TOD_TO_TIME(IN := DTL_TO_TOD(IN := #CurrentTime_DTL));
   
   // Timer update per batch
   FOR #iTimerIndex := #nStartTimerIndex TO #nEndTimerIndex DO
      IF "TimerDatabase".timers[#iTimerIndex].timerState = 1 THEN
         "TimerDatabase".timers[#iTimerIndex].elapsedTime := "TimerDatabase".timersTime - "TimerDatabase".timers[#iTimerIndex].startTime;
         IF "TimerDatabase".timers[#iTimerIndex].elapsedTime >= "TimerDatabase".timers[#iTimerIndex].setTime THEN
            "TimerDatabase".timers[#

iTimerIndex].timerState := 3; // Timer elapsed
         END_IF;
      END_IF;
   END_FOR;
END_FUNCTION
```

### Function TimerElapsed

Checks if a timer has elapsed (state 3).

```scl
FUNCTION "TimerElapsed" : Bool
{ S7_Optimized_Access := 'TRUE' }
VERSION : 0.1
   VAR_INPUT 
      Timer_Nr : Int; // Index of the timer to check
   END_VAR
BEGIN
   #TimerElapsed := "TimerDatabase".timers[#Timer_Nr].timerState = 3;
END_FUNCTION
```

## Summary

This documentation describes a comprehensive timer management system for PLC applications. The various functions enable starting, pausing, resetting, and monitoring of timers in an optimized batch process to maximize PLC performance.