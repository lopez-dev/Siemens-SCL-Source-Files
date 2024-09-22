# Documentation of the SCL Code for Siemens TIA Portal

## Introduction

This documentation describes the SCL code for the Siemens TIA Portal, which provides functions for data logging in a PLC application. The code enables the creation, writing, closing, opening, and deletion of data logs and manages messages through a buffer.

## Table of Contents

- [Data Type Definition (`typ_DataLogging_DB`)](#data-type-definition-typ_datalogging_db)
- [Data Block (`DB_LogMsg`)](#data-block-db_logmsg)
- [Functions](#functions)
  - [`fc_LogMsg_InputBuffer`](#function-fc_logmsg_inputbuffer)
  - [`fc_LogMsg_CallEntry`](#function-fc_logmsg_callentry)
  - [`fc_LogMsg_NextEntry`](#function-fc_logmsg_nextentry)
  - [`fc_LogMsg_inputBool`](#function-fc_logmsg_inputbool)
- [Function Block (`FB_LogMsg`)](#function-block-fb_logmsg)
- [Usage Notes](#usage-notes)
- [Conclusion](#conclusion)

## Data Type Definition (`typ_DataLogging_DB`)

The data type `typ_DataLogging_DB` defines the structure for data logging. It contains all the necessary variables and structures for managing log entries and operations.

### Structure Definition

```scl
TYPE "typ_DataLogging_DB"
VERSION : 0.1
STRUCT
    LogEntryIndex : UInt;   // Current log entry position
    name : String := '';    // Name of the log entry
    nextPosInStack : UInt := 1;  // Next log file index
    newName : String := 'DataLog_';  // Name for new log files
    logID : DWord;  // ID of the log entry
    TurnAllOn : Bool;  // Activates all logging tags
    TurnAlloff : Bool;  // Deactivates all logging tags
    reset : Bool;  // Reset command to acknowledge errors and reset the error counter
    deleteAll : Bool;  // Deletes all log entries
    dataLogEntries : Array[0..7] of Struct
        name : String;  // Name of the log entry
        ID : DWord;     // ID of the log entry
        DLclosed : Bool;  // Indicates if the log entry is closed
    END_STRUCT;
    DLcreate : Struct
        execute : Bool;  // Execute
        done : Bool;     // Completed
        busy : Bool;     // Running
        error : Bool;    // Error
        status : Word;   // Current status
        memStatusMsg : String[200];  // Status message in text form
        memStatus : Word;  // Last stored status message
        dlogCreated : Bool;  // Log entry successfully created
    END_STRUCT;
    // Additional structures: DLdelete, DLwrite, DLnewfile, DLclose, DLopen
    // Parameter structure for logging settings
    // Buffer for messages
    // Statistical variables
    // Current log entry being processed (myData)
END_STRUCT;
END_TYPE
```

### Main Variables Description

- **LogEntryIndex**: Holds the index of the current log entry.
- **name**: Name of the current log entry.
- **nextPosInStack**: Index for the next log file in the stack.
- **newName**: Generated name for new log files.
- **logID**: ID of the current log entry.
- **TurnAllOn** / **TurnAlloff**: Controls activation or deactivation of all logging tags.
- **reset**: Resets error states and acknowledges error messages.
- **deleteAll**: Command to delete all log entries.
- **dataLogEntries**: Array storing information about each log entry.
- **DLcreate** to **DLopen**: Structures to manage various data logging operations like creating, deleting, writing, etc.
- **Parameter**: Contains settings for logging, such as maximum errors, buffer size, logging tags, etc.
- **Buffer**: Buffer to store messages to be logged.
- **Statistics**: Holds statistical information such as maximum buffer usage and error count.
- **myData**: Current log entry being written to the log.

## Data Block (`DB_LogMsg`)

The data block `DB_LogMsg` is an instance of the type `typ_DataLogging_DB` and stores runtime data and parameters for data logging.

### Initialization

```scl
DATA_BLOCK "DB_LogMsg"
{ S7_Optimized_Access := 'TRUE' }
VERSION : 0.1
"typ_DataLogging_DB"

BEGIN
    Parameter.LoggingTag[0].On := TRUE;
    Parameter.LoggingTag[0].Tag := 'System';
    Parameter.LoggingTag[1].On := TRUE;
    Parameter.LoggingTag[1].Tag := 'Infeed';
    Parameter.LoggingTag[2].On := TRUE;
    Parameter.LoggingTag[2].Tag := 'Outfeed';
    Parameter.LoggingTag[3].On := TRUE;
    Parameter.LoggingTag[3].Tag := 'Loader';
    Parameter.LoggingTag[4].On := TRUE;
    Parameter.LoggingTag[4].Tag := 'Unloader';
    Parameter.LoggingTag[5].On := TRUE;
    Parameter.LoggingTag[5].Tag := 'Deck';
    Parameter.LoggingTag[6].On := TRUE;
    Parameter.maxPosEntry := 8;
    Parameter.newRECORDS := 2000;
    Parameter.NeNamePrefix := 'DataLog_';
END_DATA_BLOCK
```

### Description

- **Parameter.LoggingTag**: Array for configuring the logging tags. Each element enables logging for a specific area (e.g., 'System', 'Infeed', 'Outfeed', etc.).
- **Parameter.maxPosEntry**: Maximum number of log entries in the stack.
- **Parameter.newRECORDS**: Maximum number of records for new log files.
- **Parameter.NeNamePrefix**: Prefix for new log files.

## Functions

### Function `fc_LogMsg_InputBuffer`

This function adds new messages to the message buffer and manages shifting existing messages.

#### Declaration

```scl
FUNCTION "fc_LogMsg_InputBuffer" : Void
{ S7_Optimized_Access := 'TRUE' }
VERSION : 0.1
VAR_INPUT 
    Logging_i : Int;
    Text : String;
END_VAR

VAR_TEMP 
    i : Int;
END_VAR
```

#### Description

- **Logging_i**: Index of the logging tag.
- **Text**: The message text to be logged.
- **i**: Temporary variable for loop iterations.

#### Functionality

- Checks if the given `Logging_i` is within the maximum number of logging tags and if logging for this tag is enabled.
- Shifts all existing messages in the buffer down by one position.
- Inserts the new message at the first position of the buffer.
- Sets the `unwritten` flag of the new message to `TRUE`, indicating it has not been written to the log yet.

#### Code Example

```scl
IF #Logging_i <= "DB_LogMsg".Parameter.MaxLoggingTags -1 THEN
    IF "DB_LogMsg".Parameter.LoggingTag[#Logging_i].On THEN
        FOR #i := "DB_LogMsg".Parameter.BufferSize - 1 TO 1 BY -1 DO
            "DB_LogMsg".Buffer[#i] := "DB_LogMsg".Buffer[#i - 1];
        END_FOR;
        
        "DB_LogMsg".Buffer[0].id := "DB_LogMsg".Parameter.NextID;
        "DB_LogMsg".Parameter.NextID := "DB_LogMsg".Parameter.NextID + 1;
        "DB_LogMsg".Buffer[0].Text := #Text;
        "DB_LogMsg".Buffer[0].tag := "DB_LogMsg".Parameter.LoggingTag[#Logging_i].Tag;
        "DB_LogMsg".Buffer[0].unwritten := TRUE;
    END_IF;
END_IF;
```

### Function `fc_LogMsg_CallEntry`

This function updates the name and ID of the current log entry based on the selected `LogEntryIndex`.

#### Declaration

```scl
FUNCTION "fc_LogMsg_CallEntry" : Void
{ S7_Optimized_Access := 'TRUE' }
VERSION : 0.1
VAR_TEMP 
    tempEntry : UInt;
END_VAR
```

#### Functionality

- Sets the `name` and `logID` from the currently selected log entry.

#### Code Example

```scl
"DB_LogMsg".name := "DB_LogMsg".dataLogEntries["DB_LogMsg".LogEntryIndex].name;
"DB_LogMsg".logID := "DB_LogMsg".dataLogEntries["DB_LogMsg".LogEntryIndex].ID;
```

### Function `fc_LogMsg_NextEntry`

This function calculates the next `LogEntryIndex` and updates `nextPosInStack` to always be one step ahead of the current index.

#### Declaration

```scl
FUNCTION "fc_LogMsg_NextEntry" : Void
{ S7_Optimized_Access := 'TRUE' }
VERSION : 0.1
VAR_TEMP 
    nextIndex : UInt;
END_VAR
```

#### Functionality

- Increments the `LogEntryIndex` by 1 and resets it to 0 when the maximum is reached.
- Updates `nextPosInStack` accordingly.

#### Code Example

```scl
nextIndex := "DB_LogMsg".LogEntryIndex + 1;

IF nextIndex > ("DB_LogMsg".Parameter.maxPosEntry - 1) THEN
    nextIndex := 0;
END_IF;

"DB_LogMsg".LogEntryIndex := nextIndex;

"DB_LogMsg".nextPosInStack := "DB_LogMsg

".LogEntryIndex + 1;

IF "DB_LogMsg".nextPosInStack > ("DB_LogMsg".Parameter.maxPosEntry -1) THEN
    "DB_LogMsg".nextPosInStack := 0;
END_IF;
```

### Function `fc_LogMsg_inputBool`

This function monitors state changes of boolean inputs and logs them.

#### Declaration

```scl
FUNCTION "fc_LogMsg_inputBool" : Void
{ S7_Optimized_Access := 'TRUE' }
VERSION : 0.1
VAR_INPUT 
    InputState : Bool;
    ActualState : Bool;
    Text : String;
    LogId : Int;
END_VAR
```

#### Description

- **InputState**: Current state of the input.
- **ActualState**: Previous state of the input.
- **Text**: The message text to be logged.
- **LogId**: ID of the logging tag.

#### Functionality

- When the input transitions from `FALSE` to `TRUE`, a message with `Text TRUE` is logged.
- When the input transitions from `TRUE` to `FALSE`, a message with `Text FALSE` is logged.

#### Code Example

```scl
IF #InputState AND NOT #ActualState THEN
    "fc_LogMsg_InputBuffer"(Logging_i := #LogId,
                            Text := CONCAT(IN1:= #Text, IN2:=' TRUE'));
ELSIF NOT #InputState AND #ActualState THEN
    "fc_LogMsg_InputBuffer"(Logging_i := #LogId,
                            Text := CONCAT(IN1:= #Text, IN2:=' FALSE'));
END_IF;
```

## Function Block (`FB_LogMsg`)

The function block `FB_LogMsg` is the core of the data logging system and manages all operations like creating, writing, closing, and deleting data logs.

### Declaration

```scl
FUNCTION_BLOCK "FB_LogMsg"
{ S7_Optimized_Access := 'TRUE' }
VERSION : 0.1
VAR 
    DataLogCreate_Instance : DataLogCreate;
    DataLogClose_Instance : DataLogClose;
    DataLogWrite_Instance : DataLogWrite;
    DataLogOpen_Instance : DataLogOpen;
    DataLogNewFile_Instance : DataLogNewFile;
    DataLogDelete_Instance : DataLogDelete;
    LoggingFlag : Bool;
    Error_flag : Bool;
    LoggingOn : Bool;
    Deleted : Array[0..7] of Bool;
    DeleteState : UInt;
    i2 : Int;
END_VAR

VAR_TEMP 
    i : Int;
    AllTagOff : Bool;
    LEnr : Array[0..7] of DWord;
END_VAR

VAR CONSTANT 
    EMPTYSTRING : String;
    LENr0 : DWord := 16#7200_1000;
    LENr1 : DWord := 16#7200_1002;
    // Additional constants up to LENr7
END_VAR
```

### Main Areas of the Function Block

#### 1. **Info**

- Contains general information and notes for using the logging system.
- Important for the correct configuration and use of the system.

#### 2. **Reset Status**

- Resets all stored status messages when a new command is entered.

#### 3. **Buffer Manager**

- Manages the message buffer.
- Checks if the buffer is full and generates error messages in case of overflow.
- Manages automatic stopping of logging if too many errors occur.
- Transfers messages from the buffer to the data log.

#### 4. **DataLogCreate**

- Manages the creation of new data logs.
- Saves the name and ID of the created log.
- Handles possible errors and stores corresponding status messages.

#### 5. **DataLogClose**

- Closes an open data log.
- Updates the status of the log entry after closing.

#### 6. **DataLogOpen**

- Opens an existing data log for write operations.
- Handles cases where the log is already open.

#### 7. **DataLogWrite**

- Writes data from the buffer to the data log.
- Monitors the write status and handles errors.

#### 8. **DataLogNewFile**

- Creates a new data log if the current one is full or if a new section is needed.
- Updates the log entry index accordingly.

#### 9. **DataLogDelete**

- Deletes existing data logs.
- Supports deleting all logs through a defined procedure.

#### 10. **Reset**

- Resets error states and statistical variables.

#### 11. **All ON/OFF**

- Allows activating or deactivating all logging tags using the `TurnAllOn` and `TurnAlloff` variables.

### Functionality

The function block operates cyclically and performs the corresponding operations depending on the flags and commands set. It continuously monitors the status of logging operations and responds to events such as buffer overflow, error states, or user commands.

### Status Messages

- Each data logging operation is assigned status codes.
- The status codes are converted into readable text messages and stored in `memStatusMsg`.
- This helps in diagnosing problems and monitoring the system.

### Example of Status Handling in `DataLogCreate`

```scl
CASE "DB_LogMsg".DLcreate.memStatus OF
    0:
        "DB_LogMsg".DLcreate.memStatusMsg := 'No errors.';
    7000:
        "DB_LogMsg".DLcreate.memStatusMsg := 'No order processing active.';
    32915:
        "DB_LogMsg".DLcreate.memStatusMsg := 'Data log already exists.';
    ELSE
        "DB_LogMsg".DLcreate.memStatusMsg := 'Unknown error code';
END_CASE;
```

## Usage Notes

- **CPU Configuration**:
  - Enable the CPU web server under PROFINET interface > Access to the web server.
  - Enable the web server in the CPU properties.
  - Create a user with admin rights (all read, write, and delete rights).
- **Important Notes**:
  - Always select the log entry to be accessed before performing a data logging command.
  - Always wait for a data logging operation to complete (`DONE` is `TRUE`) before starting the next one.
- **Error Handling**:
  - Monitor `memStatus` and `memStatusMsg` to check the status of operations.
  - Use the `reset` variable to clear error states.
- **Automatic Functions**:
  - `Auto_F_Stopp`: Automatically stops logging when the maximum number of errors is reached.
  - `Auto_open`: Automatically opens the file when writing.
  - `Auto_NewFile`: Automatically creates a new log file when the current one is full.

## Conclusion

This SCL code provides a comprehensive solution for data logging in PLC applications using Siemens TIA Portal. Through its modular structures and functions, you can flexibly adapt and manage logging to suit your requirements. The implementation includes error handling, automatic control, and extensive diagnostic options via status messages.

**Note**: Adjust parameters and settings to meet your specific requirements and the configuration of your PLC.
```