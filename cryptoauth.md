# CryptoAuth

**Draft 2**

Authors:

- Emery Hemingway (@ehmry)
- Lars Gierth (@lgierth)


#### *TODO*
- *describe the primatives*
- *ConnectToMe* <-- nolonger exists

This document describes the CryptoAuth layer of the FC stack that provides
confidentiality and authentication between network nodes, known as CryptoAuth.
It contains a description of CryptoAuth headers and the session handshake protocol.

- Introduction
- Handshake
- Header layout
- Header fields
- Handshake states
  - Hello
  - Key
- Data

- Introduction
- State flow
- Handshake
-

## Introduction

The confidentiality and authentication provided by the FC stack originate from
CryptoAuth layer. CryptoAuth at its essence defines the application of well known
crypographic primatives to network data above the network transport layer and defines
a handshake protocol. Sessions are authenticated with permanent asymmetric keys and
session confidentiality is provided by an ephemeral symmetric key crafted during the
session handshake.


## Handshake

The CryptoAuth handshake is the method by which two nodes perform mutual
authentication and ephemeral key generation. A CrypoAuth packet contains a
handshake header whenever the state field contains the following values:

0. HELLO
1. HELLO_REPEAT
2. KEY
3. KEY_REPEAT


### Header layout

Every handshake header contains the following layout:

```
                     1               2               3
     0 1 2 3 4 5 6 7 0 1 2 3 4 5 6 7 0 1 2 3 4 5 6 7 0 1 2 3 4 5 6 7
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  0 |                         Session State                         |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  4 |                                                               |
    +                                                               +
  8 |                         Auth Challenge                        |
    +                                                               +
 12 |                                                               |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
 16 |                                                               |
    +                                                               +
 24 |                                                               |
    +                         Random Nonce                          +
 28 |                                                               |
    +                                                               +
 32 |                                                               |
    +                                                               +
 36 |                                                               |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
 40 |                                                               |
    +                                                               +
 44 |                                                               |
    +                                                               +
 48 |                                                               |
    +                                                               +
 52 |                                                               |
    +                   Permanent Node Public Key                   +
 56 |                                                               |
    +                                                               +
 60 |                                                               |
    +                                                               +
 64 |                                                               |
    +                                                               +
 68 |                                                               |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
 72 |                                                               |
    +                                                               +
 76 |                                                               |
    +                  Message Authentication Code                  +
 80 |                                                               |
    +                                                               +
 84 |                                                               |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
 88 |                                                               |
    +                                                               +
 92 |                                                               |
    +                                                               +
 96 |                                                               |
    +                                                               +
100 |                                                               |
    +                Encrypted Handshake Public Key                 +
104 |                                                               |
    +                                                               +
108 |                                                               |
    +                                                               +
112 |                                                               |
    +                                                               +
116 |                                                               |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```


### Header fields

The handshake header contains the following fields:
- Session state
 - Communicates the local authentication state, big endian.
- Auth challenge
 - Authentication code generated from peering password.
- Random nonce
 - Nonce used during encryption of handshake public key.
- Permanent node public key
 - Permanent node key, identifies node and is used to derive FC00::/8 address.
- Message authentication code
 - MAC for key payload - Poly1305 Authenticator.
- Encrypted handshake public key
 - Handshake public keys exchanged between nodes to generate session symmetric key.


### Handshake states

#### Hello

bzzzzzt this is the authenticator and the hello packet relates to the entire packet header

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
8 | |        Derivations          |           (Unused)            |
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

--- this is the end of the part about the authenticator

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


#### Key

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

Packets following the handshake have the following header:

```
                     1               2               3
     0 1 2 3 4 5 6 7 0 1 2 3 4 5 6 7 0 1 2 3 4 5 6 7 0 1 2 3 4 5 6 7
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  0 |                         Session State                         |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  4 |                                                               |
    +                                                               +
  8 |                                                               |
    +                  Message authentication code                  +
 16 |                                                               |
    +                                                               +
 20 |                                                               |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
 24 |                                                               |
    +                                                               +
 28 |                         Encrypted data                        |
    +                                                               +
 32 |                                                               |
    +~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~+


```

Data headers contain two fields:
- Session state
 - An incrementing session counter in big endian.
- Message authentication code
 - Authenticates packet payload.
- Encrypted data
 - Zero or more bytes of data.

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
