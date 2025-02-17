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

- THY is set after a connection by examining the capabilities and configuration
  for both devices in a procedure that is known as the PHY Update Procedure.
- Master or slave can initiate the procedure, but the master has the final
  decision

### Remarks on 2M PHY

- 2M PHY doesn't mean double throughput, only 33% increase in reality. There are
  other factors affecting the throughput: interframe spacing and connection
  event length
- Even though time to transmit is reduced by half, but the interframe spacing
  time is kept the same. Thefore, more time devoted to interframe spacing
- Connectin event length is another factor. TX - RX exchanges must fit within
  the event length duration.
- Duplex communication can also increase throughput since it eliminates the
  empty response, hence reducing interframe spacing
- Connection interval is the time between two data transfer events between the
  central and the peripheral device

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
