# Firmware Update Service

Custom Medtronic service (`00000800-0000-1000-0000-009132591325`) of the pump. Reverse engineered from MiniMed Mobile 2.5.0 (`p427we.FirmwareUpdate*`). The app only uses the control point to start/confirm/cancel an update; the payload transfer over the data characteristic is not documented yet.

Characteristic                        | Name                          | Properties
--------------------------------------|-------------------------------|-----------------
00000801-0000-1000-0000-009132591325 | Firmware Update Control Point | Write, Indicate
00000802-0000-1000-0000-009132591325 | Firmware Update Data          | Write w/o response, Notify

All fields are little-endian. The opcode field is a **u8**.

## Opcodes

Value | Definition
------|-----------
0x01  | Response
0x02  | Get Status
0x03  | Download
0x04  | Confirm
0x05  | Cancel

## Requests

Field  | Data Type | Size (octets)
-------|-----------|--------------
Opcode | u8        | 1

## Response (0x01)

Field          | Data Type | Size (octets)
---------------|-----------|--------------
Opcode         | u8 (0x01) | 1
Request Opcode | u8        | 1
Response Code  | u8        | 1
Error Code     | u8        | 1

Response Code | Definition
--------------|-----------
0x01          | Success
0x02          | Operation not supported
0x03          | Invalid operand
0x04          | Operation not completed

Error Code | Definition
-----------|-----------
0x01       | User cancelled
0x02       | Battery level insufficient
0x03       | Unacknowledged alarm
0x04       | Bolus in progress
0x05       | PLGM suspended
0x06       | Block mode enabled
0x07       | FOTA not possible
0x08       | Backup battery level insufficient
0xff       | Undefined error
