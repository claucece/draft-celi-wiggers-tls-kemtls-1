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

TODO

# Requirements Notation

{::boilerplate bcp14}

# Protocol Overview

TODO

# Negotiation

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

# Key schedule

## Handshake Secret

The Handshake secret ``HS`` is derived from the ephemeral key exchange shared secret.

    ...
                         v
    ephemeral_kem_shared -> HKDF-Extract = Handshake Secret
                         v
    ...

## Authenticated Handshake Secret

After deriving ``HS``, the Handshake Secret, from the ephemeral keys, we need an additional handshake secret in KEMTLS.
We require this additional handshake secret as we need an (implicitly) authenticated handshake secret to encrypt any client credentials under.
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

## Finished keys

We derived the finished keys from ``MS`` instead of from ``CHTS`` and ``SHTS``.
We separate them by individual labels and the transcript.

``finished_client <- HKDF.Expand(MS, "c finished", transcript_up_to_CF)``
``finished_server <- HKDF.Expand(MS, "s finished", transcript_up_to_SF)``

# Protocol Mechanics

TODO

# (Middlebox) Compatibility Considerations

Like in TLS 1.3, after the ephemeral key is derived a ``ChangeCipherSpec`` message is sent and the messages afterwards are encrypted.
This will make the following messages opaque to non-decrypting middleboxes.

The ``ClientHello`` and ``ServerHello`` messages are still in the clear and these require the addition of new ``key_share`` types.
Typical KEM public-key and ciphertext sizes are also significantly bigger than pre-quantum (EC)DH keyshares.
This may still cause problems.

# Security Considerations {#sec-considerations}

TODO

# IANA Considerations

* We need a new OID for each KEM to encode them in X.509 certificates.

# Acknowledgements

This work has been supported by the European Research Council through Starting Grant No. 805031 (EPOQUE).

--- back
