# History And Trace Service

This custom Medtronic service gives access to the pump's complete history and internal diagnostics. It is likely involved in uploading this data to CareLink.


## Repository And Transfer Management Control Point (RTMCP)

### Opcodes

Value | Definition
------|------------
0x01  | Response Code
0x02  | Repository Request
0x03  | Repository Request Status Response
0x04  | Cancel Repository Request
0x05  | Get Repository Request Status
0x06  | Get Next Transfer Block
0x07  | Get Slice Transport Parameters
0x08  | Set Slice Transport Parameters
0x09  | Slice Transport Parameters Response
0x0a  | Get Transfer Block Parameters
0x0b  | Set Transfer Block Parameters
0x0c  | Transfer Block Parameter Response
0x0d  | End Transfer Session
0x0e  | Get Session Metrics
0x0f  | Session Metrics Response

The opcodes that contain "response" in the name are used only in responses, the other ones only when sending commands.

### Format of _Response Code_ response

This is a special response. The pump sends it as general response to some commands and also as a way to report errors (or more general: return codes) in its _Response Code Value_ field. The original opcode that started the command is included in the _Request Opcode_ field.

### Response structure

Field Name                  | Data Type    | Size (octets) | Unit
----------------------------|--------------|---------------|------
Opcode                      | Value 0x01   | 1             | None
Request Opcode              | Enum of u8   | 1             | None
Response Code Value         | Enum of u8   | 1             | None

The following values are defined for the _Response Code Value_ field:

Value | Definition
------|------------
0x00  | Success
0x01  | Opcode Not Supported
0x02  | Invalid Operand
0x03  | Procedure Not Completed
0x04  | Parameters Out Of Range
0x05  | Procedure Not Applicable


### Format of _Repository Request_ command and response

