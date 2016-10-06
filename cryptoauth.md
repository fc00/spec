# CryptoAuth

**Draft 2**

Authors:

- Emery Hemingway (@ehmry)
- Lars Gierth (@lgierth)

CryptoAuth is a packet-oriented encryption and authentication protocol.
It is tolerant toward packet loss and out-of-order packets,
and also provides authorization using various challenges.

- [Introduction](#introduction)
- Protocol changes
- Dramatization
- Cryptographic primitives
- Implementations
- Handshake
  - Protocol
  - Packet layout
  - Hello, RepeatHello
  - Key, RepeatKey
- Data
  - Protocol
  - Packet layout
  - Session state rollover
- Authorization challenges
  - Type 0: none
  - Type 1: password
  - Type 2: login/password


## Introduction

CryptoAuth at its essence defines:

1. A handshake protocol for exchanging session secrets.
2. The application of well known cryptographic primitives to network data on layers 2 and 3.

CryptoAuth makes a few assumptions:

- The underlying network transport is unreliable
  and might lose any number of packets, or transmit packets in the wrong order.
- The remote participant's Permanent Public Key has been acquired out-of-band.

CryptoAuth's mode of operation is simple:
it takes as input individual network packets, encodes them, then outputs them.

1. Read one network packet and the respective remote public key.
2. If no matching session in `established` state:
  - start new session (`new` state) or reuse existing session (other states),
  - encode as Handshake Packet,
  - write packet,
3. With existing `established` session:
  - encode as Data Packet,
  - write packet,


## Dramatization

- **Alice:** TODO
- **Bob:** TODO


## Cryptographic primitives

To simplify the protocol and minimize packet size overhead,
CryptoAuth uses exactly one combination of key exchange, stream cipher, and HMAC.
With no negotiation needed, a whole class of vulnerabilities disappear.

- Key exchange: curve25519
- Stream cipher: salsa20
- HMAC: poly1305

When an implementation displays a private or public key
to a user, or to another program,
it should encode the key as Base32, as used by [dnscurve](dnscurve).

[dnscurve]: https://dnscurve.org/in-implement.html

- TODO: remote's public key (and auth challenge) needs to be grabbed out-of-band
- TODO: write a bit more about why these were chosen, their characteristics, etc.
- TODO: rollover of session state field


## Implementations

- TODO: should provide a sessionmanager
- TODO: should wrap packet-oriented io, e.g. golang's net.PacketConn


## Handshake

~~Before two nodes can exchange encrypted messages, they must perform a handshake,
by agreeing on a temporary shared secret and establishing a CryptoAuth session.
Encrypted data may be piggy-backed on handshake packets,
although these don't offer Perfect Forward Secrecy.~~

~~The node which sends the first packet is the initiating party.~~

~~A handshake packet or data packet MUST start with a session state field,
which determines the state of the handshake, or the recipient's identifier for the session.~~

### Protocol

The CryptoAuth handshake is the method by which two nodes perform mutual
authentication and ephemeral key generation. A CryptoAuth header signifies
a handshake whenever the Session State field contains one of the following
values.

- 0x00000000 - Hello
- 0x00000001 - RepeatHello
- 0x00000002 - Key
- 0x00000003 - RepeatKey

TODO: should it be "RepeatedHello" instead of "RepeatHello"?

- HELLO send
  - nonce: random 24 bytes
  - sharedSecret: scalarmult_curve25519(senderPrivKey, recvrPubKey) + sha256(authchallenge)
  - tempPrivKey: random 32 bytes
  - crypto_box_curve25519xsalsa20xpoly1305(tempPubKey, sharedSecret, nonce)
- HELLO receive
  - sharedSecret: scalarmult_curve25519(recvrPrivKey, senderPubKey) + sha256(authchallenge)
  -

### Packet layout

```
                     1               2               3
     0 1 2 3 4 5 6 7 0 1 2 3 4 5 6 7 0 1 2 3 4 5 6 7 0 1 2 3 4 5 6 7
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  0 |                    Session State (int32 < 4)                  |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  4 |                                                               |
  8 |                         Auth Challenge                        |
 12 |                                                               |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
 16 |                                                               |
 20 |                                                               |
 24 |                         Random Nonce                          |
 28 |                                                               |
 32 |                                                               |
 36 |                                                               |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
 40 |                                                               |
 44 |                                                               |
 48 |                                                               |
 52 |                 Sender's Permanent Public Key                 |
 56 |                                                               |
 60 |                                                               |
 64 |                                                               |
 68 |                                                               |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
 72 |                                                               |
 76 |                  Message Authentication Code                  |
 80 |                                                               |
 84 |                                                               |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
 88 |                                                               |
 92 |                                                               |
 96 |                                                               |
100 |            Sender's Encrypted Temporary Public Key            |
104 |                                                               |
108 |                                                               |
112 |                                                               |
116 |                                                               |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
120 |                Variable-length encrypted data                 |
... |
```

- Session state: communicates the local authentication state, big endian.
- Auth challenge: see section about Authorization Challenges.
- Random nonce: nonce used during encryption of handshake public key.
- Sender's permanent public key: identifies sending node and is used to derive fc00::/8 address.
- Message authentication code: MAC for key payload - Poly1305 Authenticator.
- Sender's encrypted temporary public key: Handshake public keys exchanged between nodes to generate session symmetric key.
- Variable-length encrypted data: data being piggy-backed on handshake packet.

### Hello, RepeatHello

A random 24 byte number is placed in the **random nonce** field, to be
used during encryption and decryption of the payload.

The public key of the node is written to **Permanent public key**.
This public key is the same for every session at this node, and is known
to the local switch and router, as well as the remote node.

Next a temporary key is generated to protect the handshake payload. The
**scalarmult_curve25519** primative is applied to the permanent secret key
of the local node and the permanent public key of the remote node. The
product of the combined keys is concatenated with the SHA256 digest of the
authentication password, and the SHA256 digest of that concatenation is the
temporary key. ONLY FOR AUTHTYPE 1 & 2, FOR 0 THERE IS NO COMBINING

The key to be encrypted by the aformentioned temporary key is the public
key of a randomly generated assymetric keypair local to this CryptoAuth
handshake.

This key and any trailing packet content are passed through the
**crypto_box_curve25519xsalsa20poly1305** primative using the previously
generated random nonce and temporary key. This generates both the payload
MAC and payload ciphertext.

*As an implentation detail, the input to
crypto_box_curve25519xsalsa20poly1305 must be preceded by an additional
thirty two zero bytes, and the output is preceded by an extra sixteen bytes.
Following the first sixteen extra bytes of the output is the sixteen byte
MAC, so the length of the input is the same as the length of the output.*

The HELLO packet is transmited to the remote node.

On the reception of a HELLO packet the packet is checked for proper length
and the **Permanent public key** field is stored for future use.

The **Hash code** subfield of **Authentication challenge** is validated
against local passwords using the same method of hash code generatation
described above. OR USERNAME IF IT IS AUTHTYPE 2

To decrypt the handshake key payload, the HELLO recipient generates the
>>temporary key<< DONT USE TEMPORARY KEY, CALL IT TEMPORARY SHARED SECRET (TEMP curve25519 KEYS ARE ALSO USED) used by the remote node during encryption using the same
process. The local permanent secret key is combined with the remote
permanent public key using the **scalarmult_curve25519** primative,
concatenated with the SHA256 digest of the password (UNLESS AUTHTYPE 0), and hashed again with
SHA256 to create the temporary key.

The payload is authenticated and decrypted, and the recipient stores the
remote handshake key for a KEY response packet.

### Key, RepeatKey

The transmitting node zeros out the **Authentication challenge**,
generates the **Random Nonce**, and copies the local permanent public key
to **Permanent Public Key**.

The node generates an asymetric handshake key pair just as the remote node
did during the HELLO state, and this key is encrypted using a temporary key.

This temporary key is generated by applying the
**curve25519xsalsa20poly1305_beforenm** primative to the public key
contained in the preceding HELLO packet and the local permanent secret key.

The temporary key encrypts and authenticates the handshake public key as
described above and the KEY packet is transmitted.

The node receiving the KEY packet checks the packet length and
**Permanent Public Key**. The node recreates the temporary key used to
encrypt the KEY packet by applying
**crypto_box_curve25519xsalsa20poly1305_beforenm** to the remote permanent
public key and the local handshake secret key generated during the HELLO
state. This temporary key decrypts the KEY payload, revealing the remote
handshake public key. This key is then combined with the local handshake secret
key using **crypto_box_curve25519xsalsa20poly1305_beforenm** to generate the final
symmetric key used to encrypt the remainder of the CyptoAuth session.

When the node that received the inital HELLO packet receives the first data
packet, it can assume that the handshake has completed at the remote end,
and performs aformentioned operation to generate the symmetric session key,
combining the remote public handshake key with the local secret handshake
key. The handshake is now complete.


## Data

### Protocol

Data packets start with the same **Session state** field as the handshake
header. This field is used as a nonce for authentication and encryption.
Those primatives require a twenty-four byte nonce, so for the node that
initiates the session, the state is converted to little endian and copied
to the first four bytes bytes of the nonce, for the other node, the state
is copied little endian to the second four bytes of the nonce. When either
node has sent (2^32)-4 data packets, the nounce is exhausted and a new
session must be negotiated.

Packet payload is decrypted by passing the session symmetric key, the
previously mentioned nonce, and the remainder of the packet starting with
the MAC to the **crypto_box_curve25519xsalsa20poly1305_open_afternm**
primative. After a packet is decrypted the payload is forwarded to the
switch.

### Packet layout

```
                     1               2               3
     0 1 2 3 4 5 6 7 0 1 2 3 4 5 6 7 0 1 2 3 4 5 6 7 0 1 2 3 4 5 6 7
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  0 |                   Session State (int32 >= 4)                  |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  4 |                                                               |
  8 |                  Message authentication code                  |
 16 |                                                               |
 20 |                                                               |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
 24 |                Variable-length encrypted data                 |
... |
```

- Session state: an incrementing session counter in big endian.
- Message authentication code: authenticates packet payload.
- Variable-length encrypted data: zero or more bytes of data.

### Session state rollover

TODO: specify this


## Authorization challenges

The node initiating the session first generates the authentication
challenge field from the peering password previously transmitted
out-of-band.

Layout of this field:
```
                  1               2               3
  0 1 2 3 4 5 6 7 0 1 2 3 4 5 6 7 0 1 2 3 4 5 6 7 0 1 2 3 4 5 6 7
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
0 |   Auth Type   |                                               |
  +-+-+-+-+-+-+-+-+           Hash Code                           +
4 |                                                               |
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
8 |                           (Unused)                            |
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

**Auth Type** indicates the how **Hash Code** is generated, and is currently
only set to 0x01 to indicate a password digest.

0x02 -> hash code is digest of username and password digest is used in session setup

**Hash Code** is the second through eighth byte of the SHA256 digest of
the peering password.

**Derivations** is used when a second node a shares a credential with
a third node by creating a derived credential from the password for the
first node. This is not documented here and in the common case this field
and the remaining bytes of the header are set to zero.

### Type 0: none

TODO: specify this

### Type 1: password

TODO: specify this

### Type 2: login/password

TODO: specify this


## Replay / out-of-order protection

TODO
