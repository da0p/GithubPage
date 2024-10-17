---
title: "CAN and CANOpen"
date: 2024-10-17
---
# CAN
## Main Characteristics
- Works under harsh environment conditions, resilient to electromagnetic interference
- High error tolerance
- Maximal bus length is limited by signal propagation time on the media
- Short frame length
- Data rates up to 1 Mbit/s

## Standard Data Frame

![Can Frame](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/CAN-Frame.drawio.png)

- **SoF**: Start of frame (1 dominant bit): Indicated the begin of a data or remote frame. All devices synchronize on the failing edge of the SoF that is sent first after the bus has been in idle state.
- **Arbitration Field**
  - *Identifier Field*: 11-bit identifier
  - *RTR (Remote Transmission Request)*: distinction between data frame (RTR = 0) and remote frame (RTR = 1)
- **Control Field**: Specification of the data block length in the 4 lower bits (DLC)
- **Data Field (0.. 8 byte)**: contains a data segment with at most 8 byte of user data
- **CRC Field**: CRC sequence plus CRC delimiter bit. Frame verification sequence is built over SoF, Identifier Field, CRC, Control Field, and Data Field (not considering stuff bits)
- **CRC Delimiter**: time for comparison of locally calculated and received CRC polynomial
- **Acknowledge field**
  - *ACK Slot* + *ACK delimiter*: all devices that correctly received the frame (matching CRC sequences), apply a dominant signal level during the ACK slot on bus as confirmation to the transmitting device
- **End of frame field**: Indication of end of frame
- **Intermission**: Gap between two frames

## Extended Data Frame
- 29-bit identifier
- Differentiation between the two formats: we have the IDE bit, if IDE = 0, then the format is extended format
- Both formats can coexist on the bus

## CAN Arbitration
- Depending on the *Arbitration Field*, the lower, the higher priority
- When a master sends with a lower arbitration, other nodes will stop transmission and switch to listen mode

## CAN Bit Stuffing
- In CAN, more than 5 consecutive bits of the same polarity means error
- If there are more than 5 consecutive bits of the same polarity, a bit with reverse polarity is inserted to avoid signaling errors.

## CAN Error Handling
- There are *Active Error Frame* and *Passive Error Frame*
- *Active Error Frame* is sent when CAN is in active state, otherwise, *Passive Error Frame* is sent

## CAN Error Detection
Errors

- **Bit monitoring**: Not the same value read than send (not during arbitration process)
- **Bit stuffing**: More than 5 consecutive bits of the same level
- **Frame check**: Fixed format(CRC Delimiter, ACK Delimiter, End of Frame)
- **Acknowledgement check**: No dominant level in the ACK slot
- **Cyclic Redundancy Check**: 15-bit CRC

If an error is detected, the node that detects will send an error frame. Error counter will be incremented, and the message will be resent.

![Can Error Limitation](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/CAN-Error-Limitation.drawio.png)

