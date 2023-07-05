---
title: "Privacy Pass Issuance Protocol Extensions"
abbrev: "Privacy Pass Issuance Protocol Extensions"
category: std

docname: draft-hendrickson-privacypass-issuance-extensions-latest
submissiontype: IETF
number:
date:
consensus: true
v: 3
area: "Security"
workgroup: "Privacy Pass"
keyword:
 - token
 - extensions
venue:
  group: "Privacy Pass"
  type: "Working Group"
  mail: "privacy-pass@ietf.org"
  github: "chris-wood/draft-hendrickson-privacypass-token-extensions"
  latest: "https://chris-wood.github.io/draft-hendrickson-privacypass-token-extensions/draft-hendrickson-privacypass-token-extensions.html"

author:
 -
    fullname: Scott Hendrickson
    organization: Google
    email: "scott@shendrickson.com"
 -
    fullname: Christopher A. Wood
    organization: Cloudflare, Inc.
    email: caw@heapingbits.net

normative:
   AUTH-EXTENSIONS: I-D.wood-privacypass-auth-scheme-extensions
   AUTH-SCHEME: I-D.ietf-privacypass-auth-scheme
   EXTENDED-ISSUANCE: I-D.hendrickson-privacypass-public-metadata-issuance
   BASIC-ISSUANCE: I-D.ietf-privacypass-protocol
   ARCHITECTURE: I-D.ietf-privacypass-architecture


--- abstract

This document describes several extensions for Privacy Pass issuance
protocols, including one for token expiration and one for geolocation
information.

--- middle

# Introduction

Some Privacy Pass token types support binding additional information to
the tokens, often referred to as public metadata. {{AUTH-EXTENSIONS}} describes
an extension parameter to the basic PrivateToken HTTP authentication scheme {{AUTH-SCHEME}}
for supplying this metadata alongside a token. {{EXTENDED-ISSUANCE}} describes
variants of the basic Privacy Pass issuance protocols {{BASIC-ISSUANCE}} that
support issuing tokens with public metadata. However, there are no existing
extensions defined to make use of these protocol extensions.

This document defines several extensions for Privacy Pass token types with public metadata,
including one for token expiration and one for geolocation. The use case and deployment
considerations, especially with respect to the resulting privacy loss, are also discussed.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Expiration Extension

The expiration extension is an extension used to convey the expiration for an issued
token. It is useful for Privacy Pass deployments that make use of cached tokens, i.e.,
those that are not bound to a specific TokenChallenge redemption context, without having
to frequently rotate issuing public keys.

For example, consider a Privacy Pass deployment wherein clients use cached tokens that
are valid for one hour. Clients could pre-fetch these tokens each hour and the issuer
and origin could rotate the verification key every hour to force expiration. Alternatively,
clients could pre-fetch tokens for the entire day all at once, including an expiration
timestamp in each token to indicate the time window for which the token is valid.

Including a specific expiration timestamp could reveal specific information about a client,
if, say, a client has a uniquely skewed clock. For that reason, clients can round the timestamp,
resulting in a loss of precision but overall less unique value. Clients do this by choosing an
expiration timestamp and the precision they would like, e.g., the UNIX time of expiration rounded
to the nearest hour, day, or week.

The value of this extension is an ExpirationTimestamp, defined as follows.

~~~
struct {
   uint64 timestamp_precision;
   uint64 timestamp;
} ExpirationTimestmap;
~~~

The ExpirationTimestmap fields are defined as follows:

- "timestamp_precision" is an 8-octet integer, in network byte order, representing the granularity of the timestamp.

- "timestamp" is an 8-octet integer, in network byte order, representing the expiration timestamp. The
  expiration timestamp is the UNIX time in seconds at which a token expires, divided by "timestamp_precision".

As an example, an ExpirationTimestamp structure with the following value would be interpreted as an
expiration timestamp of 1688583600, i.e., July 05, 2023 at 19:00:00 GMT+0000:

~~~
struct {
   uint64 timestamp_precision = 3600;
   uint64 timestamp = 1688583600;
} ExpirationTimestmap;
~~~

# Security Considerations

Use of extensions risks revealing additional information to parties that see these extensions, including
the Attester, Issuer, and Origin. Each extension in this document specifies security and privacy
considerations relevant to its use. More general information regarding the use of extensions and their
possible impact on client privacy can be found in {{ARCHITECTURE}}.

# IANA Considerations

This document registers the following entries into the "Privacy Pass PrivateToken Extensions" registry.

- Expiration extension
   - Type: 0x0001
   - Name: Expiration
   - Value: ExpirationTimestamp value as defined in {{expiration-extension}}
   - Reference: This document
   - Notes: Any notes associated with the entry


--- back

# Acknowledgments
{:numbered="false"}

This document received input and feedback from Jim Laskey.
