---
title: Resource Digests for HTTP
abbrev: RDHTTP
docname: draft-polli-resource-digests-latest
category: std

ipr: -
area: General
workgroup: 
keyword: Internet-Draft

stand_alone: yes
pi: [toc, tocindent, sortrefs, symrefs, strict, compact, comments, inline, docmapping]

author:
 -
    ins: R. Polli
    name: Roberto Polli
    organization: Team Digitale
    email: robipolli@gmail.com
normative:
  RFC5843:
  RFC2119:
  RFC3230:
  RFC7230:
  RFC7231:
  RFC7233:
  RFC4648:
  RFC1321:
  FIPS180-4:
    title: NIST FIPS 180-4, Secure Hash Standard
    author:
      name: NIST
      ins: National Institute of Standards and Technology, U.S. Department of Commerce
    date: 2012-03
    target: http://csrc.nist.gov/publications/fips/fips180-4/fips-180-4.pdf
 
 
informative:
  RFC2818:
  RFC6962:
  RFC7233:
  RFC3230:

--- abstract

This document defines the Digest and Want-Digest header fields for HTTP, thus allowing client
 and server to negotiate an integrity checksum of the exchanged resource representation.

This document obsoletes RFC 3230. It replaces the term "instance" with "representation",
which makes it consistent  with the HTTP Semantic and Context defined in RFC 7231.   


--- note_Note_to_Readers

*RFC EDITOR: please remove this section before publication*

Discussion of this draft takes place on the HTTP working group mailing list
(ietf-http-wg@w3.org), which is archived at
<https://lists.w3.org/Archives/Public/ietf-http-wg/>.

The source code and issues list for this draft can be found at
<https://github.com/martinthomson/http-mice>.


--- middle

# Introduction

Integrity protection for HTTP content is typically achieved via TCP or HTTPS [RFC2818].
However, additional integrity protection might be desirable for some use cases. 
This might be for additional protection against failures or attack (see [SRI]), 
programming errors, corruption of stored data or because content needs 
to remain unmodified throughout multiple HTTPS-protected exchanges.

## Brief history of integrity headers 

The Content-MD5 header field was originally introduced to provide integrity, but HTTP/1.1 in https://tools.ietf.org/html/rfc7231#appendix-B obsoleted it:

  > The Content-MD5 header field has been removed because it was
  >  inconsistently implemented with respect to partial responses.

RFC 3230 provided a more flexible solution introducing the concept of "instance",
and the headers Digest and Want-Digest.

## This proposal

The concept of `selected representation` defined in RFC 7231 made RFC 3230 definitions
inconsistent with the current standard. A refresh was then required.

This document updates the Digest and Want-Digest header field definitions to align with RFC 7231 concepts.

This approach can be easily adapted to use-cases where the transferred data 
does require some sort of manipulation to be considered a representation 
or conveys a partial representation of a resource (eg. Range Requests). 

Changes are semantically compatible with existing implementations 
and better cover both the request and response cases.

Being calculated on the selected representation, the
Digest is tied to the Content-Coding. 

