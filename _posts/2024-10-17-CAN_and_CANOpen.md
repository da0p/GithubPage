---
title: "CAN and CANOpen"
date: 2024-10-17
---

## CAN

### Main Characteristics

- Works under harsh environment conditions, resilient to electromagnetic
  interference
- High error tolerance
- Maximal bus length is limited by signal propagation time on the media
- Short frame length
- Data rates up to 1 Mbit/s

### Standard Data Frame

![Can Frame](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/CAN-Frame.drawio.png)

- **SoF**: Start of frame (1 dominant bit): Indicated the begin of a data or
  remote frame. All devices synchronize on the failing edge of the SoF that is
  sent first after the bus has been in idle state.
- **Arbitration Field**
  - _Identifier Field_: 11-bit identifier
  - _RTR (Remote Transmission Request)_: distinction between data frame (RTR
    = 0) and remote frame (RTR = 1)
- **Control Field**: Specification of the data block length in the 4 lower bits
  (DLC)
- **Data Field (0.. 8 byte)**: contains a data segment with at most 8 byte of
  user data
- **CRC Field**: CRC sequence plus CRC delimiter bit. Frame verification
  sequence is built over SoF, Identifier Field, CRC, Control Field, and Data
  Field (not considering stuff bits)
- **CRC Delimiter**: time for comparison of locally calculated and received CRC
  polynomial
- **Acknowledge field**
  - _ACK Slot_ + _ACK delimiter_: all devices that correctly received the frame
    (matching CRC sequences), apply a dominant signal level during the ACK slot
    on bus as confirmation to the transmitting device
- **End of frame field**: Indication of end of frame
- **Intermission**: Gap between two frames

### Extended Data Frame

- 29-bit identifier
- Differentiation between the two formats: we have the IDE bit, if IDE = 0, then
  the format is extended format
- Both formats can coexist on the bus

### CAN Arbitration

- Depending on the _Arbitration Field_, the lower, the higher priority
- When a master sends with a lower arbitration, other nodes will stop
  transmission and switch to listen mode

### CAN Bit Stuffing

- In CAN, more than 5 consecutive bits of the same polarity means error
- If there are more than 5 consecutive bits of the same polarity, a bit with
  reverse polarity is inserted to avoid signaling errors.

### CAN Error Handling

- There are _Active Error Frame_ and _Passive Error Frame_
- _Active Error Frame_ is sent when CAN is in active state, otherwise, _Passive
  Error Frame_ is sent

### CAN Error Detection

Errors

- **Bit monitoring**: Not the same value read than send (not during arbitration
  process)
- **Bit stuffing**: More than 5 consecutive bits of the same level
- **Frame check**: Fixed format(CRC Delimiter, ACK Delimiter, End of Frame)
- **Acknowledgement check**: No dominant level in the ACK slot
- **Cyclic Redundancy Check**: 15-bit CRC

If an error is detected, the node that detects will send an error frame. Error
counter will be incremented, and the message will be resent.

![CAN Error Limitation](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/CAN-Error-Limitation.drawio.png)

## CANOpen

### Device Model

Every CANOpen device has to implement certain standard features in its
controlling software.

- **Communication Unit** implements the protocols for messaging with the other
  nodes in the network. Starting and resetting the device is controlled via a
  state machine. It must contains Initialization, Pre-operational, Operational,
  and Stopped (NTM SM)
- **Object Dictionary** is an array of variables with a 16-bit index.
  Additionally, each variable can have an 8-bit subindex.
- **Application** performs the desired function of the device, after the state
  machine is set to operational state

![CAN Device Model](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/CAN_DeviceModel.drawio.png)

### Object Dictionary

- An array with 65535 (0xFFFF) entries
- Divided into different areas
- Application data is stored here

### Process Data Objects (PDO)

