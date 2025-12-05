---
title: "An AuthZEN profile for trust registries"
abbrev: "AuthZEN Trust"
category: info

docname: draft-johansson-authzen-trust-latest
submissiontype: IETF
number:
date:
consensus: true
v: 3
area: SEC
#workgroup: OAuth Working Group
keyword:
 - authzen
 - trust registries
venue:
#  group: WG
#  type: Working Group
#  mail: WG@example.com
#  arch: https://example.com/WG
  github: leifj/draft-johansson-authzen-trust
  latest: https://leifj.github.io/draft-johansson-authzen-trust

author:
 -
    fullname: Leif Johansson
    organization: SIROS Foundation
    email: leifj@siros.org

normative:
  AUTHZEN:
    target: https://openid.github.io/authzen/
    title: OpenID AuthZEN Authorization API
    date: July 2024
    author:
    -
      name: Omri Gazitt
      ins: O. Gazitt
      org: Aserto
    -
      name: David Brossard
      ins: D. Brossard
      org: Axiomatics
    -
      name: Atul Tulshibagwale
      ins: A. Tulshibagwale
      org: SGNL
  RFC7517:
  RFC6749:
  RFC8259:
informative:
  DID:
    title: Decentralized Identifiers (DIDs) v1.0
    target: https://www.w3.org/TR/did-1.0/
    date: 2022
  RFC5280:
  RFC9525:
  NIST.SP.800-162:
  XACML:
    title: eXtensible Access Control Markup Language (XACML) Version 1.1
    target: https://www.oasis-open.org/committees/xacml/repository/cs-xacml-specification-1.1.pdf
    author:
    - name: Simon Godik
      role: editor
      org: Overxeer
    - name: Tim Moses (Ed.)
      role: editor
      org: Entrust
  date: 2006

...

--- abstract

Trust registries come in many forms; ETSI trust status lists, OpenID Federation, ledgers. This document describes a simple protocol in the form of a profile of AuthZen that provides a local interface to one or more trust registries. The protocol is mant to be used as a local abstraction layer for any application that needs to evaluate trust.

--- middle

# Introduction

Technical trust in systems using asymmetric cryptography often involves binding a name to a public key. One such example is, given an X.509 certificate (as a representative of a public key and name), determining its validity relative to a set of trust anchors by means of PKIX path construction and path validation. In this example the trust registry is the set of trust anchors together with the rules for path validation and construction set down in [RFC5280].

The proliferation of distributed identity systems have led to the development of a multitude of trust registries each with their own APIs for querying the registry and rules for evaluating trust. Application developers are often faced with the choice of choosing one of these trust registries which leads to interoperability problems. It is often common for an service to register with multiple trust registries in order to reach all intended audiences.

This document describes an API for trust evaluation that is intended to fill a role similar to the stub resolver in the DNS architecture. The API is defined as a profile of [AuthZen]. AuthZen is a proposed standard for communicating between an authorization policy enforcement point (PEP) and a policy decision point (PDP).


# Conventions and Definitions

{::boilerplate bcp14-tagged}

This specification uses the terms "PDP" and "PEP" defined by [NIST.SP.800-162] and [XACML]. A trust registry refers to any service that provides a binding (or mapping) between public keys and names. This is referred to as "name to key" or name-to-key binding.

# Endpoints

Implementations of this specification MUST provide the `/evaluation` endpoint and SHOULD also provide the `/evaluations` and discovery endpoints. The `/search` endpoint MAY be provided but providing this endpoint may provide significant challenges for this profile and clients MUST NOT assume that it is present.

# Authorization Request

This profile implements the following semantic: The client (PEP) requests that the server (PDP) authorizes the binding between the name specified by the Subject with the public key specified by the Resource. Optionally the Action is used to constrain the authorization to a specific role that the entity that the public key is bound to must have for the authorization to be approved. The PDP may also attempt to *resolve* the name into metadata that provides additional information about the name-to-key binding.

## Subject

Subject is used to represent the name part of the name-to-key binding.

The `subject` datafield MUST be present in requests and MUST contain the following elements:

- `id` MUST be the name bound to the public key to be validated or resolved
- `type` MUST be the constant string `"key"`

## Resource

The `resource` datafield MUST be present in requests and MUST contain the following elements:

- `id` MUST be the name bound to the public key to be validated or resolved. Furthermore, the value of the `resource.id` element MUST be the same string as in the `subject.id` element.
- `type` MAY be present and if present MUST be one of "jwk" or "x5c".
- `key` If present, MUST be the public key in a format that depends on the `type`. If `type` is absent then `key` MUST NOT be present.

If `type`is present then,

- If `type` is `"jwk"` then `key` MUST contain a JWK ([RFC7517]) format key.
- If `type` is `"x5c"` then `key` MUST contain an array of base64 encoded X.509 certificates formatted according to section 4.7 of [RFC7517].

Some trust registries support unambiguous name-to-key discovery. For such trust registries `key` and `type` MAY be elided from the Resource as described above.

When `type` and `key` is present however, a PDP implementing this specification MUST validate that the key is bound to `subject.id` even if `subject.id` is a name bound to a trust registry that supports unambiguous name-to-key discovery.

The PDP MAY include additional *metadata* associated with `subject.id` in the result. The method by which this is done is an implementation detail but for instance when the `subject.id` is a "DID" then the resolution MAY be done by the lookup process of a supported DID method.  It is RECOMMENDED that PDPs that support such trust registries return the appropriate metadata in the response.

Other specifications may define additional key formats in the future.

## Action

The `action` datafield MAY be present in requests and SHOULD if present be used to represent the role associated with the name-to-key binding. This is used to distinguish different uses of the same name-to-key binding. For instance the `action` can be used to request authorization that a X.509 certificate is allowed to act as a TLS server endpoint or as a digital credential issuer.

If the `action` is present then it MUST contain at least the `name` parameter which MUST contain a string that represents the role. The interpretation of the role depends on the deployment.

## Context

The `context` datafield MAY be present in requests but MUST NOT contain information that is critical for the correct processing of the request.

# Authorization Response

The Authorization Response MAY return metadata associated with `subject.id` in the response using the `trust_metadata` field. When the request `type` is absent then the `trust_metadata` field SHOULD be present.

# Examples (non-normative)

The following example is a query to check if a provided certificate chain is bound to the name "did:foo:bla" and is allowed act as a EUDI wallet provider.

~~~
{
  "type": "authzen",
  "request": {
      "subject": {
        "type": "key",
        "id": "did:foo:bla"
    },
    "resource": {
      "type": "x5c",
      "id": "did:foo:bla",
      "key": ["... x5c data ..."]
    },
    "action": {
      "name": "http://ec.europa.eu/NS/wallet-provider",
    }
  }
}
~~~

The following example is a query to check if a provided certificate chain is bound to "www.example.com" and is allowed to act as a TLS server.


~~~
{
  "type": "authzen",
  "request": {
      "subject": {
        "type": "key",
        "id": "www.example.com"
    },
    "resource": {
      "type": "x5c",
      "id": "www.example.com",
      "key": ["... x5c data ..."],
    },
    "action": {
      "name": "TODO:oid:tls-server",
    }
  }
}
~~~

The following is an example response with no additional context:

~~~
{
  "decision": true
}
~~~

The following is an example response with trust_metadata that contains an (abbreviated) DID document.

~~~
{
  "decision": true,
  "context": {
    trust_metadata: {
      {
        "@context": [
          "https://www.w3.org/ns/did/v1",
          "https://w3id.org/security/suites/ed25519-2020/v1"
        ],
        "id": "did:example:123",
        ....
    }
  }
}
~~~

The following is an hypothetical response with error messages:

~~~
{
  "decision": false,
  "context": {
    "reason": {
      "403": "Unknown service - contact helpdesk@registry.example.com for support using the following identifier: #ID4711"
    }
  }
}
~~~

# AuthZen Trust as a DID resolver

As should be obvious from the specification above, a DID resolver as specified in section 7 of [DID] share many properties with this specification. Notable differences is that error handling is slightly different and content negotiation is handled by the PDP which means that DID resolution options (section 7.1.1 of [DID]) isn't needed in this case.

# Security Considerations

The protocol described in this specification is meant to be used by applications that share a common security domain and it may be perfectly reasonable for deployments of this specification to be deployed without authentication on "localhost" or in situations where security requirements for the protocol is provided elsewhere in the stack. In general implementations of this specification MAY implement [RFC6749] authentication for the purpose of authenticating the client (PDP) to the server (PEP) and SHOULD provide a way for the PDP to be authenticated to the client.

In addition to the above the security considerations for authentication for AuthZen applies in equal measure to this profile.

# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