A given resource has thus multiple possible digests values.
To allow both parties to exchange a Digest of a representation 
with only the Identity [content coding](https://tools.ietf.org/html/rfc7231#section-3.1.2.1)
applied, two more algorithms are added (id-sha-256 and id-sha-512).

## Goals

The goals of this proposal are:

   1. Digest coverage for either the resource's `representation data` or `selected representation data`
      communicated via HTTP.

   2. Support for multiple digest algorithms.

   3. Negotiation of the use of digests.

The goals do not include:

   -  header integrity
      The digest mechanisms described here cover only representation data,
      and do not protect the integrity of associated
      representation metadata headers or other message headers.

   -  authentication
      The digest mechanisms described here are not meant to support
      authentication of the source of a digest or of a message or
      anything else.  These mechanisms, therefore, are not a sufficient
      defense against many kinds of malicious attacks.

   -  privacy
      Digest mechanisms do not provide message privacy.

   -  authorization
      The digest mechanisms described here are not meant to support
      authorization or other kinds of access controls.


## Notational Conventions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be
interpreted as described in [RFC2119].

The definitions "representation", "selected representation", "representation data", 
"representation metadata" and "payload body" in this document are to be
interpreted as described in [RFC7231].


# Resource representation and representation-data

This document uses the definitions "representation", "selected representation", 
"representation data", "representation metadata" and "payload body" defined in RFC 7231.

The value of the digest is calculated against the selected representation of a 
resource, that is  defined in RFC 7231 as:

~~~
  representation-data := Content-Encoding( Content-Type( bits ) )
~~~

and is thus independent of Transfer-Coding and other transformation applied from Intermediaries
or tied to Range Requests.

Note that [Content-Encoding](https://tools.ietf.org/html/rfc7231#section-3.1.2.2) can be an ordered list.



# Digest Algorithm values {#algorithms}

   Digest algorithm values are used to indicate a specific digest
   computation.  For some algorithms, one or more parameters may be
   supplied.

~~~
      digest-algorithm = token
~~~

   The BNF for "parameter" is as is used in RFC 7230.  All digest-
   algorithm values are case-insensitive.


   The Internet Assigned Numbers Authority (IANA) acts as a registry for
   digest-algorithm values.  The registry contains the
   following tokens.
   
   **NB: This RFC updates RFC 5843 which is still delegated for all algorithms updates** 

  - SHA-256: The SHA-256 algorithm [FIPS-180-3].  The output of
      this algorithm is encoded using the base64 encoding [RFC4648].

      Reference: [FIPS-180-3], [RFC4648], this document.

  - SHA-512: The SHA-512 algorithm [FIPS-180-3].  The output of
      this algorithm is encoded using the base64 encoding [RFC4648].

      Reference: [FIPS-180-3], [RFC4648], this document.


  - MD5               The MD5 algorithm, as specified in [RFC1321].
                     The output of this algorithm is encoded using the
                     base64 encoding  {{?RFC4648}}.

  - SHA               The SHA-1 algorithm [12].  The output of this
                     algorithm is encoded using the base64 encoding  {{?RFC4648}}.

  - UNIXsum           The algorithm computed by the UNIX "sum" command,
                     as defined by the Single UNIX Specification,
                     Version 2 [13].  The output of this algorithm is an
                     ASCII decimal-digit string representing the 16-bit
                     checksum, which is the first word of the output of
                     the UNIX "sum" command.

  - UNIXcksum         The algorithm computed by the UNIX "cksum" command,
                     as defined by the Single UNIX Specification,
                     Version 2 [13].  The output of this algorithm is an
                     ASCII digit string representing the 32-bit CRC,
                     which is the first word of the output of the UNIX
                     "cksum" command.

To allow sender and recipient to provide a checksum which is independent from the Content-Coding,
the following additional algorithms are defined:

   - id-sha-512 	The sha-512 digest of the representation-data of the resource when only the Identity
                       	content coding is applied (eg. `Content-Encoding: identity`)
   - id-sha-256 	The sha-256 digest of the representation-data of the resource when only the Identity
                       	content coding is applied (eg. `Content-Encoding: identity`)

   If other digest-algorithm values are defined, the associated encoding
   MUST either be represented as a quoted string, or MUST NOT include
   ";" or "," in the character sets used for the encoding.

## Representation digest {#digest}

A representation  digest is the value of the output of a digest
algorithm, together with an indication of the algorithm used (and any
parameters).

~~~
    representation-data-digest = digest-algorithm "="
                            <encoded digest output>
~~~

The digest is computed on the entire `representation data` of the resource
and is tied to the Content-Type and the Content-Encoding of the message.

The encoded digest output uses the encoding format defined for the
specific digest-algorithm.

### digest-algorithm encoding examples
The sha-256 digest-algorithm uses base64 encoding

~~~
sha-256=......
~~~

The "UNIXsum" digest-algorithm uses ASCII string of decimal digits.

~~~
UNIXsum=30637
~~~

# Header Specifications

The following headers are defined

## Want-Digest

The Want-Digest message header field indicates the sender's desire to
receive a representation digest on messages associated with the Request-
URI and representation metadata.

    Want-Digest = "Want-Digest" ":"
                     #(digest-algorithm [ ";" "q" "=" qvalue])

If a digest-algorithm is not accompanied by a qvalue, it is treated
as if its associated qvalue were 1.0.

The sender is willing to accept a digest-algorithm if and only if it
is listed in a Want-Digest header field of a message, and its qvalue
is non-zero.

If multiple acceptable digest-algorithm values are given, the
sender's preferred digest-algorithm is the one (or ones) with the
highest qvalue.

Examples:

   Want-Digest: sha-256
   Want-Digest: SHA-256;q=0.3, sha;q=1

## Digest

The Digest header field provides a digest of the representation data

~~~
      Digest = "Digest" ":" #(representation-data-digest)
~~~

`Representation data` might be:

- fully contained in the message body, 
- partially-contained in the message body,
- or not at all contained in the message body.

The resource is specified by the effective
Request-URI and any cache-validator contained in the message.

For example, in a response to a HEAD request, the digest is calculated using  the
representation data that would have been enclosed in the payload body
if the same request had been a GET.

Digest can be used in requests too. 
Returned value depends on the representation metadata headers.

A Digest header field MAY contain multiple representation-data-digest values.
This could be useful for responses expected to reside in caches
shared by users with different browsers, for example.

A recipient MAY ignore any or all of the representation-data-digests in a Digest
header field.

A sender MAY send a representation-data-digest using a digest-algorithm without
knowing whether the recipient supports the digest-algorithm, or even
knowing that the recipient will ignore it.

...

# Deprecate Negotiation of Content-MD5
This RFC deprecates the negotiation of Content-MD5 as 
this header has been obsoleted by RFC7231

This RFC DISCOURAGES the use of MD5 algorithm for security reasons.

# Examples


## Unsolicited Digest response

### Representation data is fully contained in the payload

~~~
Request:

  GET /items/123

Response:

  HTTP/1.1 200 Ok
  Content-Type: application/json
  Content-Encoding: identity 
  Digest: sha-256=X48E9qOokqqrvdts8nOJRJN3OWDUoyWxBf7kbu9DBPE=

  {"hello": "world"}
~~~

###  Representation data is not contained in the payload

~~~
Request:

  HEAD /items/123

Response:

  HTTP/1.1 200 Ok
  Content-Type: application/json
  Content-Encoding: identity 
  Digest: sha-256=X48E9qOokqqrvdts8nOJRJN3OWDUoyWxBf7kbu9DBPE=

~~~

###  Representation data is partially contained in the payload (range request?)

~~~

Request:

  GET /items/123
...

Response:

  HTTP/1.1 200 Ok
  Content-Type: application/json
  Content-Encoding: identity 
  Digest: sha-256=X48E9qOokqqrvdts8nOJRJN3OWDUoyWxBf7kbu9DBPE=

~~~

### Digest in both Request and Response. Returned value depends on representation metadata 

Digest can be used in requests too. Returned value depends on the representation metadata headers.

~~~
Request:

PUT /items/123
Content-Type: application/json
Content-Encoding: identity
Accept-Encoding: br
Digest: sha-256=4REjxQ4yrqUVicfSKYNO/cF9zNj5ANbzgDZt3/h3Qxo=

{"hello": "world"}

Response:

Content-Type: application/json
Content-Encoding: br
Digest: sha-256=X48E9qOokqqrvdts8nOJRJN3OWDUoyWxBf7kbu9DBPE=

b'\x8b\x08\x80{"hello": "world"}\x03'

~~~

## Want-Digest solicited digest responses

### Client request data is fully contained in the payload

The client requests a digest, preferring sha. The server is free to reply with sha-256 anyway.

~~~
Request:

  GET /items/123
  Want-Digest: sha-256;q=0.3, sha;q=1

Response:

  HTTP/1.1 200 Ok
  Content-Type: application/json
  Content-Encoding: identity 
  Digest: sha-256=X48E9qOokqqrvdts8nOJRJN3OWDUoyWxBf7kbu9DBPE=

  {"hello": "world"}
~~~

###  A client requests an unsupported Digest, the server MAY reply with an unsupported digest

The client requests a sha digest only. The server is currently free to reply with a Digest containing an unsupported algorithm

Request:

  GET /items/123
  Want-Digest: sha;q=1

Response:

  HTTP/1.1 200 Ok
  Content-Type: application/json
  Content-Encoding: identity 
  Digest: sha-256=X48E9qOokqqrvdts8nOJRJN3OWDUoyWxBf7kbu9DBPE=

  {"hello": "world"}

Eg.  A client requests an unsupported Digest, the server MAY reply with a 400

The client requests a sha Digest, the server advises for sha-256 and sha-512

Request:

  GET /items/123
  Want-Digest: sha;q=1

Response:

  HTTP/1.1 400 Bad Request
  Want-Digest: sha-256, sha-512


...


# Security Considerations

The integrity of an entire message body depends on the means by which the
integrity proof for the first record is protected.  If this value comes from the
same place as the message, then this provides only limited protection against
transport-level errors (something that TLS provides adequate protection
against).

Separate protection for header fields might be provided by other means if the
first record retrieved is the first record in the message, but range requests do
not allow for this option.


## Message Truncation

...

## Algorithm Agility

...

# IANA Considerations

## The "mi-sha256" HTTP Content Encoding

This memo registers the "mi-sha256" HTTP content-coding in the HTTP Content Codings
Registry, as detailed in {{encoding}}.

* Name: mi-sha256
* Description: A Merkle Hash Tree based content encoding that provides
               progressive integrity.
* Reference: this specification


## The "mi-sha256" Digest Algorithm {#iana-digest}

This memo registers the "mi-sha256" digest algorithm in the [HTTP Digest
Algorithm
Values](https://www.iana.org/assignments/http-dig-alg/http-dig-alg.xhtml)
registry:

* Digest Algorithm: mi-sha256
* Description: As specified in {{digest-mi-sha256}}.


--- back

# Acknowledgements

David Benjamin and Erik Nygren both separately suggested that something like
this might be valuable.  James Manger and Eric Rescorla provided useful
feedback.


# FAQ

1. Why remove all references to content-md5?

   Those were unnecessary to understanding and using this spec.

2. Why remove references to instance manipulation?

   Unnecessary again for correctly using and applying the spec. An
   example with Range Request is more than enough.

3. How to use `Digest` with `PATCH` method?

   The PATCH verb brings some complexities (eg. about representation metadata headers, patch document format, ...),
   eg ```Note that entity-headers contained in the request apply only to the
   contained patch document and MUST NOT be applied to the resource
   being modified. ``` see [rfc5789#section-2]
   Moreover a `200 OK` response to a PATCH request would contain the digest of the result of applying the patch
   and relate to the etag of this new object, but this behavior is probably tighly coupled to the application logic
   and the client has a low probability of guessing the actual outcome of the operation because there may have been other 
   changes to the document (by the server, or other clients), making all this difficult to implement.
