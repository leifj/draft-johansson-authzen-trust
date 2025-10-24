---
###
# Internet-Draft Markdown Template
#
# Rename this file from draft-todo-yourname-protocol.md to get started.
# Draft name format is "draft-<yourname>-<workgroup>-<name>.md".
#
# For initial setup, you only need to edit the first block of fields.
# Only "title" needs to be changed; delete "abbrev" if your title is short.
# Any other content can be edited, but be careful not to introduce errors.
# Some fields will be set automatically during setup if they are unchanged.
#
# Don't include "-00" or "-latest" in the filename.
# Labels in the form draft-<yourname>-<workgroup>-<name>-latest are used by
# the tools to refer to the current version; see "docname" for example.
#
# This template uses kramdown-rfc: https://github.com/cabo/kramdown-rfc
# You can replace the entire file if you prefer a different format.
# Change the file extension to match the format (.xml for XML, etc...)
#
###
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

informative:

...

--- abstract

Trust registries come in many forms; ETSI trust status lists, OpenID Federation, ledgers. This document describes a simple protocol in the form of a profile of AuthZen that provides a local interface to one or more trust registries. The protocol is mant to be used as a local abstraction layer for any application that needs to evaluate trust.

--- middle

# Introduction

Technical trust in systems using asymmetric cryptography amounts to answering the question: is a given public key pk bound to a name n in context c. One example is, given an X509 certificate (as a representative of a public key and name), doing PKIX path construction and subsequent path validation to determine if the X509 is "valid" relative to a given set of trust roots. In this example the trust registry is the set of roots together with the rules for path validation and construction set down in RFC 5280 and any additional local policy applied to the validation.

The proliferation of distributed architectures have led to the development of a multitude of trust registries each with their own APIs for querying the registry and rules for evaluating trust. Application developers are often faced with the choice of supporting exactly one of these which of course leads to interoperability challenges.

This document describes a local abstraction layer for trust evaluation that is intended to fill a role similar to the stub resolver in the DNS architecture.


# Conventions and Definitions

{::boilerplate bcp14-tagged}


# Security Considerations

The protocol described in this specification is meant to be used by applications that share a common trust domain and it may be perfectly reasonable for deployments of this specification to run on "localhost" and in other situations where security properties for the protocol is provided elsewhere in the stack. Nevertheless the recommendations for authentication for AuthZen applies to this profile aswell and deployments SHOULD deploy the AuthZen trust endpoint with provisions for mutual authentication to ensure that false trust decisions cannot affect the overall system.

# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
