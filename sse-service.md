# Secure Session Establishment Service

SSL tunnel between pump and CareLink server


## SSE Control Point

### Opcodes

// class SseOpCode

Value | Definition
------|------------
0x01  | Response
0x02  | Get Status
0x03  | Get Status Response
0x04  | Accept Begin
0x05  | Create
0x06  | Close
0x07  | Accept End


### Format of _Response Code_ response

### Response structure

Field Name                  | Data Type    | Size (octets) | Unit
----------------------------|--------------|---------------|------
Opcode                      | Value 0x01   | 1             | None
Request Opcode              | Enum of u8   | 1             | None
Response Code Value         | Enum of u8   | 1             | None

The following values are defined for the _Response Code Value_ field:

Value | Definition
------|------------
0x01  | Success


### Format of _Get Status Response_ response

Field Name                  | Data Type    | Size (octets) | Unit
----------------------------|--------------|---------------|------
Opcode                      | Value 0x03   | 1             | None
Secure Session Status       | Enum of u8   | 1             | None

The following values are defined for the _Secure Session Status_ field:

Value | Definition
------|------------
0x01  | Secured
0x02  | In Progress
0x03  | Not Secured