- Very efficient data transmission based on producer-consumer model
- Transfer at most 8 byte of data with no protocol overhead
- PDOs correspond to objects in the object dictionary
- Number and length of PDOs is device-specific, and is specified in device or
  application profiles or alternatively manufacturer specific.
- CANOpen distinguishes between transmit (TPDO) and receive (RPDO) process data
  objects

### Communication Profile

- Part of Object Dictionary
- Include information how data are placed in a PDO
- How PDOs can be sent

### Service Data Objects (SDO)

- Provide access to all entries in the Object Dictionary of a device
- Possible to transfer data with more than 8 bytes but requiring segmentation in
  the SDO protocol, flow control
- Upload/download of data requires the specification of the service type to be
  executed within SDO protocol
- SDO transfer types:
  - normal transfer: any number of data bytes
  - expedited transfer: fast transfer, up to 4 bytes
  - block transfer (bulk data)
- Client/Server model, communication directly between two nodes

### Network Management (NMT)

![CAN NMT](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/CAN_NMT.drawio.png)

| Transition       | Description                                                                       |
| ---------------- | --------------------------------------------------------------------------------- |
| (1)              | At power on, the NMT state initialization                                         |
| (2)              | NMT state initialization finished - enter NMT state Pre-operational automatically |
| (3)              | NMT service start remote node indication or by local control                      |
| (4), (7)         | NMT service enter pre-operational indication                                      |
| (5), (8)         | NMT service stop remote node indication                                           |
| (6)              | NMT service start remote node indication                                          |
| (9), (10), (11)  | NMT service reset node indication                                                 |
| (12), (13), (14) | NMT service reset communication indication                                        |

### Layer Setting Service (LSS)

- LSS is used to change CANOpen nodeID and CAN bit rate without using HW
  components
- LSS is based on master-slave model
- LSS slaves require a unique signature by which they can be addressed
- LSS address is a 128-bit value, which consists of the 4 sub-indexes of the
  identity object in the object dictionary
- LSS Services:
  - Switch state
  - Configuration
  - Inquire
  - Identification
  - Fastscan
- LSS uses two reserved CAN identifier (0x7E4 and 0x7E5)
- LSS services use a fixed CAN frame length of 8 data bytes
- LSS uses command specifiers (0x00 to 0x7F) to identify the commands

## SocketCAN

CAN, CANOpen, and SocketCAN can be represented as the following diagram

![OSI](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/SocketCAN.drawio.png)

Inside Linux CAN, we can find three different socket types: **CAN_RAW**,
**CAN_BCM**, and **CAN_ISOTP**

- **CAN_RAW**: Socket for CAN communication
- **CAN_BCM**: Broadcast-Manager for CAN. Allows to send periodic messages
- **CAN_ISOTP**: Transport-layered socket for ISOTP communication

![Linux CAN](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/Linux_CAN.drawio.png)

### CAN Interfaces

- canX: real CAN interfaces connected to the system
- vcanX: virtual CAN-interfaces
- slcanX: serial line CAN-interface. Socket wrapper for serial-to-CAN
  interfaces, supporting the slcan protocol
- vxcan: virtual can tunnel across network namespaces, used for forwarding
  traffic to a container

### Virtual CAN

Virtual CAN can be used for testing purpose. In order to set up a vcan interface

```bash
sudo ip link add dev vcan0 type vcan
```

Configure vcan0 Increase TX queue

```bash
sudo ip link set vcan0 txqueue 4000
```

Then bring it up

```bash
sudo ip link set vcan0 up
```

### CAN utilities

There are several CAN utilities provided by Linux

#### candump

Listen to CAN packets and filter them

![SocketCAN](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/SocketCANUseCase1.drawio.png)

#### cangw

Manage PF_CAN netlink gateway

- We can route message from one vcan to another. In order to use cangw utility,
  the driver must be loaded first.
- We can modify the packet

```bash
sudo modprobe can-gw
```

And we can add a rule to route the packets from one interface to another

