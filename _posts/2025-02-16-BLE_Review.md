---
title: "Bluetooth Low Energy Review"
date: 2025-02-16
---

## Bluetooth Classic

- Also referred to as **Bluetooth Basic Rate/Enhanced Data Rate (BR/EDR)**
- 79 channels in 2.4GHz unlicensed ISM frequency band
- Point-to-point communication, mainly used to enable _wireless audio streaming_

## Bluetooth Low Energy (LE)

- Over 40 channels in 2.4GHz ISM band
- Support multiple communication topologies: point-to-point, to broadcast, and
  most recently, mesh
- Can be used now as a device positioning technology to address the increasing
  demand for high accuracy indoor location services

<!-- prettier-ignore -->
|                          | Bluetooth Low Energy                                                                                              | Bluetooth Classic                                                                  |
| ------------------------ | ----------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------- |
| Frequency Band           | 2.4 GHZ ISM (2.402 - 2.480 GHz)                                                                                   | 2.4GHz ISM (2.402 - 2.480 GHz)                                                     |
| Channels                 | 40 channels with 2 Mhz spacing <br>(3 advertising channels)                                                       | 79 channels with 1 MHz spacing                                                     |
| Channel Usage            | Frequency hopping spread spectrum (FHSS)                                                                          | Frequency hopping spread spectrum (FHSS)                                           |
| Modulation               | GFSK                                                                                                              | GFSK, pi/4 DQPSK, 8DPSK                                                            |
| Data Rate                | LE 2M PHY: 2 Mb/s <br> LE 1M PHY: 1Mb/s<br>LE Coded PHY (S-2): 500 Kb/s<br>LE Coded PHY(S-8): 125 Kb/s            | EDR PHY (8DPSK): 3 Mb/s<br> EDR PHY (pi/4 DQPSK): 2 Mb/s,br> BR PHY (GFSK): 1 Mb/s |
| Tx Power                 | <= 100 mW (+20 dBm)                                                                                               | <= 100 mW (+20 dBm)                                                                |
| Rx Sensitivity           | LE 2M PHY: <= -70 dBm<br>LE 1M PHY: <= -70 dBm<br>LE Coded PHY (S=2): <= -75 dBm<br>LE Coded PHY (S=8) <= -82 dBM | <= -70 dBm                                                                         |
| Data Transports          | Asynchronous, Isochronous connection-oriented<br> async, sync, isochronous connectionless                         | Async, sync connection-oriented                                                    |
| Communication Topologies | PtP <br> Broadcast <br> Mesh                                                                                      | PtP                                                                                |
| Positioning Features     | Presence: advertising <br> Direction: Direction Finding (AoA/ AoD) <br> Distance: RSSI, Channel Sounding          | None                                                                               |

## Physical Layer

### 1M PHY

- 1 Megabit PHY, commonly referred to as 1M PHY, has been the de facto Bluetooth
  PHY up to this point.

### Coded PHY

- A new PHY configuration introduced in 5.0.
- Increase maximum range without increasing transmit power (4 time range). Coded
  PHY provides enhanced bit error detection and correction (**Forward Error
  Correction**), allowing further range while staying within the maximum BER.
  This is achieved at a cost to data rate, as it increases the number of symbols
  per bit.
- **Trade-offs**:
  - **Higher power consumption:** since transmitting multiple symbols to
    represent one bit of data, resulting in longer radio-on time to transmit the
    same amout of data
  - **Reduced speeds:** due to the fact that more bits are needed to transmit
    the same amount of data (125 kbps or 500 kbps, depending on the **coding
    scheme** used)

### 2M PHY

- A new PHY configuration introduced in 5.0
- Increase symbol rate at the PHY layer. It achieves a symbol rate of 2 Mega
  symbols per second, where each symbol corresponds to a single bit. This allows
  a user to double the number of bits sent over the air during a given period,
  or conversely reduce energy consumption for a given amount of data by halving
  the necessary transmit time.
- Reduced range compared to Coded PHY
- Improvement of wireless coexistence because of the decreased radio-on time
- **Restrictions:** 2M PHY is not allowed in primary advertisements. Two ways to
  utilize this mode:
  - Secondary advertisements (**extended advertising mode**) are used and sent
    on the 2M PHY, which allow a connection on that PHY from the central device.
  - Advertising on the primary or secondary channels using the 1M or the coded
    PHY. A connection is then established, and either side can request a PHY
    update to use the 2M PHY during the connection
