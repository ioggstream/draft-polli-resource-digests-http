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

## Attacker non-capabilities

# Attacks and mitigations

## Alteration of HTTP message headers

### The attack

An intermediary can alter an HTTP header by change or purpose.

### Proposed mitigation

Use an end-to-end transport-layer security mechanism.


## Alteration of HTTP representation-metadata headers

### The attack

An TLS-terminator intermediary can alter representation-metadata by chance or purpose.

### Proposed mitigation

Use a signature mechanism that covers both `Digest` and the representation-metadata.

## Send a Digest header with a different payload

### The attack

A TLS-terminator intermediary or a malicious User can replace the representation-data with another
one having the same digest-algorithm. 

### Proposed mitigation

Use a digest-algoritm wich is not subject to collision (eg. sha-256).

## Server resources exhaustion for calculating Digest on partial representations

### The attack

A malicious User could exahust server resources via:

- re-construction of the complete representation
- digest calculation of the complete representation

### Proposed mitigation

If you are exposed to this kind of attack because 
re-constructing complete representations of your 
objects is computationally intensive, avoid using
Digest with mechanisms like Range-Requests and PATCH.

