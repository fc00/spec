# The FC Switch

The Switch provides multihop message passing.
It receives packets from several Peering Connections,
and passes them on to the respective next hop,
as defined by the packet's Switch Label.

The Switch operates below the Routing layer and above the Peering layer.
The Switch is to FC, what Layer 2 is to an IP/Ethernet network.

## Overview

- Wire
- Switch label
  - Encoding schemes
    - Fixed-width vs. variable-width
  - Label operations
- Packets
- PeerLink
  - Congestion Control
  - Multipath Peering

The Router adds a Switch Label to the packet,
which is used by the Switch to pass the packet on
to the right connection maintained by the Peering Controller.

An fc00 node doesn't neccessarily run the layers above the Switch.
It can just as well merely provide transit to its peers.
(how to look past the switch?)

The Peering Controller opens, maintains, and closes Peering Connections.
Every connection is registered with the Switch, which assigns it a Director.
Multiple connections might be registered per peer,
and will result in separate Directors, e.g. for Wifi and cable.
In addition to outgoing and incoming connections,
a "self connection" is registered, which is not backed by a transport.
The self connection is used by the router to transmit and receive packets.

## Packet Layout

```
                     1               2               3
     0 1 2 3 4 5 6 7 0 1 2 3 4 5 6 7 0 1 2 3 4 5 6 7 0 1 2 3 4 5 6 7
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  0 |                                                               |
    +                         Switch Label                          +
  4 |                                                               |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  8 |   Congest   |S| V |labelShift |            Penalty            |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
 12 |                                                               |
    +                 Data Packet or Control Packet                 +
    |
```
