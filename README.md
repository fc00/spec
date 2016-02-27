# fc00 specs

This repository hosts a **work-in-progress** specification
of the Futuristic Connectivity networking stack.

It's being derived from Caleb James DeLisle's [cjdns][cjdns].
Futuristic Connectivity a.k.a. FC a.k.a. fc00
is named after the address space used by its IPv6 network: `fc00::/8`

- [Introduction](intro.md)
- Peering
  - [CryptoAuth](cryptoauth.md)
  - [UDP Transport](peering/udp.md)
  - [Ethernet Transport](peering/ethernet.md)
- [Switch](switch.md)
  - [Label Encoding](switch/label_encoding.md)
  - [Label Operations](switch/label_operations.md)
  - Congestion Control **TODO**
- Routing
  - [Pathfinder](routing/pathfinder.md)
- Services
  - [IPv6](services/ipv6.md)
  - [IPTunnel](protocols/iptunnel.md)

Find us in #fc00 on freenode IRC.

We invite anyone to take part in the discussion and editing of these specs.
Please take note of the behaviour we practice and expect: [conduct][code of conduct]

[cjdns]: https://github.com/cjdelisle/cjdns
[conduct]: https://www.djangoproject.com/conduct/

```
+-----------------------------------------------------+
|                                                     |
|  +--------------------------------------------------+
|  |
|  | +-------------------------------------------+
|  | | Switch                                    |
|  | |                   +---+ +---+ +---+ +---+ | +-----------+
|  | |                   | 2 | | 3 | | 4 | | 1 <---+ Self Peer |
|  | |                   +-^-+ +-^-+ +-^-+ +---+ | +-----------+
|  | |                     |     |     |         |
|  | +-------------------------------------------+
|  |                       |     |     |
|  |                       |     |     |
|  | +-------------------------------------+
|  | | Peering Controller  |     |     |   |
|  | |                   +-+-+ +-+-+ +-+-+ |
|  | |                   | U | | U | | . | |
|  | |                   | D | | D | | . | |
|  | |                   | P | | P | | . | |
|  | |                   +---+ +---+ +---+ |
|  | +-------------------------------------+
|  |
|  |
|  +--------------------------------------------------+
|                                                     |
+-----------------------------------------------------+
```
