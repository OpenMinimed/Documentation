# Bluetooth communication

The insulin pumps and any connected devices use Bluetooth Low Energy (BLE) for communication. The pumps act as Central, while the sensors act as Peripheral. This is the only way multiple devices can be connected to a pump at the same time. It also means that any app (such as Medtronic's MiniMed Mobile app or our own scripts) must act as Peripheral when talking to the pump.

Establishing an initial connection roughly works like this: The pump is instructed (by the user) to scan for Peripherals that want to connect to it. The Peripheral sends out advertising packets that are picked up by the pump. These include some information that the pump can use to decide whether this is a suitable device or not. The pump then connects to a selected Peripheral. So it is always the pump that initiates the actual connection.

Both devices then exchange some keys (known as _pairing_ step) and store them for later use. Most importantly, this allows them to reestablish a lost connection without user interaction.

The devices do NOT use _LE Secure Connection_, meaning the communication should be crackable and sniffable if we are able to sniff the pairing process (we can reconstruct the SAKE session key)

The sensor can be easily connected with the provided [python script](/PythonConnector/). The pump is a bit different and it requires MITM protection. It still currently under investigation.

The devices utilize standardized and custom GATT services and characteristics. The interesting data (as payload in specific characteristics) is encrypted using a Medtronic protocol called SAKE (Secure? Authenticated? Key Exchange?). The protocol uses two different GATT service UUIDs: <code>0xfe82</code> for pairing and the actual data transfer, <code>0xfe81</code> when advertising for reconnecting already paired phones.

Please check out our overview of [pump's](pump-services.md) and [app's](app-services.md) GATT services as well as the [Bluetooth SIG's official specifications](https://www.bluetooth.com/specifications/specs/) for more information.


## MAC addresses

The sensor MACs start with DC:16:A2 (Medtronic Diabetes, https://standards-oui.ieee.org/oui/oui.txt), while the pump use the private OUI 00:23:f7. The lower 3 bytes are the sensor's serial number, converted to hexadecimal. For example:

    CGM GT1122867N → 1122867 = 0x112233
    → Its MAC address should be DC:16:A2:11:22:33.

There seems to be no such relation between a pump's MAC address and its serial number. The addresses look rather random.


## Device names

The mobile apps and the pumps use a pseudo device name on the GATT layer. The apps on first startup will generate a `Mobile xxxxxx` like string, where the number is always a random 6-digit number (**odd** on Android devices and **even** on iOS devices).

Pumps use their serial numbers of the form `NGxxxxxxxH` and translate them into `Pump xxxxxxxH`, keeping the 7-digit number the same.
