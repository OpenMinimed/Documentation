# App GATT services

A Bluetooth LE Peripheral wishing to pair to a 700-series pump must provide the following services:


## Generic Access Profile (GAP)

UUID   | Type            | Notes
-------|-----------------|-------
0x1800 | Primary Service |

This is just a standard GATT service as defined by the Bluetooth SIG.

### Service characteristics

UUID   | Name                       | Properties | SAKE-encrypted | Notes
-------|----------------------------|------------|----------------|-------
0x2a00 | Device Name                | Read       | no             |
0x2a01 | Appearance                 | Read       | no             |
0x2aa6 | Central Address Resolution | Read       | no             |


## Generic Attribute (GATT)

UUID   | Type            | Notes
-------|-----------------|-------
0x1801 | Primary Service |

This is just a standard GATT service as defined by the Bluetooth SIG.

### Service characteristics

UUID   | Name                       | Properties  | SAKE-encrypted | Notes
-------|----------------------------|-------------|----------------|-------
0x2a05 | Service Changed            | Indicate    | no             |
0x2b3a | Server Supported Features  | Read        | no             |
0x2b29 | Client Supported Features  | Read, Write | no             |
0x2b2a | Database Hash              | Read        | no             |


## Device Information

UUID                                 | Type            | Notes
-------------------------------------|-----------------|-------
00000900-0000-1000-0000-009132591325 | Primary Service | same as standard _Device Information Service_, UUID 0x180a

This seems to be based on the _Device Information Service_ (DIS, UUID 0x180a) defined by the Bluetooth SIG and which, interestingly, _the pump_ also uses. It is not clear why Medtronic decided to go with a custom UUID here instead.

### Service characteristics

UUID   | Name                       | Properties | SAKE-encrypted | Notes
-------|----------------------------|------------|----------------|-------
0x2a29 | Manufacturer Name String   | Read       | no             |
0x2a24 | Model Number String        | Read       | no             |
0x2a25 | Serial Number String       | Read       | no             |
0x2a27 | Hardware Revision String   | Read       | no             |
0x2a26 | Firmware Revision String   | Read       | no             |
0x2a28 | Software Revision String   | Read       | no             |
0x2a23 | System ID                  | Read       | no             | put 8 zero-bytes
0x2a50 | PnP ID                     | Read       | no             | put 7 zero-bytes
0x2a2a | IEEE 11073-20601 Regulatory Certification Data List | Read | no | leave empty


## Medtronic (SAKE)

UUID   | Type            | Notes
-------|-----------------|-------
0xfe82 | Primary Service |

This is an unnamed member service allocated for Medtronic by the Bluetooth SIG. In the context of Medtronic's insulin pumps this is related to their proprietary SAKE encryption.

### Service characteristics

UUID                                 | Name      | Properties    | SAKE-encrypted | Notes
-------------------------------------|-----------|---------------|----------------|-------
0000fe82-0000-1000-0000-009132591325 | SAKE Port | Write, Notify | no             |

App and pump use this for their initial handshake/key exchange which is used to establish Medtronic's proprietary SAKE-encrypted communication. Several pump services then use this to send encrypted data to the app.

