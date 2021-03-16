---
# vim: set ft=markdown ts=2 sw=2 tw=0 et :

title: KEMTLS
abbrev: KEMTLS
docname: draft-wiggers-tls-kemtls-latest
date:
category: info

ipr: trust200902
workgroup: tls
keyword: Internet-Draft

stand_alone: yes
pi: [toc, sortrefs, symrefs]

author: &authors
  - ins: T. Wiggers
    name: Thom Wiggers
    org: "Radboud University, Institute of Computing and Information Sciences, Nijmegen, The Netherlands"
    abbrev: "Radboud University"
    email: thom@thomwiggers.nl

normative:
  RFC8446:

informative:
  KEMTLS:
    title: "Post-Quantum TLS without Handshake Signatures"
    date: 2020-11
    author:
      - ins: D. Stebila
        name: Douglas Stebila
        org: University of Waterloo
      - ins: P. Schwabe
        name: Peter Schwabe
        org: "Radboud University and Max Planck Institute for Security and Privacy"
      - ins: T. Wiggers
        name: Thom Wiggers
        org: "Radboud University"
    seriesinfo:
      DOI: 10.1145/3372297.3423350
      "IACR ePrint": https://ia.cr/2020/534

--- abstract

TODO

--- middle

# Introduction

The primary goal of TLS 1.3 as defined in {{!RFC8446}} is to provide a
secure channel between two communicating peers; the only requirement
from the underlying transport is a reliable, in-order data stream.
Due to the advent of quantum computers, moving the TLS ecosystem
to post-quantum cryptography is a need. The protocol achieving should
goal is called KEMTLS. Specifically, this post-quantum secure channel
should provide the following properties:

-  Authentication: The server side of the channel is always
   authenticated; the client side is optionally authenticated.
   Authentication can happen via asymmetric cryptography by the
   usage of key-encapsulation mechanisms (KEM) by using the long-term
   KEM public keys in the Certificate.

-  Confidentiality: Data sent over the channel after establishment is
   only visible to the intended endpoints.  KEMTLS does not hide the
   length of the data it transmits, though endpoints are able to pad
   KEMTLS records in order to obscure lengths and improve protection
   against traffic analysis techniques.

-  Integrity: Data sent over the channel after establishment cannot
   be modified by attackers without detection.

KEMTLS consists of two primary components:

-  A handshake protocol that authenticates the communicating parties,
   negotiates cryptographic modes and parameters, and establishes
   shared keying material.  The handshake protocol is designed to resist
   tampering; an active attacker should not be able to force the peers
   to negotiate different parameters than they would if the connection
   were not under attack. This is model after TLS 1.3.

-  A record protocol that uses the parameters established
   by the handshake protocol to protect traffic between the
   communicating peers.  The record protocol divides traffic up into
   a series of records, each of which is independently protected
   using the traffic keys. This sub-protocol is the same as used in TLS
   1.3.

In this document, we will describe two modes in which KEMTLS can be used:
a full post-quantum mode where only post-quantum algorithms are used
and a hybrid post-quantum mode where traditional and post-quantum
algorithms are combined.

# Requirements Notation

{::boilerplate bcp14}

# Terminology

TODO: add definition of traditional algorithms, post-quantum algorithms
and KEM functionality.

# Protocol Overview

Figure 1 below shows the basic full KEMTLS handshake with both KEMTLS
modes:

       Client                                           Server

Key  ^ ClientHello
Exch | + kem_key_share*
     |  + key_share*
     | + kem_signature_algorithms*
     v + signatuere_algorithms*     -------->
                                                      ServerHello  ^ Key
                                                 + kem_key_share*  | Exch
                                                     + key_share*  v Exch
                                            {EncryptedExtensions}  ^  Server
                                            {CertificateRequest*}  v  Params
     ^                                             {Certificate*}  ^