While code for this command is found in the MMM, the 780G pump sends back responses of type _Response Code_ with value _Opcode Not Supported_. So maybe this is just a leftover from development and not supported by the pumps. Its replacement is likely the _Secure Repository Request_ in the [_RMCP SE_ characteristic](#repository-management-control-point-se-rmcp-se).

#### Command structure

Field Name                  | Data Type    | Size (octets) | Unit
----------------------------|--------------|---------------|------
Opcode                      | Value 0x02   | 1             | None
Request Type                | Enum of u8   | 1             | None
Request Control Flags       | 8 bits       | 1             | None
Device Source ID            | u32          | 4             | None
History Type                | Enum of u8   | 1             | None
Instance ID                 | u8           | 1             | None
Token Type                  | Enum of u8   | 1             | None
Start Index                 | u32          | 4             | None
End Index                   | u32          | 4             | None

The following values are defined for the _Request Type_ field:

Value | Definition
------|------------
0x00  | Asynchronous Request
0x01  | Transactional Request
0x02  | Session Encrypted Asynchronous Request
0x03  | Session Encrypted Transactional Request

Bits in the _Request Control Flags_ field are defined as follows:

Bit | Definition                  | Description
----|-----------------------------|-------------
0   | Multi Block Enable          |
1   | Force Re-cache              |
2   | Requested Index Granularity |

The following values are defined for the _History Type_ field:

Value | Definition
------|------------
0x00  | Repository List
0x01  | Pump History
0x02  | Pump Diagnostic Trace
0x03  | Pump Detailed Trace
0x04  | Pump Sensor History
0x05  | GST Trace
0x06  | BGM Logbook
0x07  | Pump User Settings
0x09  | All BGM

The following values are defined for the _Token Type_ field:

Value | Definition
------|------------
0x00  | Absolute Reference
0x01  | Relative Reference From End

#### Response structure

???


### Format of _Cancel Repository Request_ command and response

#### Command structure

This command has no operands.

Field Name                  | Data Type    | Size (octets) | Unit
----------------------------|--------------|---------------|------
Opcode                      | Value 0x04   | 1             | None

#### Response structure

???


### Format of _Get Repository Request Status_ command and response

#### Command structure

This command has no operands.

Field Name                  | Data Type    | Size (octets) | Unit
----------------------------|--------------|---------------|------
Opcode                      | Value 0x05   | 1             | None

#### Response structure

Field Name                  | Data Type    | Size (octets) | Unit
----------------------------|--------------|---------------|------
Opcode                      | Value 0x03   | 1             | None
Repository Status Flags     | 8 bit        | 1             | None
Repository Request Status   | Enum of u8   | 1             | None
Transfer Session ID         | u32          | 0 or 4        | None
Block Continuation Sequence | u8           | 0 or 1        | None
Total Transfer Blocks       | u8           | 0 or 1        | None
Re-cache Result             | Enum of u8   | 0 or 1        | None

Bits in the _Repository Status Flags_ field are defined as follows:

Bit | Definition                          | Description
----|-------------------------------------|-------------
0   | Transfer ID Available               | If this bit is set, fields _Transfer Session ID_ and _Block Continuation Sequence_ are present
1   | Additional Transfer Block Available |
2   | Total Transfer Blocks Reported      | If this bit is set, field _Total Transfer Blocks_ is present
3   | Re-cache Result                     | If this bit is set, field _Re-cache Result_ is present

The following values are defined for the _Repository Request Status_ field:

Value | Definition
------|------------
0x00  | Transfer Block Ready
0x01  | Transfer Block Pending
0x02  | Unable To Process Transfer Block
0x03  | Transfer Session Expired
0x04  | Transfer Session Invalid
0x05  | Info Not Available
0x06  | Block Sequence Number Invalid
0x07  | Secure Session Unavailable
0x08  | Repository Does Not Exist

The following values are defined for the _Re-cache Result_ field:

Value | Definition
------|------------
0x00  | Operation Successful
0x01  | Operation Unsuccessful


### Format of _Get Next Transfer Block_ command and response

#### Command structure

Field Name                  | Data Type    | Size (octets) | Unit
----------------------------|--------------|---------------|------
Opcode                      | Value 0x06   | 1             | None
Transfer Session ID         | u32          | 4             | None
Block Sequence Number       | u8           | 1             | None

#### Reponse structure

???


### Format of _Get Slice Transport Parameters_ command and response

#### Command structure

This command has no operands.

Field Name                  | Data Type    | Size (octets) | Unit
----------------------------|--------------|---------------|------
Opcode                      | Value 0x07   | 1             | None

#### Response structure

Field Name                  | Data Type    | Size (octets) | Unit
----------------------------|--------------|---------------|------
Opcode                      | Value 0x09   | 1             | None
Slice Size                  | u16          | 2             | bytes


### Format of _Set Slice Transport Parameters_ command and response

#### Command structure

Field Name                  | Data Type    | Size (octets) | Unit
----------------------------|--------------|---------------|------
Opcode                      | Value 0x08   | 1             | None
Slice Size                  | u16          | 2             | bytes (?)

#### Response structure

The response is the same as the one to the [_Get Slice Transport Parameters_](#format-of-get-slice-transport-parameters-command-and-response) command.


### Format of _Get Transfer Block Parameters_ command and response

#### Command structure

This command has no operands.

Field Name                  | Data Type    | Size (octets) | Unit
----------------------------|--------------|---------------|------
Opcode                      | Value 0x0a   | 1             | None

#### Response structure

Field Name                  | Data Type    | Size (octets) | Unit
----------------------------|--------------|---------------|------
Opcode                      | Value 0x0c   | 1             | None
Compression Setting         | Enum of u8   | 1             | None
Encryption Setting          | Enum of u8   | 1             | None

The following values are defined for the _Compression Setting_ field:

Value | Definition
------|------------
0x00  | None
0x01  | Normal

The following values are defined for the _Encryption Setting_ field:

Value | Definition
------|------------
0x00  | None
0x01  | Full Block Session Encryption
0x02  | Payload Session Encryption
0x03  | Payload Default Encryption


### Format of _Set Transfer Block Parameters_ command and response

#### Command structure

Field Name                  | Data Type    | Size (octets) | Unit
----------------------------|--------------|---------------|------
Opcode                      | Value 0x0b   | 1             | None
Parameter Type              | Enum of u8   | 1             | None
Encryption Setting          | Enum of u8   | 0 or 1        | None
Compression Setting         | Enum of u8   | 0 or 1        | None

The following values are defined for the _Parameter Type_ field:

Value | Definition          | Description
------|---------------------|-------------
0x00  | Compression Setting | If this value is set, field _Compression Setting_ is present
0x01  | Encryption Setting  | If this value is set, field _Encryption Setting_ is present

Values for fields _Compression Setting_ and _Encryption Setting_ are as defined for the [_Get Transfer Block Parameters_](#format-of-get-transfer-block-parameters-command-and-response).

#### Response structure

The response is the same as the one to the [_Get Transfer Block Parameters_](#format-of-get-transfer-block-parameters-command-and-response) command.


### Format of _End Transfer Session_ command and response

#### Command structure

Field Name                  | Data Type    | Size (octets) | Unit
----------------------------|--------------|---------------|------
Opcode                      | Value 0x0d   | 1             | None
Transfer Session ID         | u32          | 4             | None

#### Response structure

???


### Format of _Get Session Metrics_ command and response

#### Command structure

Field Name                  | Data Type    | Size (octets) | Unit
----------------------------|--------------|---------------|------
Opcode                      | Value 0x0e   | 1             | None
Session Metric ID           | u8           | 1             | None

#### Response structure

Field Name                  | Data Type    | Size (octets) | Unit
----------------------------|--------------|---------------|------
Opcode                      | Value 0x0f   | 1             | None
Session Metric ID           | Enum of u8   | 1             | None
Compression Time            | u32          | 0 or 4        | ms
Encryption Time             | u32          | 0 or 4        | ms

The following values are defined for the _Session Metric ID_ field:

Value | Definition              | Description
------|-------------------------|-------------
0x00  | Transfer Block Creation | If this value is set, fields _Compression Time and Encryption Time_ are present
0xff  | Invalid

The special value `0xffffffff` marks invalid times.



## Repository Management Control Point SE (RMCP SE)

### Command structure

Field Name                  | Data Type    | Size (octets) | Unit
----------------------------|--------------|---------------|------
Opcode                      | Enum of u8   | 1             | None
Request Type                | Enum of u8   | 1             | None
Request Control Flags       | 8 bit        | 1             | None
Secure Token Size           | u8           | 1             | bytes
Secure Token                | variable     | 0–48          | None

The following values are defined for the _Opcode_ field:

Value | Definition
------|------------
0xa2  | Secure Repository Request

Values for fields _Request Type_ and _Request Control Flags_ are as defined for the [_Repository Request_](#format-of-repository-request-command-and-response) in the _RMCP_ characteristic.

### Response structure

???
