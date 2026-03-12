# NOS service

This custom Medtronic service deals with the _Network Operational State_. It contains only a single characteristic.


## NOS Port

This characteristic can be both written and read. The meaning of its data depends on the _Opcode_ field. Different opcodes are allowed/defined for read and write operations:

### Command structure (Write)

Field Name              | Data Type   | Size (octets) | Unit
------------------------|-------------|---------------|------
Opcode                  | u8          | 1             | N/A
Min Connection Interval | u16         | 2             | ???
Max Connection Interval | u16         | 2             | ???
Slave Latency           | u16         | 2             | ???
Supervision Timeout     | u16         | 2             | ???

The following values are allowed for the _Opcode_ field:

Value | Description
------|------------
0x00  | Active Mode
0x01  | Observation Mode
0x03  | Terminate Connection
0x04  | Get Effective MTU

Only messages with an _Opcode_ value of _Observation Mode_ allow you to provide values for the remaining fields. Set them to zero in all other cases.

### Response structure (Read/Notify)

Field Name              | Data Type   | Size (octets) | Unit
------------------------|-------------|---------------|------
Opcode                  | u8          | 1             | N/A
Result                  | u16[4]      | 8             | N/A

The following values are defined for the _Opcode_ field:

Value | Description
------|------------
0x00  | Active Mode
0x01  | Observation Mode
0x02  | Transition Mode
0x04  | Get Effective MTU

The _Result_ field takes on different meanings depending on the _Opcode_.

For _Opcode_ values _Active Mode_, _Observation Mode_, _Transition Mode_:

Field Name              | Data Type   | Size (octets) | Unit
------------------------|-------------|---------------|------
Min Connection Interval | u16         | 2             | ???
Max Connection Interval | u16         | 2             | ???
Slave Latency           | u16         | 2             | ???
Supervision Timeout     | u16         | 2             | ???

For _Opcode_ value _Get Effective MTU_:

Field Name              | Data Type   | Size (octets) | Unit
------------------------|-------------|---------------|------
Effective MTU           | u16         | 2             | N/A
Padding                 | u16[3]      | 6             | N/A

The _Padding_ field does not contain any meaningful data and can be ignored.
