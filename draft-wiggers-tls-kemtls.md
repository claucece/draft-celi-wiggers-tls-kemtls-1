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

# Key schedule


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

TODO

# Acknowledgements

This work has been supported by the European Research Council through Starting Grant No. 805031 (EPOQUE).

--- back
