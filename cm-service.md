# Certificate Management Service

This custom Medtronic service handles reading and updating of certificates stored in the pump.


## Certificate Management Control Point & Certificate Management Data

Write to the _Control Point_ to send commands and subscribe to receive its indications which are sent as messages with an _Opcode_ value of _Response_.

### Requests

Field Name                  | Data Type    | Size (octets) | Unit
----------------------------|--------------|---------------|------
Opcode                      | Enum of u8   | 1             | None

The following values are defined for the _Opcode_ field:

Value | Definition
------|------------
0x00  | Get Certificate
0x01  | Get Enrollment
0x02  | Set Enrollment
0x03  | Response
0x04  | Get Certificate Authority
0x05  | Set Certificate Authority
0x06  | Get Registration Authority
0x07  | Set Registration Authority
0x08  | Get Firmware Authority
0x09  | Set Firmware Authority

The data requested by the _Get X_ requests is returned in the _Certificate Management Data_ characteristic as [GATT streams](gatt-streaming.md).

> [!NOTE]
> The _Set X_ requests likely pass their data in the _Certificate Management Data_ characteristic. This still needs to be carefully tested and documented properly.

### Responses

All of the requests are answered with a response of type _Response_ whose _Request Opcode_ matches the request's _Opcode_:

Field Name                  | Data Type    | Size (octets) | Unit
----------------------------|--------------|---------------|------
Opcode                      | Value 0x03   | 1             | None
Request Opcode              | Enum of u8   | 1             | None
Response Code               | Enum of u8   | 1             | None

The following values are defined for the _Response Code_ field:

Value | Definition
------|------------
0x01  | Success
0x02  | Operation Not Supported
0x03  | Invalid Operand
0x04  | Operation Not Completed
