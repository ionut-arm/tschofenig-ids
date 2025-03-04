---
title: The Use of Attestation in OAuth 2.0 Dynamic Client Registration
abbrev: Attestation in Dynamic Client Registration
docname: draft-tschofenig-oauth-attested-dclient-reg-00
category: std

ipr: trust200902
area: Security
workgroup: OAUTH
keyword: Internet-Draft

stand_alone: yes
pi:
  rfcedstyle: yes
  toc: yes
  tocindent: yes
  sortrefs: yes
  symrefs: yes
  strict: yes
  comments: yes
  inline: yes
  text-list-symbols: -o*+
  docmapping: yes
  toc_levels: 4

author:
 -
       ins: H. Tschofenig
       name: Hannes Tschofenig
       organization: Siemens
       email: hannes.tschofenig@gmx.net

 -
       ins: J. Herrmann
       name: Jan Herrmann
       organization: Siemens
       email: jan.herrmann@siemens.com


normative:
  RFC2119:
  RFC8174:
  RFC7591:

informative:
  RFC9334:
  I-D.ftbs-rats-msg-wrap:
  TPM20:
     author:
        org: Trusted Computing Group
     title: Trusted Platform Module Library Specification, Family 2.0, Level 00, Revision 01.59
     target: https://trustedcomputinggroup.org/resource/tpm-library-specification/
     date: November 2019

--- abstract

The OAuth 2.0 Dynamic Client Registration specification described in RFC 7591
describes how an OAuth 2.0 client can be dynamically registered with an authorization
server to obtain information to interact with this authorization server, including an
OAuth 2.0 client identifier.

To offer proper security protection for this dynamic client registration some security
credentials need to be available on the OAuth 2.0 client. For this purpose RFC 7591
relies on two mechanisms, a trust-on-first-use model and a model where the client is
in possession of a software statement (a sort-of bearer token).

This specification improves the security of the OAuth 2.0 Dynamic Client Registration
specification by introducing the support of attestation.

--- middle

# Introduction

The OAuth 2.0 Dynamic Client Registration specification described in RFC 7591
describes how an OAuth 2.0 client can be dynamically registered with an authorization
server to obtain information to interact with this authorization server, including an
OAuth 2.0 client identifier. As part of the registration process, this specification
also defines a mechanism for the client to present the authorization server with a
set of metadata, such as a set of valid redirection URIs.

To offer proper security protection for this dynamic client registration some security
credentials need to be available on the OAuth 2.0 client. For this purpose RFC 7591
relies on two mechanisms, a trust-on-first-use model and a model where the client is
in possession of a software statement (a sort-of bearer token).

This specification improves the security of the OAuth 2.0 Dynamic Client Registration
specification by introducing the support of attestation.

{{fig-arch}} shows the high-level communication pattern of the IETF RATS passport
model where the attester transmits the evidence in the OAuth 2.0 Dynamic
Client Registration to the authorization server. The authorization server thereby
acts as a relying party and relays the evidence to the verifier. The verifier
processes the received evidence and computes an attestation result, which is then
processed by the authorization server. Note that the verifier is a logical role that may
be included in an authorization server product. In this case the interaction between
the relying party and the verifier is local.

~~~
                              .-------------.
                              |             | Compare Evidence
                              |   Verifier  | against
                              |             | policy
                              '--------+----'
                                   ^   |
                          Evidence |   | Attestation
                                   |   | Result
                                   |   v
 .------------.               .----|----------.
 |            +-------------->|---'           | Compare Attestation
 | OAuth 2.0  |   Evidence    | Relying       | Result against
 |   Client   |  in Dynamic   | Party (AS)    | policy
 '------------' Client Reg.   '---------------'
