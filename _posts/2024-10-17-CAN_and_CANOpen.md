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

![CAN Error Limitation](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/CAN-Error-Limitation.drawio.png)

# CANOpen
## Device Model
Every CANOpen device has to implement certain standard features in its controlling software.
- **Communication Unit** implements the protocols for messaging with the other nodes in the network. Starting and resetting the device is controlled via a state machine. It must contains Initialization, Pre-operational, Operational, and Stopped (NTM SM)
- **Object Dictionary** is an array of variables with a 16-bit index. Additionally, each variable can have an 8-bit subindex. 
- **Application** performs the desired function of the device, after the state machine is set to operational state

![CAN Device Model](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/CAN_DeviceModel.drawio.png)

## Object Dictionary
- An array with 65535 (0xFFFF) entries
- Divided into different areas
- Application data is stored here

## Process Data Objects (PDO)
- Very efficient data transmission based on producer-consumer model
- Transfer at most 8 byte of data with no protocol overhead
- PDOs correspond to objects in the object dictionary
- Number and length of PDOs is device-specific, and is specified in device or application profiles or alternatively manufacturer specific.
- CANOpen distinguishes between transmit (TPDO) and receive (RPDO) process data objects

## Communication Profile
- Part of Object Dictionary
- Include information how data are placed in a PDO
- How PDOs can be sent

## Service Data Objects (SDO)
- Provide access to all entries in the Object Dictionary of a device
- Possible to transfer data with more than 8 bytes but requiring segmentation in the SDO protocol, flow control
- Upload/download of data requires the specification of the service type to be executed within SDO protocol
- SDO transfer types:
  - normal transfer: any number of data bytes
  - expedited transfer: fast transfer, up to 4 bytes
  - block transfer (bulk data)
- Client/Server model, communication directly between two nodes

## Network Management (NMT)
![CAN NMT](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/CAN_NMT.drawio.png)

|Transition            |Description                                                                        |
|----------------------|-----------------------------------------------------------------------------------|
| (1)                  | At power on, the NMT state initialization                                         |
| (2)                  | NMT state initialization finished - enter NMT state Pre-operational automatically |
| (3)                  | NMT service start remote node indication or by local control                      |
| (4), (7)             | NMT service enter pre-operational indication                                      |
| (5), (8)             | NMT service stop remote node indication                                           |
| (6)                  | NMT service start remote node indication                                          |
| (9), (10), (11)      | NMT service reset node indication                                                 |
| (12), (13), (14)     | NMT service reset communication indication                                        |

## Layer Setting Service (LSS)
- LSS is used to change CANOpen nodeID and CAN bit rate without using HW components
- LSS is based on master-slave model
- LSS slaves require a unique signature by which they can be addressed
- LSS address is a 128-bit value, which consists of the 4 sub-indexes of the identity object in the object dictionary
- LSS Services:
  - Switch state
  - Configuration
  - Inquire
  - Identification
  - Fastscan
- LSS uses two reserved CAN identifier (0x7E4 and 0x7E5)
- LSS services use a fixed CAN frame length of 8 data bytes
- LSS uses command specifiers (0x00 to 0x7F) to identify the commands