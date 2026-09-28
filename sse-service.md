# Secure Session Establishment Service

Custom Medtronic service (`00000700-0000-1000-0000-009132591325`) of the pump. Reverse engineered from MiniMed Mobile 2.5.0 (`gf.SecureSessionEstablishment*`); the app only sends requests that carry no operand.

Characteristic                        | Name              | Properties (per this documentation) 
--------------------------------------|-------------------|-----------------
00000701-0000-1000-0000-009132591325 | SSE Control Point | Write, Indicate
00000702-0000-1000-0000-009132591325 | SSE Data          | Write w/o response, Notify

All fields are little-endian. The opcode field is a **u8**.

## Opcodes

Value | Definition
------|-----------
0x01  | Response
0x02  | Get Status
0x03  | Get Status Response
0x04  | Accept Begin
0x05  | Create
0x06  | Close
0x07  | Accept End

## Requests (Control Point, write)

Field  | Data Type | Size (octets)
-------|-----------|--------------
Opcode | u8        | 1

Requests have no operand in the app.

## Responses (Control Point, indicate)

_Response_ (0x01):

Field           | Data Type | Size (octets)
----------------|-----------|--------------
Opcode          | u8 (0x01) | 1
Request Opcode  | u8        | 1
Response Code   | u8        | 1

_Get Status Response_ (0x03):

Field  | Data Type | Size (octets)
-------|-----------|--------------
Opcode | u8 (0x03) | 1
Status | u8        | 1

Response Code | Definition
--------------|-----------
0x01          | Success
0x02          | Operation not supported
0x03          | Invalid operand
0x04          | Operation not completed

Status | Definition
-------|-----------
0x01   | Secured
0x02   | In progress
0x03   | Not secured

The data characteristic carries the actual handshake payload (see [Certificate Management](cm-service.md) and the app-side "secure session" code); its format is not documented yet.
