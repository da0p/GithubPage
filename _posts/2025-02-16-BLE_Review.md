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

| Characteristics          | Bluetooth Low Energy                                                                                              | Bluetooth Classic                                                                  |
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

## Security

Security in BLE is handled by the **security manager (SM)** layer of the
architecture. The security manager defines the protocols and algorithms for
generating and exchanging keys between two devices. It involves five security
features:

- **Pairing:** the process of creating shared **secret keys** between two
  devices
- **Bonding:** the process of creating and storing shared **secret keys** on
  each side (central and peripheral) for use in subsequent connections between
  the devices.
- **Authentication:** the process of verifying that the two devices share the
  same secret keys.
- **Encryption:** the process of encrypting the data exchanged between the
  devices. Encryption in BLE uses the 128-bit AES Encryption standard, which is
  a **symmetric-key** algorithm (meaning that the same key is used to encrypt
  and decrypt the data on both sides).
- **Message Integrity:** the process of signing the data, and verifying the
  signature at the other end. This goes beyond the simple integrity check of a
  calculated CRC.

In version 4.2, the concept of **LE Secure Connections (LESC)** is introduced,
which makes the communication much more secure compared to the methods used in
earlier version of Bluetooth.

The Security Manager addresses the different security concerns as follows:

- **Confidentiality** via encryption
- **Authentication** via pairing & bonding
- **Privacy** via resolvable private addresses
- **Integrity** via digital signatures

In BLE, the master device is the **initiator** of security procedures. The slave
(**responder**) may request the start of a security procedure by sending a
security request message to the master, but it is up to the master to then send
the packet that officially starts the security process.

![BLE Security](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/ble_security.drawio.png)

**Pairing** is the combination of Phases 1 and 2. **Bonding** is represented by
Phase 3 of the process. One important thing to note is that Phase 2 is the only
phase that differs between LE Legacy Connections and LE Secure Connections.

### Pairing and Bonding

Note that pairing is a temporary security measure that does not persist across
connections. It has to be initiated and completed each time the two devices
reconnect and would like to encrypt the connection between them. In order to
extend the encryption across subsequent connections, bonding must occur between
the two devices.

#### Phase One

- The slave may request the start of the pairing process
- The master initiates the pairing process by sending a **pairing request**
  message to the slave, which then responds with a **pairing response** message.
- The pairing request and pairing response messages represent an exchange of the
  features supported by each device, as well as the security requirements for
  each device. Each of these messages include the following:
  - **Input Output (IO) capabilities:** display support, keyboard support,
    yes/no input support
  - **Out-Of-Band (OOB)** method support
  - **Authentication requirements:** includes MITM protection requirement,
    bonding requirement, secure connections support
  - **Maximum encryption key size** that the device supports
  - The different **security keys** each device is requesting to use

The information exchanged between the two devices in this phase determines the
pairing method used.

#### Phase Two

**Phase two** differs based on which method is used: **LE secure connections**
or **LE legacy connections**.

- **Legacy Connections:** In **legacy connections**, there are two keys used:
  the **temporary key (TK)** and the **short term key (STK)**. The TK is used
  along with other values exchanged between the two devices to generate the STK.

- **Secure Connections:** in **secure connections**, the pairing method does not
  involve exchanging keys over the air between the two devices. Rather, the
  devices utilize the ECDH protocol to each generate a **public/private key**
  pair. The devices then exchange the public keys only, and from that generate a
  shared secret key called the **long term key (LTK)**

#### Phase Three

**Phase three** represents the **bonding** process. This is an optional phase
that's utilized to avoid the need to re-pair on every connection to enable a
secure communication channel.

The result of bonding is that each devices stores a set of keys that can be used
in each subsequent connection and allows the devices to skip the pairing phase.
These keys are exchanged between the two devices over a link that's encrypted
using the keys resulting from phase two.

### Pairing Methods

Legacy Connections and Secure Connections each have different Pairing Methods.
Some of the methods share the same name, but the process and the data exchanged
differs among them.

