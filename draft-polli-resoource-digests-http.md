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

This document obsoletes RFC 3230, which initially defined those headers. That definition
is now inconsistent with the HTTP Semantic and Context specified in RFC 7231.   


--- note_Note_to_Readers

*RFC EDITOR: please remove this section before publication*

Discussion of this draft takes place on the HTTP working group mailing list
(ietf-http-wg@w3.org), which is archived at
<https://lists.w3.org/Archives/Public/ietf-http-wg/>.

The source code and issues list for this draft can be found at
<https://github.com/martinthomson/http-mice>.


--- middle

# Introduction

Although HTTP is typically layered over a reliable transport
protocol, such as TCP, this does not guarantee reliable transport of
information from sender to recipient.  Various problems, including
undetected transmission errors, programming errors, corruption of
stored data, and malicious intervention can cause errors in the
transmitted information.

A common approach to the problem of data integrity in a network
protocol or distributed system, such as HTTP, is the use of digests,
checksums, or hash values.  The sender computes a digest and sends it
with the data; the recipient computes a digest of the received data,
and then verifies the integrity of this data by comparing the
digests.

 The Content-MD5 header field was originally introduced to provide integrity, 
 but [HTTP/1.1](https://tools.ietf.org/html/rfc7231#appendix-B) obsoleted it:

  > The Content-MD5 header field has been removed because it was
  >  inconsistently implemented with respect to partial responses.

The proposed solution uses the checksum of the selected representation of a resource. 
This approach is more flexible and can be easily adapted to use-cases where the transferred 
data does require some sort of manipulation to be considered a representation (eg. Range Requests RFC 7233). 

This information can be:

  - sent in response to a HEAD request with the same value of the corresponding GET request;
  - sent alongside a 206 (Partial Content) response in Range Requests or similar mechanisms. 
    Its value can be validated once all representation data has been collected;

Being calculated on the selected representation this value is tied to the representation-data and 
the Content-Coding. A given resource has thus multiple possible digests dependending on the applied Content-Codings.

## Goals

The goals of this proposal are:

  1. Digest coverage for representation data communicated via HTTP.

  2. Support for multiple digest algorithms.

  3. Negotiation of the use of digests.

The goals do not include:

  -  header integrity
     The digest mechanisms described here cover only representation data, and do not protect the integrity of associated
     representation metadata headers or other message headers.

  -  authentication
     The digest mechanisms described here are not meant to support
     authentication of the source of a digest or of a message or
     anything else.  These mechanisms, therefore, are not sufficient
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

   If other digest-algorithm values are defined, the associated encoding
   MUST either be represented as a quoted string, or MUST NOT include
   ";" or "," in the character sets used for the encoding.

## Content Encoding Structure {#records}

In order to produce the final content encoding the content of the message is
split into equal-sized records.  The final record can contain less than the
defined record size.

For non-empty payloads, the record size is included in the first 8 octets of the
message as an unsigned 64-bit integer.  This refers to the length of each data
block.

The final encoded stream comprises of the record size ("rs"), plus a sequence of
records, each "rs" octets in length.  Each record, other than the last, is
followed by a 32 octet proof for the record that follows.  This allows a
receiver to validate and act upon each record after receiving the proof that
precedes it.  The final record is not followed by a proof.

Note:

: This content encoding increases the size of a message by 8 plus 32 octets
  times the length of the message divided by the record size, rounded up, less
  one.  That is, 8 + 32 * (ceil(length / rs) - 1).

Constructing a message with the "mi-sha256" content encoding requires processing
of the records in reverse order, inserting the proof derived from each record
before that record.

This structure permits the use of range requests [RFC7233]. However, to validate
a given record, a contiguous sequence of records back to the start of the
message is needed.


## Validating Integrity Proofs {#validating}

...

# The "mi-sha256" Digest Algorithm {#digest-mi-sha256}


# Examples

## Simple Example

blabla

~~~
HTTP/1.1 200 OK
Digest: mi-sha256=dcRDgR2GM35DluAV13PzgnG6+pvQwPywfFvAu1UeFrs=
Content-Encoding: mi-sha256
Content-Length: 49

<0x0000000000000029>When I grow up, I want to be a watermelon
~~~


## Example with Multiple Records

 blabla
~~~
PUT /test HTTP/1.1
Host: example.com
Digest: mi-sha256=IVa9shfs0nyKEhHqtB3WVNANJ2Njm5KjQLjRtnbkYJ4=
Content-Encoding: mi-sha256
Content-Length: 113

<0x0000000000000010>When I grow up,
OElbplJlPK+Rv6JNK6p5/515IaoPoZo+2elWL7OQ60A=
I want to be a w
iPMpmgExHPrbEX3/RvwP4d16fWlK4l++p75PUu_KyN0=
atermelon
~~~

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

