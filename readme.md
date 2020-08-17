Akshay's Knowledge Base
=======================

This website serves more as a personal knowledge base which I would use for spaced repetition of various topics I have read and understood.
The blog would mainly be oriented towards my profession automotive embedded cyber security.

Topics Covered
--------------

- [I2C](Communication/I2C.md)

# UART

## References

i.MX8 Manual

## Notes
- UART stands for Universal Asynchronous Receiver / Transmitter
- UART has a receiver, a transmitter and a baud rate generator.
- The pins are TX, RX, CTS(Clear to Send), RTS(Request To Send).
- UART is similar to I2c as both are NRZ (NON Return to zero) protocols.
- Our receiver controls the RTS and the CTS controls our transmitter (CTS is input pin, RTS is output pin).
- Typically is 8 bits between Start and Stop bits (which is called 8 bit mode).
- 9 bit mode is used for 8 bits + parity.
- We can have 1 or 2 stop bits.

# CAN

## References

## Notes
- A Standard CAN has a 11 bit identifier.
- An extended CAN has a 29 bit identifier.
- ISO 11898
- UDS is ISO 14229
- Lower the identifier, higher the priority.
- 0 is called "dominant" - 1 is "recessive"
- If one transmits a dominant bit and another a receccive bit, the dominant bit wins (arbitration)
- Six bit clocks between the frames.
- There are mainly four types of CAN frames:
  - Data frame
  - Remote frame
  - Error frame
  - Overload frame

# SPI

# NVM

# OSAL Layers
