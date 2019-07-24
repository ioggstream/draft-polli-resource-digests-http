# Security issues for HTTP Digest

## Actors

- User
- User-Agent: see RFC7230
- Intermediaries that do not do TLS termination
- Intermediaries that do TLS termination.

- Application: the application actually running the service

## Attacker capabilities

1. Intermediaries can alter HTTP messages

2. Intermediaries that do transport-layer termination can alter HTTP messages

3. The application can erroneously or purposedly alter the message

4. User can forge messages with the same checksum values

## Attacker non-capabilities

# Attacks and mitigations

## Alteration of HTTP message headers

### The attack

An intermediary can alter an HTTP header by chance or purpose.

### Proposed mitigation

Use an end-to-end transport-layer security mechanism.


## Alteration of HTTP representation-metadata headers

### The attack

A TLS-terminator intermediary can alter representation-metadata by chance or purpose.

### Proposed mitigation

Use a signature mechanism that covers both `Digest` and the representation-metadata.

## Send a Digest header with a different payload

### The attack

A TLS-terminator intermediary can replace the representation-data with another
one having the same digest-algorithm.

A malicious User can forge two messages with:

  - different representation-data
  - same Digest header values
  
send an HTTP message with the first representation-data
then pretend he sent the one with the second representation-data.

A malicious Server could implement a similar behavior.

### Proposed mitigation

Use a digest-algorithm that is not subject to collision (eg. sha-256),
or refuse to use digest-algorithms subect to collision.

## Server resource exhaustion for calculating Digest on partial representations

### The attack

A malicious User could exhaust server resources via:

1- reconstruction of the complete representation
2- digest calculation of the complete representation
3- forcing the server to evaluate many different digest-algorithms for a given representation

### Proposed mitigation

If you are exposed to this kind of attack because 
reconstructing complete representations of your 
objects is computationally intensive, avoid using
Digest with mechanisms like Range-Requests and PATCH.

If you support multiple digest-algorithms:

- limit the supported digest-algorithms to the
  ones you really care for;
- establish a policy for validating only the digest
  using the stronger digest-algorithm.

## Confidence in recipient's Digest validation 

### The attack

The recipient may skip Digest header validation by chance or purpose
and still be compliant to the specification: as explained in {#digest-header} 
the recipient MAY ignore any or all the received representation-data-digests.

This is because Digest was originally conceived in RFC3230 as a tool 
for the recipient to validate the received representation, but not
for the sender to receive an integrity proof of the communication. 

You should not rely on the recipient having correctly verified the Digest value
of the selected representation.

### Proposed mitigation

If you want the recipient to validate the `Digest`:

- agree on `Digest` processing with your peer, like it is
  done in [MICE](https://github.com/martinthomson/http-mice/blob/master/draft-thomson-http-mice.md)