Auth | {KEMCiphertext*}                                            |  Auth
     | {Certificate*}               -------->                      |
     |                              <--------    {KEMCiphertext*}  |
     | {Finished}                   -------->                      |
     | [Application Data*]          -------->                      |
     v                              <-------           {Finished}  |
                                                                   v
       [Application Data]           <------->  [Application Data]

              +  Indicates noteworthy extensions sent in the
                 previously noted message.

              *  Indicates optional or situation-dependent
                 messages/extensions that are not always sent.

              {} Indicates messages protected using keys
                 derived from a [sender]_handshake_traffic_secret.

              [] Indicates messages protected using keys
                 derived from [sender]_application_traffic_secret_N.

             Figure 1: Message Flow for Full KEMTLS Handshake

The handshake can be thought of as having three phases (indicated in
the diagram above):

-  Key Exchange: Establish shared keying material and select the
   cryptographic parameters.  Everything after this phase is
   encrypted and preserves post-quantum confidentiality.

-  Server Parameters: Establish other handshake parameters
   (whether the client is authenticated, application-layer protocol
   support, etc.).

-  Authentication: Authenticate the server (and, optionally, the
   client) and provide key confirmation and handshake integrity.
   This is post-quantum authentication.

In the Key Exchange phase, the client sends the ClientHello
message, which contains a random nonce (ClientHello.random); its
offered protocol versions (which should include support for KEMTLS);
a list of symmetric cipher/HKDF hash pairs; a set of KEM key
shares (in the "kem_key_share" extension), either a set of
pre-shared key labels (in the "pre_shared_key" extension), or both;
and potentially additional extensions. Additional fields and/or
messages may also be present for middlebox compatibility.

The server processes the ClientHello and determines the appropriate
cryptographic parameters for the connection.  It then responds with
its own ServerHello, which indicates the negotiated connection parameters.
The combination of the ClientHello and the ServerHello determines the shared
keys.  The ServerHello contains a "kem_key_share" extension with
the server's ephemeral KEM share; the server's share MUST
be in the same group as one of the client's shares.  If PSK key
establishment is in use, then the ServerHello contains a
"pre_shared_key" extension indicating which of the client's offered
PSKs was selected.  Note that implementations can use KEM and PSK
together, in which case both extensions will be supplied.

The server then sends two messages to establish the Server
Parameters:

EncryptedExtensions:  responses to ClientHello extensions that are
   not required to determine the cryptographic parameters, other than
   those that are specific to individual certificates.

CertificateRequest:  if certificate-based client authentication is
   desired, the desired parameters for that certificate.  This
   message is omitted if client authentication is not desired.

Finally, the client and server exchange Authentication messages.
KEMTLS uses the same set of messages every time that certificate-based
authentication is needed.  (PSK-based authentication happens as a
side effect of key exchange.)  Specifically:

Certificate:  The certificate of the endpoint and any per-certificate
   extensions.  This message is omitted by the server if not
   authenticating with a certificate and by the client if the server
   did not send CertificateRequest (thus indicating that the client
   should not authenticate with a certificate). The Certificate
   should include a long-term public KEM key.
   // TODO: can KEMTLS authenticate with anything else but a Cert?

KEMCiphertext:  A key encapsulation against the certificate's long-term
  public key, which yields an implicitly authenticated shared secret.
  This message is omitted by the Client if a CertificateRequest has
  not been sent.

Finished:  A MAC (Message Authentication Code) over the entire
   handshake.  This message provides key confirmation and binds the
   endpoint's identity to the exchanged keys.
   TODO: will there be a PSK mode?

Upon receiving the server's messages, the client responds with its
Authentication messages, namely Certificate and KEMCiphertext (if
requested), and Finished.

At this point, the handshake is complete, and the client and server
derive the keying material required by the record layer to exchange
application-layer data protected through authenticated encryption.

Application Data MUST NOT be sent prior to sending the Finished
message, except as specified in Section 2.3.  Note that while the
client may send Application Data prior to receiving the server's
last Authentication message, any data sent at that point is, of course,
being sent to an implicitly authenticated peer. It is worth noting
that Application Data sent prior to receiving the server's last
Authentication message can be subject to a client downgrade
attack. Full downgrade resilience is only achieved when explicit
authentication is achieved: when the Client receives the Finished
message from the Server.

## Resumption and Pre-Shared Key (PSK)

TODO: can it be used with KEMTLS?

## Prior-knowledge KEM-TLS