~~~
{: #fig-arch title="Generic Attestation Architecture Applied to OAuth 2.0"}

As a design goal, the proposed extension is agnostic to the attestation
technology being used. No new attestation technology (and evidence
format in particular) is defined by this specification. Instead, this
document focuses on conveying evidence to the authorization server
during the process of dynamically registering a client.

To prevent the replay of evidence, attestation technologies require some
replay protection to be used. Nonce-based freshness is one
approach supported by various attestation technologies. TPM v2.0 {{TPM20}},
for example, supports nonce-based replay protection. Hence, it is necessary
to convey a nonce from the authorization server (which typically obtains
it from the verifier) to the OAuth 2.0 Client.

In {{extension}} we describe the protocol mechanism 

# Conventions and Definitions

{::boilerplate bcp14-tagged}

This document re-uses the terms defined in RFC 9334 related to remote
attestation. Readers of this document are assumed to be familiar with
the following terms: evidence, claim, attestation result, attester,
verifier, and relying party.

# Extension {#extension}

This section defines the extensions needed to support the use of
attestation.

The abstract OAuth 2.0 client dynamic registration flow illustrated
in {{fig-arch2}} describes the interaction between the client or developer
and the endpoint defined in {{RFC7591}}.  This figure does not
demonstrate error conditions.  

~~~
        +--------(A)- Attestation Evidence (OPTIONAL)
        |
        |   +----(B)- Initial Access Token (OPTIONAL),
        |   |         Software Statement (OPTIONAL)
        v   v
    +-----------+                                      +---------------+
    |           |--(C)- Client Registration Request -->|    Client     |
    | Client or |                                      | Registration  |
    | Developer |<-(D)- Client Information Response ---|   Endpoint    |
    |           |        or Client Error Response      +---------------+
    +-----------+
~~~
{: #fig-arch2 title="Abstract Dynamic Client Registration Flow"}

This flow includes the following steps:

(A)   Optionally, but defined in this specification, the client is
     obtains evidence from an attester. The attester is typically a
     component on the device that consists of hardware and low-level
     software. The methodused by the OAuth 2.0 software to obtain evidence is
     specific to a given attestation technology and outside the
     scope for this specification.

(B)   Optionally, the client or developer is issued an initial access
     token or a software statement giving access to the client
     registration endpoint. The method by which the initial access
     token or the software statement are issued to the
     client or developer is out of scope for this specification.

(C)   The client or developer calls the client registration endpoint
     with the client's desired registration metadata, optionally
     including information from (A) or (B) if one is required
     by the authorization server.

(D)   The authorization server registers the client and returns:

 - the client's registered metadata,

 - a client identifier that is unique at the server, and

 - a set of client credentials such as a client secret, if
  applicable for this client.

This specification re-uses the client registration endpoint, as
described in Section 3.1 of {{RFC7591}}.

Upon a successful registration request, the authorization server
returns a client identifier for the client.  The server responds with
an HTTP 201 Created status code and a body of type "application/json"
with content as described in Section 3.2.1 of {{RFC7591}}.

If evidence was used as part of the registration, its
value MUST NOT be returned in the response along with other
metadata to the OAuth 2.0 client. Evidence is only communicated
from the OAuth 2.0 client to the authorization server - not vice versa.
   
Upon an unsuccessful registration request, the authorization server
responds with an error, as described in Section 3.2.2 of {{RFC7591}}.

This specification defines a new error message that is used when the
authorization server challenges the OAuth 2.0 client for evidence
using a fresh nonce. The nonce MUST be randomly generated with a
length of 32 bytes or more (before base64 encoding the value).
The error is defined as:

~~~
stale_evidence
  The provided evidence is not current. Resend fresh evidence.
~~~

A new member is used in the response JSON object called "nonce".
It contains the nonce value that will be used by the OAuth 2.0
client as input to the API call for requesting fresh evidence
from the attester. Typically, the nonce value will be included
(directly or indirectly) in the returned evidence. The verifier
who provided the nonce to the authorization server MUST store
the provided nonce for later comparison against the nonce
included in the evidence.

An OAuth 2.0 client that is configured to use attestation
information with dynamic client registration MAY send a
minimal registration request in the anticipation that an error
response will follow. This error response will contain a nonce,
which is then used to obtain fresh evidence from the attester.
At a minimum the OAuth 2.0 client MUST convey the "client_name"
but MAY also include any (likely stale) evidence available since
it might provide the authorization server with information
to distinguish clients with different capabilities, some of which
may not offer any attestation capabilities.

Once an error response with a nonce is available the OAuth 2.0
client MUST include the obtained evidence in the newly constructed
full request.

The following is a non-normative example of an error response
containing a nonce (with line breaks within values for display
purposes only):

~~~
 HTTP/1.1 400 Bad Request
 Content-Type: application/json
 Cache-Control: no-store
 Pragma: no-cache

 {
  "error": "stale_evidence",
  "error_description": "The provided evidence is not current.",
  "nonce": "lBjvTtuPbpzIaqyiAOOkrIol3WmflPUUepzUXNDFuUgMKUL"
 }
~~~

To include evidence in a registration request the following OPTIONAL
member is included in the JSON object:

~~~
   evidence
      Evidence is a set of claims generated by an attester to
      be appraised by a verifier. Evidence may include configuration
      data, measurements, telemetry, or inferences. This is a string
      value containing the evidence, as produced by the selected
      attestation technology.
~~~

[Editor's Note: The evidence structure may utilize the format defined
in the RATS Conceptual Messages Wrapper specification
{{I-D.ftbs-rats-msg-wrap}}, which allows to indicate type information
as well.]

In the following example, a request is sent containing attestation
evidence and registering a JWK Set by value (with line breaks
within values for display purposes only). Some registration
parameters are conveyed in the evidence while some values specific
to the client instance are conveyed as regular parameters.

~~~
 POST /register HTTP/1.1
 Content-Type: application/json
 Accept: application/json
 Host: server.example.com

 {
   "redirect_uris": [
    "https://client.example.org/callback",
    "https://client.example.org/callback2"
   ],
   "client_name": "My Example Client",
   "jwks": {"keys": [{
       "e": "AQAB",
       "n": "nj3YJwsLUFl9BmpAbkOswCNVx17Eh9wMO-_AReZwBqfaWFcfG
   HrZXsIV2VMCNVNU8Tpb4obUaSXcRcQ-VMsfQPJm9IzgtRdAY8NN8Xb7PEcYyk
   lBjvTtuPbpzIaqyiUepzUXNDFuAOOkrIol3WmflPUUgMKULBN0EUd1fpOD70p
   RM0rlp_gg_WNUKoW1V-3keYUJoXH9NztEDm_D2MQXj9eGOJJ8yPgGL8PAZMLe
   2R7jb9TxOCPDED7tY_TU4nFPlxptw59A42mldEmViXsKQt60s1SLboazxFKve
   qXC_jpLUt22OC6GUG63p-REw-ZOr3r845z50wMuzifQrMI9bQ",
       "kty": "RSA"
     }]},
   "evidence": "eyJhbGciOiJSUzI1NiJ9eyJzb2Z0d2FyZV9pZCI6IjRO
    UkIxLTBYWkFCWkk5RTYtNVNNM1IiLCJjbGllbnRfbmFtZSI6IkV4YW1wbGUg
    U3RhdGVtZW50LWJhc2VkIENsaWVudCIsImNsaWVudF91cmkiOiJodHRwczov
    GHfL4QNIrQwL18BSRdE595T9jbzqa06R9BT8w409x9oIcKaZo_mt15riEXHa
    zdISUvDIZhtiyNrSHQ8K4TvqWxH6uJgcmoodZdPwmWRIEYbQDLqPNxREtYn0
    5X3AR7ia4FRjQ2ojZjk5fJqJdQ-JcfxyhK-P8BAWBd6I2LLA77IG32xtbhxY
    fHX7VhuU5ProJO8uvu3Ayv4XRhLZJY4yKfmyjiiKiPNe-Ia4SMy_d_QSWxsk
    U5XIQl5Sa2YRPMbDRXttm2TfnZM1xx70DoYi8g6czz-CPGRi4SW_S2RKHIJf
    IjoI3zTJ0Y2oe0_EJAiXbL6OyF9S5tKxDXV8JIndSA",
   "scope": "read write"
}
~~~

# Security Considerations {#sec-cons}

It is up to the verifier and to the authorization server to place
as much or as little trust in the evidence provided by the attester
on the OAuth 2.0 client information as dictated by policies.

This document defines the transport of evidence of different formats
in the OAuth 2.0 Dynamic Client Registration protocol. Some of the
attestation formats are based on standards while others are
proprietary formats. A verifier will need to understand these
formats for matching the received values against policies.

Policies drive the processing of evidence at the verifier and other
policies influence the decision making at the relying party when
evaluating the attestation result. The relying party is ultimately
responsible for making a decision of what attestation-related
information it will accept. The presence of the evidence defined
in this specification provide the authorizations server with additional
assurance about attester (and indirectly about the OAuth 2.0 client).
Policies used at the verifier and the authorization are implementation 
and configuration dependent and out of scope for this document.
Whether to require the use of evidence in OAuth 2.0 Dynamic Client
Registration is out-of-scope for this document.

Evidence generated by the attestation generally needs to be fresh to provide
value to the verifier since the configuration on the device may change
over time. Section 10 of {{RFC9334}} discusses different approaches for
providing freshness, including a nonce-based approach, the use of timestamps
and an epoch-based technique.  The use of nonces requires an extra message
exchange and the use of timestamps requires synchronized clocks.
Epochs also require communication. This document offers a way to
convey a nonce to the tester to allow evidence to be freshly generated.

Different attestation technologies provide different security and privacy
properties. Some evidence formats convey more information about the target
environment while others offer less. Some evidence formats offer the
security capabilities equivalent to a bearer tokens while others are
semantically closer to proof-of-possession tokens. Implementers and
operators need to make a conscious decision about the attestation
technologies they want to support in their products.

#  IANA Considerations

TBD.

--- back

# Acknowledgements

Add your name here.
