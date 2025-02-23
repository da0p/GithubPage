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
- Increase maximum range without increasing transmit power. Coded PHY provides
  enhanced bit error detection and correction, allowing further range while
  staying within the maximum BER. This is achieved at a cost to data rate, as it
  increases the number of symbols per bit.

### 2M PHY

- A new PHY configuration introduced in 5.0
- Increase symbol rate at the PHY layer. It achieves a symbol rate of 2 Mega
  symbols per second, where each symbol corresponds to a single bit. This allows
  a user to double the number of bits sent over the air during a given period,
  or conversely reduce energy consumption for a given amount of data by halving
  the ncessary transmit time.
- Reduced range compared to Coded PHY

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
