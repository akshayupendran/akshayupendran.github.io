# I2C

## References

[I2C in a nutshell](https://interrupt.memfault.com/blog/i2c-in-a-nutshell)

[I2C Bus](https://www.i2c-bus.org/)

## Notes

- I2C stands for inter integrated circuit.
- I2C is cheaper than comparable low-power buses, with fewer pins than SPI, and a simpler physical layer than CAN(no differential signalling).
- It supports upto 127 devices (1-7 bits)[2^7 - 1].
- Standard Mode: 100Kbps; Fast Mode: 400 Kbps; Fast Mode Plus: 1Mbps (anything above 400Kbps).
- For higher bandwidth than I2C people go for SPI like in NOR Flash chips. Even higher like camera interfaces, use MIPI.
- If reliablity is a must, then go for CAN.
- If only a single device than use UART.
- I2C is made up of two signals: a clock (SCL) and a Data (SDA) lines.
- By default the lines are pulled high by resistors. If anyone wants to use the lines, they will pull it low.
- As I2C is a multi-master bus, signalling is done via START and STOP conditions.\
- A START is to pull SDA LOW with SCL HIGH. When SDA goes HIGH while SCL is HIGH, its a STOP.
- The 7 bits after a START is the ADDRESS.
- The Single bit after the Address is READ/WRITE COMMAND. A 1 means the COMMAND is READ and a 0 means the COMMAND is WRITE.
- A common pattern is to send a WRITE followed by a READ. (REPEATED_START_CONDITION)
- So it goes like <S><ADDRESS_OF_SLAVE><W><REG_ADDRESS><S><ADDRESS_OF_SLAVE><R><DATA><P> (REPEATED_START_CONDITION)
- The I2C Protocol states that every byte must be acknowledged by the receiver.

- So a typical WRITE is:
> S><ADDRESS_OF_SLAVE><W><ACK><ADDRESS_REG_2_WRITE><ACK><DATA_2WRITE><ACK><P>
- A typical READ is:
> S><ADDRESS_OF_SLAVE><W><ACK><ADDRESS_REG_2_WRITE><ACK><S><ADDRESS_OF_SLAVE><R><ACK><DATA><NACK><P>
- A NACK by the master in the above read means that the master does not want any more data.

- Clock Stretching is a feature where the slave can hold the SCL line low. In this case the Master should wait for the line to become high before sending any data.

#UART

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