```bash
cangw -A -s vcan0 -d vcan1 -e
```

We can also add a rule to filter certain packets from a canId

```bash
cangw -A -s vcan0 -d vcan1 -e -f 123:FFF
```

Besides, cangw can modify the packet such as filtering canId: #123, and
modifying that canId to #333. We also take only 4 bytes from the input

```bash
cangw -A -s vcan0 -d vcan1 -e -f 123:FFF -m SET:IL:234.4.1122334455667788
```

Also we can modify the packet data. Here we again filter the nodeId: #123 to
#333, also take full 8 bytes input, then change it to 0x1122334455667788

```bash
cangw -A -s vcan0 -d vcan1 -e -f 123:FFF -m SET:ILD:345.8.1122334455667788
```

![cangw](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/cangw.drawio.png)

## ISO-TP (ISO15765-2) and isotp utilities

### Overview

ISO 15765-2 or ISO-TP, is a communication protocol used in the automotive
industry to transmit data over a CAN bus. It's designed to provide a reliable
and efficient way to transfer

The protocol defines:

- Format of the data frames
- Flow control mechanism
- Error handling

The data frames are divided into smaller segments, which are transmitted over
the CAN bus.

ISO 15765-2 uses a multi-frame format, where a large message is divided into
multiple smaller frames and sent over the bus. Each frame is identified by a
unique ID, which allows the receiver to reassemble the frames in the correct
order and to detect missing or duplicate frames.

![cangw](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/isotp-communication.drawio.png)

### isotpsend and isotprecv

isotp is a point-to-point communication protocol. We can send a CAN message from
one canId to another.

![isotp-point-to-point](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/isotp-point-2-point.drawio.png)

We will send a string from one node to another via CAN First, we need to listen
to an isotp message

```bash
isotprecv -s 123 -d 321 vcan0 | xxd -ps -r
```

We can send a isotp message as such

```bash
echo -n "Hello World! This is gonna be a good day" | od -An -t x1 | isotpsend -s 456 -d 123 vcan0
```

![isotp-send-recv](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/isotp-send-recv.png)

Here we can see that the pattern as mentioned above. A first frame is sent,
following by a control flow frame, then consecutive frames.

### isotpserver

isotpserver is used to listen to messages from an IP device and then forward the
traffic to the corresponding CANId

![isotpserver](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/isotpserver.drawio.png)

We can start an isotpserver as such

```bash
isotpserver -l 8080 -s 123 -d 456 vcan0
```

then we can run isotpdump to check the received messages

```bash
isotpdump -s 456 -d 123 -a -u vcan0
```

And we can connect to the isotpserver to send packages

```bash
nc localhost 8080
<48656c6c6f20>
```

On the isotpdump terminal, we will see the messages going to the destination
canId: 123

### isotptun

Carries IP over CAN ISO-TP tunnel

![isotpserver](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/isotptun.drawio.png)

In this use case, we can use network namespace to simulate two physical devices.

Create network namespace

```bash
sudo ip netns add device2
```

Create a vxcan interface

```bash
sudo ip link add vxcan0 type vxcan peer name vxcan1
```

Move vxcan1 into network namespace device2

```bash
sudo ip link set vxcan1 netns device
```

Bring up vcan interfaces

```bash
sudo ip link set vcan0 up
sudo ip link set vxcan0 up
sudo cangw -A -s vcan0 -d vxcan0
sudo cangw -A -s vxcan1 -d vcan0
```

and also the one in the network namespace **device2**

```bash
sudo ip netns exec device2 ip link set vcan0 up
sudo ip netns exec device2 ip link set vxcan0 up
sudo ip netns exec device2 sudo cangw -A -s vcan0 -d vxcan0
sudo ip netns exec device2 sudo cangw -A -s vxcan1 -d vcan0
```

then let's create the tun interfaces

```bash
isotptun -s 123 -d 456 vcan0
```

also for device in the network namespace