- Link between a central and a peripheral can be asymmetric, meaning that the
  packets from the peripheral can be sent using the 1M PHY while packets from
  the central can be sent using the 2M PHY

### PHY Configuration

- PHY is set after a connection by examining the capabilities and configuration
  for both devices in a procedure that is known as the PHY Update Procedure.
- Master or slave can initiate the procedure, but the master has the final
  decision

### Remarks on 2M PHY

- 2M PHY doesn't mean double throughput, only 33% increase in reality. There are
  other factors affecting the throughput: interframe spacing and connection
  event length
- Even though time to transmit is reduced by half, but the interframe spacing
  time is kept the same. Thefore, more time devoted to interframe spacing
- Connecting event length is another factor. TX - RX exchanges must fit within
  the event length duration.
- Duplex communication can also increase throughput since it eliminates the
  empty response, hence reducing interframe spacing
- Connection interval is the time between two data transfer events between the
  central and the peripheral device

### Remarks on Data Throughput

Data rate is much lower than the radio data rate due to the following factors:

- **Gaps in between packets:** 150 microseconds between packets being
  transmitted as a requirement for adhering to the specification. This gap is
  time lost with no data being exchanged between two devices.
- **Packet overhead:** All packets include header information and data handled
  at levels lower than the application level, which count towards the data being
  transmitted but are not part of the data utilized by your application.
- **Slave data packets requirement:** The requirement to send back data packets
  from the slave, even when no data needs to be sent back and empty packets are
  sent.
- **Retransmission of data packets:** In the case of packet loss or interference
  from devices in the surrounding environment, the lost or corrupted data
  packets get resent by the sender.

## GAP and GATT

- **GAP (Generic Access Profile):** defines the **device discovery and
  connections** of the BLE network stack
- **GATT (Generic Attribute Profile):** manages how attributes (data) are
  transferred once devices have a dedicated connection. **GATT** specifically
  focuses on how data is formatted, packaged, and sent according to its
  described rules.

![BLE Stack](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/ble_stack_review.drawio.png)

## BLE Advertisement

- Broadcast data without the need of connection
- To connect and exchange data

### Channels

- 37 data channels
- 3 primary advertising channels
- Extended advertising channels (5.0)
- The primary advertising channels are selected for avoiding interference with
  WiFi
- Advertising simultaneously on 3 channels

### Parameters

- Advertising interval: 20ms - 10.24 s

### Scanner

Two important parameters: scan interval and scan window.

- Active scanning
- Passive scanning

![BLE Advertisement](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/ble_advertisement.drawio.png)

### Advertisement Types

| Type            | Connectable | Scannable | Directed |
| --------------- | ----------- | --------- | -------- |
| ADV_IND         | y           | y         | n        |
| ADV_DIRECT_IND  | y           | n         | y        |
| ADV_NONCONN_IND | n           | n         | n        |
| ADV_SCAN_IND    | n           | y         | n        |

### Advertising Parameters

- Advertising intervals: from 20 milliseconds up to 10.24 seconds in small
  increments of 625 microseconds

- Advertising/Scan Response Data: Follows TLV used in data communication
  (Type-Length-Value), except length comes before
  ![BLE Advertising Parameters](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/advertising_parameters.drawio.png)

### Extended Advertisements

- **Secondary advertising channels** concept is introduced in Bluetooth 5, which
  allows the device to offload data to advertise more data than what's allowed
  on the primary advertisement channels.
- Advertisement packets sent on the secondary advertisement channels can use any
  of three PHYs (1M PHY, 2M PHY, or coded PHY), whereas the primary
  advertisement channels can only use the coded PHY or the original 1M PHY
- A central should use 1M PHY or coded PHY when initially searching for
  peripherals that are sending out advertising packets.

### Periodic Advertisements

- A special case of **extended advertisements** and allow a central to
  "synchronize" to a peripheral that is sending these extended advertisements at
  a fixed interval
- Periodic advertisements are transmitted on the primary channels, which hold
  information to help locate the extended advertisement packet.

## Filter Policy

There are three types of policy

- Advertising (peripheral side)
- Scanning (central side)
- Initiator (central side)

