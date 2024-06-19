<!-- Breadcrumbs -->
< [Lab.SCHC FullSDK Documentation](/README.md) /
<!-- /Breadcrumbs -->

# The SCHC Mechanism

<!-- TOC -->

  - [Standards](#standards)
  - [Principle](#principle)
  - [Static Context](#static-context)
      - [Communication Context](#communication-context)
      - [Rules](#rules)
  - [Header Compression](#header-compression)
  - [Fragmentation](#fragmentation)
      - [Fragmentation Modes](#fragmentation-modes)
  - [The SCHC Adaptation Layer](#the-schc-adaptation-layer)
      - [About stacks](#about-stacks)

<!-- /TOC -->

SCHC (pronounce "chic") is the acronym for Static Context Header Compression, a
standard compression and fragmentation mechanism defined by the IPv6 over LPWAN
working group at the [IETF](https://www.ietf.org/).

## Standards

| Designation | IETF Standard | Usage |
| --- | --- | --- |
| SCHC | [RFC 8724](https://www.rfc-editor.org/rfc/rfc8724.html) | The global Compression & Fragmentation mechanism |
| SCHC over LoRaWAN | [RFC 9011](https://www.rfc-editor.org/rfc/rfc9011.html) | The L2 technology adaptation |
| SCHC for CoAP | [RFC 8824](https://www.rfc-editor.org/rfc/rfc8824.html) | The application adaptation |

## Principle

The principle is to transport legacy protocol data sent by connected objects in
IPv6/UDP packets that are compressed and fragmented with the SCHC mechanism, to
make them transportable over the LPWAN radio link.

## Static Context

SCHC architecture prevents continuous synchronization between elements
communicating on the network. Synchronization is actually the operation that
consumes the most bandwidth.

### Communication Context

In LPWAN networks, the nature of data flows is highly predictable. The
communication context is a collection of header fields that are recurring and
not relevant, in comparison with the payload that contains the metering data.

Lab.SCHC FullSDK stores these communication contexts both in the device and in
the network management platform, referred to as the SCHC Core, rather than
saturating the bandwidth with such information. By reducing the amount of data
transmitted over radio connectivity, using IP becomes possible.

### Rules

The SCHC mechanism translates the communication context into a set of rules,
each having a RuleID.

Rules are configured both on the SCHC Core, and on the connected objects, so
that sender and recipient share the same set of rules to describe the
communication. A "static" context assumes that the description of the rule does
not change during transmission.

With this mechanism, IPv6/UDP headers are reduced to a RuleID of 1 byte (in most
cases) within the LoRaWAN frame.

## Header Compression

Data compression is an encoding operation that shortens the size of data
transmission and/or storage. The data is restored by a decompression algorithm.

LPWAN technologies have severe constraints concerning the frame size, i.e. the
size of messages that can be sent at a time. This is due to their low data
rates, frame loss, and regulatory rules.

The IETF already produced compression schemes in the early 2000s (RoHC in 2001,
and 6LoWPAN in 2007), but these compression mechanisms cannot be applied "as-is"
to LPWAN specific networks.

- **RoHC** – This method for compressing IP, UDP, RTP and TCP headers of network
  packets is complex since the compression depends on a steep learning phase
  that varies with the network traffic and the data flows.
- **6LoWPAN** – These encapsulation and header compression mechanisms allow IPv6
  packets to be sent and received over IEEE 802.15.4 networks. However, the
  mechanisms are difficult to implement in network sensors and other constrained
  systems because of the large size of the headers (among others).

Since 2019, SCHC's header compression mechanism however makes it possible to
transmit IPv6/UDP packets over low-power wide-area networks (LPWANs). By taking
advantage of the features of LPWANs (no routing, traffic format, and highly
predictable message content) it reduces the overhead to a few bytes and saves
network traffic. And solves the difficulty encountered by RoHC and 6LoWPAN
mechanisms by the fact.

> The compression rules are configured in the SCHC Core and loaded on the device
> as a package without having to recompile the library.

## Fragmentation

Fragmentation is actually a feature of IPv4 and IPv6 that splits an initial
packet into smaller fragments named datagrams so that the resulting fragments
can be transmitted over a link with a maximum transmission unit smaller than the
packet size. Once received by the destination, a reassembly mechanisms
reconstructs the packet.

> A datagram is a data packet transmitted with its source and destination
> addresses over a telecommunications network (WAN) or a local area network
> (LAN).

- **Constraint is downlink** – SCHC includes fragmentation/reassembly to
  transmit a larger packet size than the LoRaWAN payload size and thus save the
  downlink.
- **Constraint is physical** – SCHC also incorporates a fragmentation for cases
  where compression is not sufficient to fit the limitations of the underlying
  technology. In other words, SCHC splits the messages according to the network
  requirements and manages their transmission in the network.

### Fragmentation Modes

SCHC's fragmentation mechanisms can work in three different modes: "No-Ack",
"Ack-On-Error", or "Ack-Always".

| Mode | Description |
| --- | --- |
| No-Ack | In this mode, the SCHC packet is split into several fragments which are blindly sent to the receiver. If the receiver missed a packet, it will not be able to reconstruct the sent packet. |
| Ack-On-Error | In this mode, the concept of "windows" is used. The windows have a predefined size, allowing the receiver to point out which windows or parts of windows have been received. The moment the receiver gets the last fragment sent from the sender, it will calculate which parts of the packets are missing and notify the sender. The sender will then initiate retransmission of the missing packet parts. |
| Ack-Always | In this mode, a retransmission mechanism similar to Ack-On-Error is used, with the difference that the acknowledgment is not made at the end of the transmission but for each window. |

## The SCHC Adaptation Layer

SCHC adaptation layer enables the use of the IP stack on very constrained
networks for end-to-end IP communication from the object to the application. It
dynamically adapts compression and fragmentation to the underlying network
conditions and capabilities, overcoming the variability of LoRaWAN parameters
between regions, operators, and radio conditions.

### About stacks

In a stack, each protocol layer relies on those below in order to provide
additional functionality.

There are two major models:

- The OSI (Open Systems Interconnection) reference model with 7 layers stacked
  on top of each other.
- The TCP/IP model, based on OSI, where layers 5 and 6 are merged into layer 7
  to form a single "Application" layer.

![stacks](/img/schc/stacks.png "SCHC Stack")

SCHC model activates the IP stack on constrained IoT networks by bringing an
adaptation layer between levels 2 and 3. Like a bridge over LPWAN and IP, it
ensures technology adaptation from level 1 through level 2, and application
adapation from level 2 up to level 5.

In the OSI and TCP/IP models, levels 1 to 3 are "media" layers and level 4 to 7
are "host" layers. SCHC adapatation process occurs at the level of the media
layers dealing with frames and packets.