```bash
sudo ip netns exec device2 isotptun -s 456 -d 123 vcan0
```

We then need to set up ip address for ctun0 and ctun1

```bash
sudo ip addr add 10.0.0.1 peer 10.0.0.2 dev ctun0
sudo ip link set ctun0 up

sudo ip netns exec device2 ip addr add 10.0.0.2 peer 10.0.0.1 dev ctun0
sudo ip netns exec device2 ip link set ctun0 up
```

Now we can use **tshark** to listen to traffic from one interface

```bash
sudo tshark -i ctun1
```

and we can ping the address 10.0.0.1 from 10.0.0.2

```bash
sudo tshark -i ctun0

sudo ip netns exec device2 ping 10.0.0.1
```

![isotptun_ping](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/isotptun_ping.png)

we can even run an http server

```bash
python3 -m http.server --bind 10.0.0.1
```

and then curl from the other interface in the network namespace

```bash
sudo ip netns exec device2 curl 10.0.0.1:8000/about.md
```

![isotptun_http_server](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/isotptun_http_server.png)

## J1939 and testj1939 utility

### J1939 Overview

J1939 is a set of standards defined by SAE. SAE is used for heavy-duty vehicles
such as trucks and buses, mobile hydraulics. Substandards of SAE J1939 includes

- J1939: recommended practice for a serial control & communication vehicle
  network
- J1939/11: physical layer - 250k bits/s, shielded twisted pair
- J1939/13: off-board diagnostic connector
- J1939/21: data link layer
- J1939/31: network layer
- J1939/71: vehicle application layer
- J1939/73: application layer - diagnostics
- J1939/81: network management

J1939 messages are intended to be broadcast. However, it supports also
point-to-point messages by including a specific destination address within the
message identifier. J1939 uses the 29-bit identifier defined within the CAN 2.0B
protocol.

J1939 uses 29-bit identifier defined withthe CAN 2.0B protocol

![j1939_identifier](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/j1939_message_format.drawio.png)

- **Priority**: a value of 0 has the highest priority
- **Data Page**: This bit expands the number of possible Parameter Groups that
  can be represented by the identifier
- **PDU Format**: determines the message that can be transmitted with a
  destination address or if the message is always a broadcast message.
- **PDU Specific**: If **PDU Format** is between 0 and 239, it contains the
  destination address. Otherwise, if **PDU Format** is between 240 and 255, the
  message can only be broadcast and the **PDU Specific** contains a Group
  Extension, which expands the number of possible broadcast Parameter Groups
  that can be represented by the identifier.

Parameter Group Number (PGN) is used to refer to the value of the Reserve bit,
DP, PF, and PS fields combined into a single 18 bit value.

In J1939, the Name is a 64-bit long label which gives every ECU a unique
identity. Each device on the network will be associated with at least one Name
and one Address.

Only 254 different devices of the same type can coexist on the network due to
the address limit. Address 255 is reserved as a global address for broadcast and
address 254 is reserved as the "null address" used by devices that have not yet
claimed an address or failed to claim an address.

The process of claiming an address can be described in the following diagram

![j1939_address_claim](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/j1939_address_claim.drawio.png)

Address claim is typically not used since devices on the network usually come
with a predefined address.

In order to send data, a message must be constructed with overhead that
describes the data to be sent. Messages more than 8 bytes can be sent as
multi-packet messages. Multi-packet messages are tramsmitted by means of the
Transport Protocol Functions defined in J1939/21. There are two ways of
transmitting multi-packet:

- Broadcast Announcement Message (TP_NAM)
- Connection Management (TP_CM)

### j1939cat

A utility to transfer a file with j1939

In one terminal

```bash
# Send a file from address 0x80 to 0xa0
j1939cat -i test.md vcan0:0x80 vcan0:0xa0
```

In another terminal

```bash
# Listen from j1939 address 0xa0
j1939cat vcan0:0xa0 -r
```