Device filtering gets processed at the link layer in the controller (the lower
layer of the Bluetooth stack), which saves time and overhead from being
performed at the host (the upper layer of the stack). However, the host is
responsible for configuring the white list.

### Adversiting Filter Policy

Filter scan and connection requests. There are 4 available modes that can be
configured by the host

| Scan Request | Connection Request | Modes           |
| ------------ | ------------------ | --------------- |
| n            | n                  | Default         |
| y            | y                  | Only Scan       |
| n            | y                  | Only Connection |
| y            | y                  | Both            |

### Scanner Filter Policy

Filter advertising & scan Response PDUs. There are 2 available modes that can be
configured by the host

| Adv PDU | Scan Request | Modes              |
| ------- | ------------ | ------------------ |
| n       | n            | Unfiltered Default |
| y       | y            | Filtered           |

### Initiator Filter Policy

Filter Connectable Advertising PDUs. There are 2 available modes. Requests from
devices not in the white list or not specified by host are ignored

## Bluetooth Address

Bluetooth devices are identified by a 48-bit address, similar to a MAC address.
Two main types of addresses: public addresses and random addresses.

### Public Address

- A fixed address that does not change and is factory-programmed
- Must be registered with the IEEE

### Random Address

- More popular since no need to register with IEEE
- Programmed on the device or generated at runtime
- Two sub-types: static address and private address

#### Static Address

- Used as a replacement for public addresses
- Can be generated at boot up or stay the same during lifetime
- Cannot change until a power-cycle

#### Private Address

Two sub-types: non-resolvable and resolvable private address

##### Non-resolvable Private Address

- Random, temporary for a certain time, but not common

##### Resolvable Private Address

- Used for privacy
- Generated using Identity Resolving Key (IRK) and a random number
- Changes periodically to avoid being tracked by unknown scanners
- Trusted devices (or bonded) can resolve it using the previously stored IRK

## Connection

Note that after a connection is established, the central becomes the master, and
the peripheral becomes the slave

![BLE Connections](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/ble_connections.drawio.png)

### Connection Events

- During a connection event, the master and slave exchange data packets until
  neither side has more data left to send
- A connection event occurs periodically and continuously until the connection
  is closed or lost
- A connection event contains at least one packet sent by the master
- If the master does not receive a packet back from the slave, the master will
  close the connection event and it resumes sending packets at the next
  connection event
- The connection event can be closed by either side
- The starting points of consecutive connection events are spaced by a period of
  time called the connection interval

### Connection Parameters

- **Connection interval:** 7.5 milliseconds - 4.0 seconds in increments of 1.25
  milliseconds. The central may take into account the _Peripheral Preferred
  Connection Parameters (PPCP)_, which is a way for the peripheral to informt he
  central of a set of parameters that it prefers.
- **Slave Latency:** allows the peripheral to skip a number of consecutive
  connection events and not listen to the central at these connection events
  without compromising the connection.
- **Supervision Timeout:** Detect a loss in connection.
- **Data Length Extension (DLE):** can be enabled or disabled. It allows the
  packet size to hold a larger amount of payload (up to 251 bytes vs. 27 bytes
  when disabled). This feature was introduced in 4.2 reducing the packet
  overhead and any unnecessary header data that gets transmitted with smaller
  packets.
- **Maximum Transmission Unit (MTU):** define the maximum size of a PDO that can
  be sent by a specific protocol. The **Attribute MTU** is the largest size of
  an ATT payload that can be sent between a client and a server.

## Services and Characteristics

- **Generic Attribute Profile (GATT)** only comes into play after a connection
  has been established between two BLE devices.

### Attribute Protocol (ATT)

ATT defines how a server exposes its data to a client and how this data is
structured. Two roles within the ATT:

- **Server:** This is the device that exposes the data it controls or contains.
  It accepts incoming commands from a peer device, and sends **responses**,
  **notifications**, and **indications**.
- **Client:** This is the device that interfaces with the server with the
  purpose of reading the server's exposed data and/or controlling the server's
  behavior. It is the device that sends commands and requests and aceepts
  incoming notifications and indications.

The data that the server exposes is structured as **attributes**. An attribute
is the generic term for any type of data exposed by the server and defines the
structure of this data. Services and characteristics are types of attributes.
Attributes are made up of the following: **atribute type**, **atribute handle**,
**attribute permissions**