#### LE Legacy Connections

- **Just Works:** TK is set to 0. Least secure of all methods
- **Out of Band (OOB):** The TK is exchanged between the two devices over a
  technology other than BLE - **near field communication (NFC)** being the main
  one.
- **Passkey:** TK is a six-digit number that is transferred between the devices
  by the end-user.

##### LE Secure Connections

- **Just Works:** the **public keys** for each device along with other generated
  values get exchanged between the two devices over BLE.
- **Out of Band (OOB):** the values are exchanged over a medium other than BLE.
  If the used medium is secure, then this makes the connection more secure.
- **Passkey:** an identical six-digit number is used. The six-digit number could
  either be entered by the user into each device, or one of the devices will
  generate it for the user to manually enter it into the other deivce.
- **Numeric Comparison:** Works the same as the **just works** method described
  above but adds an extra step at the end. This extra step allows protection
  from MITM attacks. This is the most secure pairing method of all methods.

### Privacy

- BLE provides a privacy feature to safeguard against users tracking. A device
  can use a frequenly changing private address for its Bluetooth address that
  only trusted devices can resolve.
- A trusted device in this case is a bonded device. The random private address
  is generated using a key called the **identity resolving key (IRK)**, which is
  exchanged between two bonded devices during phase three. This way, the peer
  device has access to the IRK and can resolve the random address.

### Different Security Keys

- **Temporary Key (TK):** Generation of the temporary key (TK) depends on the
  pairing method chosent. The TK is generated each time the pairing process
  occurs. TK is used in legacy connections only.
- **Short Term Key (STK):** Generated from the TK exchanged between the devices.
- **Long Term Key (LTK):** Generated and stored during phase three of the
  security process in legacy connections and during phase two in LE secure
  connections. It gets stored on each of the two devices that are bonded, and
  used in subsequent connections between the two devices.
- **Encrypted Diversifier (EDIV) and Random Number (Rand):** Two values are used
  to created and identify the LTK, stored during the bonding process.
- **Connection Signature Resolving Key (CSRK):** Used to sign data and verify
  the signature attached to the data at the other end. This key is stored on
  each of the two bonded devices.
- **Identiy Resolving Key (IRK):** Used to resolve random private addresses.
  This key is unique per device, so the master's IRK will get stored on the
  slave side, and the slave's IRK will be stored on the master side.

### Security Modes and Levels

Two security modes in BLE: **Security mode 1** and **security mode 2**. Security
mode 1 is concerned with encryption whereas security mode 2 is concerned with
data signing.

#### Security Mode 1

- **Level 1:** No security (no authentication and no encryption)
- **Level 2:** Unauthenticated pairing with encryption
- **Level 3:** Authenticated pairing with encryption
- **Level 4:** Authenticated LE secure connections pairing with encryption

#### Security Mode 2

- **Level 1:** Unauthenticated pairing wiht data signing
- **Level 2:** Authenticated pairing with data signing

A link is considered **authenticated** or **unauthenticated** based on the
pairing method used.

A link between two devices operates in one security mode only but can operate at
different levels within that mode (different characteristics may require
different levels of security)

## Bluetooth Mesh

The goal of Bluetooth mesh is to increase the range of BLE networks and add
support for more industrial applications that utilize BLE technology.

Two main benefits of a mesh network:

- **Extended Range:** Nodes can relay messages to far-away nodes via the nodes
  in between them, this allows a network to extend its ranges and expand the
  reach of devices
- **Self-healing capabilities:** _self-healing_ refers to the fact that there is
  no single point of failure. If a node drops from the mesh network, the other
  nodes can still participate and send messages to one another.

### Architecture of Bluetooth Mesh

![BLE Mesh Stack](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/bluetooth_mesh_stack.drawio.png)

#### Nodes

Devices that are part of a Bluetooth mesh network are called **nodes**. Devices
that are not part of the network are called **unprovisioned** devices. Once an
unprovisioned device gets provisioned, it joins the network and becomes a
**node**.

