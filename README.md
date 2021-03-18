# JSON Universal Device Interface - JUDI

JUDI is a general purpose serial protocol for interfacing between software and embedded systems. 


# Protocol 

A JUDI messaging relationship has two endpoints: a `client` (the software running on your PC) and a `server` (some physical device connected to your PC via a serial port).

```
{"request":"device_info"}
```