Given the added number of round-trips to the TLS 1.3, KEMTLS'
handshake can be improved by the usage of pre-distributed
KEM authentication keys to achieve explicit authentication
and full downgrade resilience. A party's long-term KEM
authentication key can be cached in advance.

# Handshake protocol

TODO: add KEMCiphertext to the `HandshakeType` and forbid
`CertificateVerify`.

# Key Exchange Messages

TODO: huge todo

# Key schedule

## Negotiation

1. Add KEMs to ``supported_groups`` and KEM public key to ``key_shares``.
1. Add ``KEMTLSwithSomeKEM`` to ``signature_algorithms``.
  1. Alternatively, use DC draft and add it to ``signature_algorithms_dc``.
1. Server replies with KEM ciphertext encapsulated to KEM public key (or HRR).
1. Server replies with CA-signed KEM public-key X.509 certificate.
  1. Alternatively, use DC draft and reply with KEM delegated credential.
1. Server MUST NOT submit ``CertificateVerify`` message.
1. Client detects that KEMTLS is being used via the "signature algorithm", ie. the type of credential provided by the server.
1. Client encapsulates ciphertext to server and transmits it as ``ClientKEMCiphertext`` (or piggy-back on ``ClientKeyExchange``?).
1. Client submits ``ClientFinished``.
1. Client completed handshake.
1. Server submits ``ServerFinished``.
1. Server completes handshake.


TODO probably split off DC into separate paragraph.

## Handshake Secret

The Handshake secret ``HS`` is derived from the ephemeral key exchange
shared secret.

    ...
                         v
    ephemeral_kem_shared -> HKDF-Extract = Handshake Secret
                         v
    ...

## Authenticated Handshake Secret

After deriving ``HS``, the Handshake Secret, from the ephemeral keys, we
need an additional handshake secret in KEMTLS.
We require this additional handshake secret as we need an (implicitly)
authenticated handshake secret to encrypt any client credentials under.
Otherwise we do not have confidentiality for the client certificate.

``AHS`` is derived from the hanshake secret and the static shared key.
We change the derivation of the main secret accordingly.

                      Handshake Secret
                         |
                         v
                        Derive-Secret(., "derived", "")
                         |
                         v
    static_server_shared -> HKDF.Extract = Authenticated Handshake Secret
                         |
                         +-----> Derive-Secret(., "c ahs traffic",
                         |                     ClientHello...ClientKemCiphertext)
                         |                     = client_authenticated_handshake_traffic_secret
                         |
                         +-----> Derive-Secret(., "s ahs traffic",
                         |                     ClientHello...ClientKemCiphertext)
                         |                     = server_authenticated_handshake_traffic_secret
                         |
                         v
                        Derive-Secret(., "derived", "")
                         |
                         v
    static_client_shared -> HKDF.Extract = Main Secret
                         v


``shared_secret_static_client_kem`` is the shared secret encapsulated to the client's public key.
If there is no client authentication, it's simply replaced by ``0``.

                         v
                       0 -> HKDF.Extract = Main Secret
                         v

## Finished keys

We derived the finished keys from ``MS`` instead of from ``CHTS`` and ``SHTS``.
We separate them by individual labels and the transcript.

    finished_client <- HKDF.Expand(MS, "c finished", transcript_up_to_CF)
    finished_server <- HKDF.Expand(MS, "s finished", transcript_up_to_SF)


# Protocol Mechanics

TODO

# (Middlebox) Compatibility Considerations

Like in TLS 1.3, after the ephemeral key is derived
a ``ChangeCipherSpec`` message is sent and the messages afterwards are
encrypted. This will make the following messages opaque to non-decrypting
middle boxes.

The ``ClientHello`` and ``ServerHello`` messages are still in the clear
and these require the addition of new ``key_share`` types.
Typical KEM public-key and ciphertext sizes are also significantly bigger
than pre-quantum (EC)DH keyshares. This may still cause problems.

# Integration with Delegated Credentials

# Security Considerations {#sec-considerations}

TODO

# IANA Considerations

* We need a new OID for each KEM to encode them in X.509 certificates.

# Acknowledgements

This work has been supported by the European Research Council through Starting Grant No. 805031 (EPOQUE).

--- back