#### Elements

A **node** may contain multiple parts which can be controlled independently.
These different parts of a single node are referred to as **elements**.

#### States

Elements can be in various conditions, represented by **state** values. A change
from one state to another is called a **state transition**. A **transition
period** is the time to transit from one state to another.

#### Properties

Properties add some context to a state value. There are two types of properties:

- **Manufacturer property:** provides read-only access
- **Admin property:** provides read-write access

#### Messages

All communications in Bluetooth mesh are message-oriented, and nodes send
messages to control or relay information to each other.

There are three types of messages in Bluetooth mesh:

- **GET** message: a message to request the state from one or more nodes
- **SET** message: a message to change the value of a given state
- **STATUS** message: a status message is used in different scenarios
  - Sent in response to a GET message, containing the state value
  - Sent in response to an acknowledged SET message
  - Sent independently of any message to report the element's status

#### Addresses

Messages in a Bluetooth mesh network must be sent to and from an address. There
are three types of addresses:

- **Unicast Address:** an address that uniquely identifies a single node
  assigned during the **provisioning process**
- **Group Address:** an address used to identify a group of nodes. A **group
  address** usually reflects a physical grouping of nodes such as all nodes
  within a specific room
  - SIG-Fixed Group Address: All-proxies, All-friends, and All-nodes group
    addresses
  - Dynamic Group Address, which is defined by the user via a configuration
    application
- **Virtual Address:** an address that may be assigned to one or more elements,
  spanning one or more nodes. This is like a lable with 128-bit UUID

#### Publish-Subscribe

The way messages are exchanged in a Bluetooth mesh network is via the
**publish-subscribe** pattern. Typically, messages are addressed to **group** or
**virtual addresses**.

![BLE Mesh Networking](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/bluetooth_mesh_networking.drawio.png)

#### Managed Flooding

This technique is used in order to avoid flood the network with messages.
Managed flooding relies on broadcasting messages to all nodes within range of
the sender node, with a few added optimizations:

- **Messages have a TTL assigned:** limits the number of hops a message can take
  across multiple nodes within the mesh network.
- **Messages are cached:** required by all nodes and requires that messages
  received that already exist in the cache get immediately discarded.
- **Heartbeat messages are sent periodically:** used to indicate to other nodes
  that the sender is alive and active within the network.
- **Friendship:** refers to the relationship between two nodes. Two node types
  are:
  - A _low-power node_, or LPN, conserves power and it not able to receive mesh
    messages all the time. This node spends most its time with the radio turned
    off.
  - A live-powered node called the _friend node_, which can serve as a proxy for
    the LPN. The friend node caches messages for the LPN to save power, so that
    the LPN can stay asleep most of the time and only wake up occasionally. When
    the LPN wakes up, it polls the friend node to read the cached messages and
    sends any messages it needs to send to the mesh network.

#### Models

A model defines some or all functionality of a given element. There are three
categories:

- **Server model:** a collection of states, state transitions, state bindings,
  and messages which an element containing the model may send or receive
- **Client model:** does not define any state, rather, it defines only messages
  such as GET, SET, and STATUS messages sent to a server model.
- **Control model:** contains both a server and client model allowing
  communication with other server and client models.

#### Scenes

A scene is a stored collection of states and is identified by a 16-but number
which is unique within the mesh network. Scenes allow triggering one action to
set multiple states of different nodes. They can be triggered on-demand or at a
specified time.

### Node Types

Four types of nodes with optional features enabled:

- Relay Nodes
- Proxy Nodes
- Friend Nodes
- Low power nodes

#### Relay Nodes

A relay node is one that supports the relay feature

#### Proxy Nodes

A proxy node allows communication with a mesh network from a non-mesh-supported
BLE device. A proxy node acts as an intermediary and utilizes GATT operations to
allow other nodes outside of the mesh network to interface and interact with the
network.

![Proxy Node](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/proxy_nodes.drawio.png)

