---
title: "Multiplexing RTP and Data Channels over QUIC"
docname: draft-engelbart-multiplex-roq-qdc-latest
category: std
date: {DATE}

ipr: trust200902
area: "Applications and Real-Time"
workgroup: ""
keyword: Internet-Draft

stand_alone: yes
pi: [toc, sortrefs, symrefs]
submissiontype: IETF

author:
 -
    ins: M. Engelbart
    name: Mathis Engelbart
    organization: Technical University Munich
    email: mathis.engelbart@gmail.com
 -
    ins: J. Ott
    name: Jörg Ott
    organization: Technical University Munich
    email: ott@in.tum.de

informative:

--- abstract

This document defines a multiplexing scheme for using RTP over QUIC and QUIC
Data Channels in a single QUIC connection.

--- middle

# Introduction

One of the requirements in the development of RTP over QUIC (RoQ)
({{!I-D.draft-ietf-avtcore-rtp-over-quic}}) was the ability to share a QUIC
connection for RTP, RTCP, and other protocols. RoQ uses a flow identifier to
multiplex different RTP streams, RTCP, and possibly other protocols. However,
RoQ does not describe which protocol can be multiplexed with RoQ on the same
QUIC connection. {{!I-D.draft-engelbart-quic-data-channels}} describes a simple
protocol similar to WebRTC data channels ({{?RFC8831}}) called QUIC Data
Channels (QDC). QDC uses QUIC instead of SCTP over DTLS and also uses an
identifier similar to the one used by RoQ. This document describes how the two
protocols can be multiplexed in one QUIC connection.

# Connection Establishment

QUIC requires the use of ALPN {{!RFC7301}} during connection setup. Connections
multiplexing RoQ and QDC use the "roq-qdc-mux" token during the TLS handshake.

## Draft version identification

> **RFC Editor's note:** Please remove this section prior to publication of a
> final version of this document.

Only implementations of the final, published RFC can identify themselves as
"roq-qdc-mux". Until such an RFC exists, implementations MUST NOT identify
themselves using this string.

Implementations of draft versions of the protocol MUST add the string "-" and
the corresponding draft number to the identifier. For example,
draft-engelbart-multiplex-roq-qdc-04 is identified using the string
"roq-qdc-mux-04".

Non-compatible experiments based on these draft versions MUST append the string
"-" and an experiment name to the identifier.

# Multiplexing

The two protocols for which this document describes a multiplexing scheme use a
variable length integer as the first element on every QUIC stream and datagram.
In RoQ, this integer is called a flow identifier, and it is used to multiplex
different RTP streams and RTCP. In QDC, the integer is called a channel
identifier and is used to multiplex multiple data channels.

This document describes two options for multiplexing RoQ and QDC. The first
option builds upon the external signaling that is required by RoQ
({{external-signaling}}). The second one works without any external signaling
but places some restrictions on the chosen flow and channel identifiers
({{no-signaling}}).

> **TODO:** If we define two methods, we also need to say how to choose between
> them. This could involve more external signaling, e.g., SDP, or we could use
> two different ALPNs. It might be easier to just settle on one single way to do
> it.

## Multiplexing using External Signaling {#external-signaling}

RoQ requires using an external signaling protocol like the one described in
{{?I-D.draft-dawkins-avtcore-sdp-rtp-quic}}. The signaling includes the
negotiation of flow identifiers for RoQ. In this scheme, applications MUST agree
on all identifiers used by RoQ using the external signaling. Any identifier not
assigned to a RoQ flow using external signaling MUST be treated as a QDC channel
ID. Endpoints MUST NOT use an identifier for RoQ if it has not been signaled and
agreed upon by both peers using the external signaling protocol.

## Multiplexing without Signaling {#no-signaling}

When there is no external signaling for negotiation of identifiers available,
endpoints can use the second least significant bit (0x02) of the flow or channel
identifiers for multiplexing. RoQ flows have the bit set to 1, and QDC channels
have the bit set to 0.

The second least significant bit is used instead of the least significant bit
because the least significant bit already has a special function in QDC.

# Error Handling

> **TODO:** This looks tricky. Some error codes will be easy to demultiplex when
> they are associated with a flow/channel ID. But some might not be, e.g., in
> CONNECTION_CLOSE frames.

# Security Considerations {#sec-considerations}

The security considerations for the QUIC protocol and datagram extension
described in {{Section 21 of !RFC9000}}, {{Section 9 of !RFC9001}}, {{Section 8
of !RFC9002}} and {{Section 6 of !RFC9221}} apply.

# IANA Considerations {#iana-considerations}

## Registration of an ALPN Identification String

This document creates a new registration for the identification of the protocol
in the "TLS Application-Layer Protocol Negotiation (ALPN) Protocol IDs" registry
{{?RFC7301}}.

The "roq-qdc-mux" string identifies RoQ:

  Protocol:
  : Multiplexed RoQ and QDC

  Identification Sequence:
  : TODO

  Specification:
  : This document

--- back

# Acknowledgments
{:numbered="false"}

