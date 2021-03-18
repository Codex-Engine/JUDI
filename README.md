# JSON Universal Device Interface - JUDI

JUDI is a general purpose serial protocol for interfacing between software and embedded systems. 

# Design Goals

The project requirements that spawned JUDI:
- a product line of multiple types of widgets
- control software running on a PC
- multiple widgets connected at the same time
- multiples of the same kind of widget at the same time
- widgets are simple: an 8bit MCU and an FTDI USB->UART converter
- no messing with USB vendor or product IDs
- the end user should NEVER have to mess with manually selecting a com port
- some widgets have physical UI like buttons and LEDs
- some devices are headless sensor packages with no controls

I also had previous experience with frustrating serial devices that:
- had no way of identifying them (you MUST figure out what com port they're on)
- only operated at weird baud rates
- would not speak unless spoken to (could only be interacted with by polling)
- zero configurability, even when the product itself was begging for it
- extremely difficult to differentiate multiple units
- one especially dumb device had a physical controls and a serial port, would not send updates when you changed settings with the knobs, and locked out the physical controls for multiple seconds every time it received serial data, which made polling it impossible


These raw requirements and previous pains led to a set of design goals:
- device autodetection, with minimal (or zero) user interaction
- ability to identify and remember an individual device, even if there are multiple of the same kind
- all communication must be in-band: we can't rely on the USB driver to identify a device
- platform agnostic, it should be easy to write control software in any language that can open a serial port
- must support different styles of devices:
  - sensor packages that only send data updates
  - devices with actions but no sensors that only execute commands
  - devices with physical ui like buttons and leds, such that neither serial commands nor button presses cause any ui to get out of sync

These goals ended up producing a JUDI protocol that is deeply unopinionated: it happily works anywhere you can transmit ascii characters in full duplex. It been used over RS-232, USB virtual com ports, Bluetooth emulated serial, and even TCP sockets and websockets.

# Protocol Features

## Endpoints

A JUDI messaging relationship has two endpoints: a `client` (the software running on your PC) and a `server` (some physical device connected to your PC via a serial port).

## Format 
JUDI messages are sent in plaintext JSON. There's no other fundamental restrictions.

## Message Types
The core JUDI spec has the following message types:
- `command`
  - sent from client to server
  - instructs the server to perform an action
  - commands can take time to execute
  - commands change the state of the server
- `request`
  - sent from client to server
  - instructs the server to send an update with the specified information
  - the response should be nearly immediate
  - should not have side effects or change the state of the server
- `response`
  - sent from server to client
  - usually a response to a command
  - usually contains a status code or error message
- `update`
  - sent from server to client
  - can be an answer to a request, or unsolicited 

## Message IDs
Messages from the client can include an optional message id field. If the message id is present, any responses or updates related to the original message will contain a matching message id field.

The message id can either be a JSON int or a string.

The client can choose message ids any way it wants. The simplest scheme is to count the number of sent messages and use the count as the id.

## Serial Number
A device that supports the JUDI protocol should contain a unique serial number. This is used to store settings in the pc software and to maintain a persistent connection even if the serial port changes.

# Standard Messages
The JUDI protocol has a number of standard messages that should be implemented regardless of the desired functionality of your device.
## Handshake
Support for this message is mandatory, but some of the subfields are optional.

A detailed description of the handshake fields can be found in [handshake.md](https://github.com/Codex-Engine/JUDI/blob/master/handshake.md)

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
      "hardware_version":"B",
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
    "protocol_version":"1.0"
  }
}
```

## Locate
The locate command is not mandatory, but is strongly encouraged. When the device receives the locate command, it should blink all of its external LEDs for 5-10 seconds to assist in physically locating the device, wherever it might be installed. 

server:
``` JSON
{
  "message_id": 1,
  "command":"locate", 
}
```
client: 
``` JSON
{
  "message_id": 1, 
  "response":"ok"
}
```