- **Atrribute Type (Universally Unique Identifier or UUID):** This is a 16-bit
  number for Bluetooth SIG-adopted attributes, or 128-bit number for custom
  attribute types
- **Attribute Handle:** A 16-bit value that the server assigns to each of its
  attributes. It is considered as an address that the client can use to
  reference a specific attribute, and is guaranteed by the server to uniquely
  identify the attribute during the life of the connection between two devices.
  The range of handles if 0x0001 - 0xFFFF, there the value of 0x0000 is
  reserved.
- **Attribute Permissions:** Permissions determine whether an attribute can be
  _read_ or _written_ to, whether it can be _notified_ or _indicated_, and what
  _security levels_ are required for each of these operations. These permissions
  are not defined or discovered via the _Attribute Protocol (ATT)_, but rather
  defined at a higher layer (GATT layer or Application layer).

### Generic Attribute Profile (GATT)

- **Services**, **characteristics**, and **profiles** are used specifically to
  allow hierarchy in the structuring of the data exposed by the server.
- GATT defines the format of services and their characterisrics, and the
  procedures that are used to interface with these attributes such as service
  discovery, characteristic reads, characteristic writes, notifications, and
  indications.
- GATT takes on the same roles as the Attribute Protocol (ATT). The roles are
  not set per device, but per transaction.
- A service can refer to another service. The main service is called **primary
  service** representing the primary functionality of a device, while the
  referred one is called \*secondary service\*\* providing auxiliary
  functionality of a device.

![BLE Services](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/ble_services.drawio.png)

### Characteristics

A **characteristic** is always part of a service and it represents a piece of
information/data that a server wants to expose to a client.

- **Properties:** represented by a number of bits and which defines how a
  characteristic value can be used including _read_, _write_, _write without
  response_, _notify_, _indicate_.
- **Descriptors:** used to contain related information about the characteristic
  value inlcuding _extended properties_, _user description_, fields used for
  subscribing to notifications and indications, and a field that defines the
  presentation of the value such as the format and the unit of the value.

## Profiles

- Profiles are much broader in definition than services. They includes defining
  the behavior of both the client and server when it comes to services,
  characteristics, and even connections and security requirements. Services and
  their specifications, on the other hand, deal with the implementation of these
  services and characteristics on the server side only.
- There are SIG-adopted profiles that have published specifications.

## Attribute Operations

Six different types of attribute operations including

- **Commands:** sent by the client to the server and do not require a response
- **Requests:** sent by the client to the server and require a response. Two
  types of requests:
  - Find Information Request
  - Read Request
- **Responses:** sent by the server in response to a request
- **Notifications:** sent by the server to the client to let the client know
  that a specific characteristic value has changed. In order for this to be
  triggered and sent by the server, the client has to enable notifications for
  the characteristic of interest. Note that a notification does not require a
  response from the client to acknowledge its receipt
- **Indications:** sent by the server to the client. They are very similar to
  notifications, but require an acknowledgment to be sent back from the client
  to let the server know that the indication was successfully received.
- **Confirmations:** sent by the client to the server. These are the
  acknowledgment packets sent back to the server to let it know that the client
  successfully received an indication

Note that **notifications** and **indications** are exposed via the **Client
Characteristic Configuration Descriptor (CCCD)** attribue. Writing a "1" to this
attribute value enables notifications, whereas writing a "2" enables
indications. Writing a "0" disables both notifications and indications.

### Reading Attributes

**Reads** are **requests** by nature since they require a response. There are
four different types of reads. Two most important ones are:

- **Read Request:** a simple request referencing the attribute to be read by its
  **handle**
- **Read Blob Request:** similar to the read request but adds an offset to
  indicate where the read should start, returning a portion of the value. This
  type of read is used for reading only part of a characteristic's value

### Writing to Attributes

**Writes** can be either commands or requests.

- **Write Request:** requires a response from the server to acknowledge that the
  attribute has been successfully written to
- **Write Command:** this has no response from the server
- **Queued Writes (atomic opereation behavior):** Used when a large value needs
  to be written and does not fit within a single message. Two steps are involved
  - Prepare values request, send them to the server, the server will buffer the
    prepared values
  - Execute write request, used to request from the server to either execute or
    cancel the write operation of the prepared values