The protocol used in this case is called the **proxy protocol**, which is
intended to be used with a connection-enabled device (using GATT). The protocol
is built on top of GATT and allows a device to read and write proxy protocol
PDUs from GATT characteristics exposed by the proxy node. The proxy node
performs the translation between proxy protocol PDUs and mesh PDUs.

#### Friend Nodes and Low Power Nodes

A _friend node_ and a _low power node (LPN)_ are closely related to each other.
In fact, in order for a low power node to participate in a Bluetooth mesh
network, it requires a _friendship_ relationship with another node, called the
**friend node**.

- Low power nodes usually have limited power supply such as batteries
- Low power nodes may want to send messages more than receiving them
- A friend node lets low power nodes stay asleep longer
- Friend nodes cache messages for low power nodes

### Provisioning Process

It is used for adding devices to the mesh network. The device used to add a node
to the network is called the **provisioner**.

#### Step 1: Beaconing

In this step, unprovisioned devices announces its availability to be provisioned
by sending the _mesh beacon_ advertisements in the advertisement packets. This
is a new type of advertisement data type introduced in the Bluetooth mesh
standard.

#### Step 2: Invitation

When the provisioner discovers the unprovisioned device via the beacons that
were sent, it sends an **invitation** to this unprovisioned device. This uses a
new type of PDU introduced in Bluetooth mesh called the **provisioning invite
PDU**. The unprovisioned device then responds with information about its
capabilities in a **provisioning capabilities PDU**

#### Step 3: Public Key Exchange

Security in Bluetooth mesh involves the use of a combination of symmetric and
asymmetric keys such as the Elliptic-curve Diffie-Hellman (ECDH) algorithm.

#### Step 4: Authentication

The next step is to **authenticate** the unprovisioned device. This usually
requires an action by the user by interacting with both the provisioner and the
unprovisioned device. The authentication method depends on the capabilities of
both devices used.

#### Step 5: Provision Data Distribution

After authentication is complete, each device derives a **session key** using
their private key and the public key sent to it from the other device. The
session key is then used to secure the connection for exchange of additional
provisioning data, including the network key, a device key, a security parameter
known as the IV index, and a unicast address which is assigned to the
provisioned device by the provisioner. After this step, the unprovisioned device
becomes known as a node.
![Bluetooth Mesh Provision](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/bluetooth_mesh_provision.drawio.png)

### Security in Bluetooth Mesh

Security in Bluetooth mesh is mandatory

- All mesh messages are encrypted and authenticated
- Network security, application security, and device security are all handled
  independently
- Security keys can be changed during the life of the mesh network

Due to the separation of security between the network, application, and device
levels, there are three types of security keys:

- **Network key (NetKey):** possession of this shared key makes the device part
  of the network. There are two keys derived from the NetKey: the **network
  encryption key** and the **privacy key**. Possession of the NetKey allows a
  node to decrypt and authenticate up to the **network layer**, allowing the
  relay of messages, but no application data decryption.
- **Application Key (AppKey):** A key that is shared between a subset of nodes
  within a mesh network, normally those that participate in a common
  application.
- **Device Key (DevKey):** This is a device-specific key that's used during the
  provisioning process for securing communication between the unprovisioned
  device and the provisioner.

#### Node Removal

This is a procedure for removal of a node where the device is added to a
blacklist and the keys are refreshed. This process distributes new **network
keys**, **application keys**, and other relevant data to all nodes, except those
in the blacklist.

#### Privacy

A **privacy key** derived from the network key (NetKey) is used to obfuscate the
source address to prevent the device to be tracked.

#### Replay Attacks

Bluetooth mesh provides protection against replay attacks by:

- Utilizing a **sequence number (SEQ)**. Elements increment the SEQ value every
  time they publish a message. A node, receiving a message from an element which
  contains a SEQ value that's less than or equal to the one in the last valid
  message, will discard it
- Using an incrementing IV indeix, which is an additional value that also gets
  validated when a message is received
