# JUDI Handshake

Handshake example:
server:
``` JSON
{
  "message_id": 0,
  "request":"device_info", 
}
```
client: 
``` JSON
{
  "message_id": 0, 
  "update": {
    "device_info": {
      "product_name":"Example Product",
      "serial_number":"123456789ABCDEF",
      "firmware_version":"1.2",
      "protocol_version":"1.0"
    }
  }
}
```
Alternate format:
``` JSON
{
  "message_id": 0, 
  "update": "device_info", 
  "device_info": {
    "product_name":"Example Product",
    "serial_number":"123456789ABCDEF",
    "firmware_version":"1.2",
    "protocol_version":"1.0"
  }
}
```
# Handshake Subfields
All subfields are strings unless otherwise noted.


Mandatory subfields:
- `protocol_version`
  - Which version of the JUDI protocol is supported
  - The only version so far is "1.0"
- `product_name`
  - The model of the device
- `serial_number`
  - A serial number that can be used to uniquely identify the individual device

Optional subfields
- `firmware_version`
  - The device's firmware version number
- `alias`
  - An alias or nickname that can some devices support
- `hardware_version`
  - Some devices can report thier hardware version
- `supported_baud_rates` (list of integers)
  - If the device supports baud rate negotiation, the available baud rates will be listed here