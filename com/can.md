
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
