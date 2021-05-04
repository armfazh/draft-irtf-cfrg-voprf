---
title: Oblivious Pseudorandom Functions (OPRFs) using Prime-Order Groups
abbrev: OPRFs
docname: draft-irtf-cfrg-voprf-latest
date:
category: info

ipr: trust200902
keyword: Internet-Draft

stand_alone: yes
pi: [toc, sortrefs, symrefs]

author:
 -
    ins: A. Davidson
    name: Alex Davidson
    email: alex.davidson92@gmail.com
 -
    ins: A. Faz-Hernandez
    name: Armando Faz-Hernandez
    org: Cloudflare
    street: 101 Townsend St
    city: San Francisco
    country: United States of America
    email: armfazh@cloudflare.com
 -
    ins: N. Sullivan
    name: Nick Sullivan
    org: Cloudflare
    street: 101 Townsend St
    city: San Francisco
    country: United States of America
    email: nick@cloudflare.com
 -
    ins: C. A. Wood
    name: Christopher A. Wood
    org: Cloudflare
    street: 101 Townsend St
    city: San Francisco
    country: United States of America
    email: caw@heapingbits.net

normative:
  RFC2119:
  RFC7748:
  PrivacyPass:
    title: Privacy Pass
    target: https://github.com/privacypass/challenge-bypass-server
    date: false
  BB04:
    title: Short Signatures Without Random Oracles
    target: http://ai.stanford.edu/~xb/eurocrypt04a/bbsigs.pdf
    date: false
    authors:
      -
        ins: D. Boneh
        org: Stanford University, CA, USA
      -
        ins: X. Boyen
        org: Voltage Security, Palo Alto, CA, USA
  BG04:
    title: The Static Diffie-Hellman Problem
    target: https://eprint.iacr.org/2004/306
    date: false
    authors:
      -
        ins: D. Brown
        org: Certicom Research
      -
        ins: R. Gallant
        org: Certicom Research
  Cheon06:
    title: Security Analysis of the Strong Diffie-Hellman Problem
    target: https://www.iacr.org/archive/eurocrypt2006/40040001/40040001.pdf
    date: false
    authors:
      -
        ins: J. H. Cheon
        org: Seoul National University, Republic of Korea
  JKKX16:
    title: Highly-Efficient and Composable Password-Protected Secret Sharing (Or, How to Protect Your Bitcoin Wallet Online)
    target: https://eprint.iacr.org/2016/144
    date: false
    authors:
      -
        ins: S. Jarecki
        org: UC Irvine, CA, USA
      -
        ins: A. Kiayias
        org: University of Athens, Greece
      -
        ins: H. Krawczyk
        org: IBM Research, NY, USA
      -
        ins: Jiayu Xu
        org: UC Irvine, CA, USA
  JKK14:
    title:  Round-Optimal Password-Protected Secret Sharing and T-PAKE in the Password-Only model
    target: https://eprint.iacr.org/2014/650
    date: false
    authors:
      -
        ins: S. Jarecki
        org: UC Irvine, CA, USA
      -
        ins: A. Kiayias
        org: University of Athens, Greece
      -
        ins: H. Krawczyk
        org: IBM Research, NY, USA
  SJKS17:
    title:  SPHINX, A Password Store that Perfectly Hides from Itself
    target: https://eprint.iacr.org/2018/695
    date: false
    authors:
      -
        ins: M. Shirvanian
        org: University of Alabama at Birmingham, USA
      -
        ins: S. Jarecki
        org: UC Irvine, CA, USA
      -
        ins: H. Krawczyk
        org: IBM Research, NY, USA
      -
        ins: N. Saxena
        org: University of Alabama at Birmingham, USA
  DGSTV18:
    title: Privacy Pass, Bypassing Internet Challenges Anonymously
    target: https://www.degruyter.com/view/j/popets.2018.2018.issue-3/popets-2018-0026/popets-2018-0026.xml
    date: false
    authors:
      -
        ins: A. Davidson
        org: RHUL, UK
      -
        ins: I. Goldberg
        org: University of Waterloo, Canada
      -
        ins: N. Sullivan
        org: Cloudflare, CA, USA
      -
        ins: G. Tankersley
        org: Independent
      -
        ins: F. Valsorda
        org: Independent
  SEC1:
    title: "SEC 1: Elliptic Curve Cryptography"
    target: https://www.secg.org/sec1-v2.pdf
    date: false
    author:
      -
        ins: Standards for Efficient Cryptography Group (SECG)
  SEC2:
    title: "SEC 2: Recommended Elliptic Curve Domain Parameters"
    target: http://www.secg.org/sec2-v2.pdf
    date: false
    author:
      -
        ins: Standards for Efficient Cryptography Group (SECG)
  x9.62:
    title: "Public Key Cryptography for the Financial Services Industry: the Elliptic Curve Digital Signature Algorithm (ECDSA)"
    date: Sep, 1998
    seriesinfo:
      "ANSI": X9.62-1998
    author:
      -
        org: ANSI

--- abstract

An Oblivious Pseudorandom Function (OPRF) is a two-party protocol for
computing the output of a PRF. One party (the server) holds the PRF
secret key, and the other (the client) holds the PRF input. The
'obliviousness' property ensures that the server does not learn anything
about the client's input during the evaluation. The client should also
not learn anything about the server's secret PRF key. Optionally, OPRFs
can also satisfy a notion 'verifiability' (VOPRF). In this setting, the
client can verify that the server's output is indeed the result of
evaluating the underlying PRF with just a public key. This document
specifies OPRF and VOPRF constructions instantiated within prime-order
groups, including elliptic curves.

--- middle

# Introduction

A pseudorandom function (PRF) F(k, x) is an efficiently computable
function taking a private key k and a value x as input. This function is
pseudorandom if the keyed function K(\_) = F(K, \_) is indistinguishable
from a randomly sampled function acting on the same domain and range as
K(). An oblivious PRF (OPRF) is a two-party protocol between a server
and a client, where the server holds a PRF key k and the client holds
some input x. The protocol allows both parties to cooperate in computing
F(k, x) such that: the client learns F(k, x) without learning anything
about k; and the server does not learn anything about x or F(k, x).
A Verifiable OPRF (VOPRF) is an OPRF wherein the server can prove to the
client that F(k, x) was computed using the key k.

The usage of OPRFs has been demonstrated in constructing a number of
applications: password-protected secret sharing schemes {{JKKX16}};
privacy-preserving password stores {{SJKS17}}; and
password-authenticated key exchange or PAKE {{!I-D.irtf-cfrg-opaque}}. A VOPRF is
necessary in some applications, e.g., the Privacy Pass protocol
{{!I-D.davidson-pp-protocol}}, wherein this VOPRF is used to generate
one-time authentication tokens to bypass CAPTCHA challenges. VOPRFs have
also been used for password-protected secret sharing schemes e.g.
{{JKK14}}.

This document introduces an OPRF protocol built in prime-order groups,
applying to finite fields of prime-order and also elliptic curve (EC)
groups. The protocol has the option of being extended to a VOPRF with
the addition of a NIZK proof for proving discrete log equality
relations. This proof demonstrates correctness of the computation, using
a known public key that serves as a commitment to the server's secret
key. The document describes the protocol, the public-facing API, and its
security properties.

## Change log

[draft-06](https://tools.ietf.org/html/draft-irtf-cfrg-voprf-06):

- Specify of group element and scalar serialization.
- Remove info parameter from the protocol API and update domain separation guidance.
- Fold Unblind function into Finalize.
- Optimize ComputeComposites for servers (using knowledge of the private key).
- Specify deterministic key generation method.
- Update test vectors.
- Apply various editorial changes.

[draft-05](https://tools.ietf.org/html/draft-irtf-cfrg-voprf-05):

- Move to ristretto255 and decaf448 ciphersuites.
- Clean up ciphersuite definitions.
- Pin domain separation tag construction to draft version.
- Move key generation outside of context construction functions.
- Editorial changes.

[draft-04](https://tools.ietf.org/html/draft-irtf-cfrg-voprf-04):

- Introduce Client and Server contexts for controlling verifiability and
  required functionality.
- Condense API.
- Remove batching from standard functionality (included as an extension)
- Add Curve25519 and P-256 ciphersuites for applications that prevent
  strong-DH oracle attacks.
- Provide explicit prime-order group API and instantiation advice for
  each ciphersuite.
- Proof-of-concept implementation in sage.
- Remove privacy considerations advice as this depends on applications.

[draft-03](https://tools.ietf.org/html/draft-irtf-cfrg-voprf-03):

- Certify public key during VerifiableFinalize.
- Remove protocol integration advice.
- Add text discussing how to perform domain separation.
- Drop OPRF_/VOPRF_ prefix from algorithm names.
- Make prime-order group assumption explicit.
- Changes to algorithms accepting batched inputs.
- Changes to construction of batched DLEQ proofs.
- Updated ciphersuites to be consistent with hash-to-curve and added
  OPRF specific ciphersuites.

[draft-02](https://tools.ietf.org/html/draft-irtf-cfrg-voprf-02):

- Added section discussing cryptographic security and static DH oracles.
- Updated batched proof algorithms.

[draft-01](https://tools.ietf.org/html/draft-irtf-cfrg-voprf-01):

- Updated ciphersuites to be in line with
  https://tools.ietf.org/html/draft-irtf-cfrg-hash-to-curve-04.
- Made some necessary modular reductions more explicit.

## Terminology {#terminology}

The following terms are used throughout this document.

- PRF: Pseudorandom Function.
- OPRF: Oblivious Pseudorandom Function.
- VOPRF: Verifiable Oblivious Pseudorandom Function.
- Client: Protocol initiator. Learns pseudorandom function evaluation as
  the output of the protocol.
- Server: Computes the pseudorandom function over a secret key. Learns
  nothing about the client's input.
- NIZK: Non-interactive zero knowledge.
- DLEQ: Discrete Logarithm Equality.

## Requirements

{::boilerplate bcp14}

# Preliminaries

The (V)OPRF protocol in this document has two primary dependencies:

- `GG`: A prime-order group implementing the API described below in {{pog}},
  with base point defined in the corresponding reference for each group.
  (See {{ciphersuites}} for these base points.)
- `Hash`: A cryptographic hash function that is indifferentiable from a
  Random Oracle, whose output length is Nh bytes long.

{{ciphersuites}} specifies ciphersuites as combinations of `GG` and `Hash`.

## Prime-Order Group Dependency {#pog}

In this document, we assume the construction of an additive, prime-order
group `GG` for performing all mathematical operations. Such groups are
uniquely determined by the choice of the prime `p` that defines the
order of the group. We use `GF(p)` to represent the finite field of
order `p`. For the purpose of understanding and implementing this
document, we take `GF(p)` to be equal to the set of integers defined by
`{0, 1, ..., p-1}`.

The fundamental group operation is addition `+` with identity element
`I`. For any elements `A` and `B` of the group `GG`, `A + B = B + A` is
also a member of `GG`. Also, for any `A` in `GG`, there exists an element
`-A` such that `A + (-A) = (-A) + A = I`. Scalar multiplication is
equivalent to the repeated application of the group operation on an
element A with itself `r-1` times, this is denoted as `r*A = A + ... + A`.
For any element `A`, `p*A=I`. We denote `G` as the fixed generator of
the group. Scalar base multiplication is equivalent to the repeated
application of the group operation `G` with itself `r-1` times, this
is denoted as `ScalarBaseMult(r)`. The set of scalars corresponds to
`GF(p)`.

We now detail a number of member functions that can be invoked on a
prime-order group `GG`.

- Order(): Outputs the order of `GG` (i.e. `p`).
- Identity(): Outputs the identity element of the group (i.e. `I`).
- HashToGroup(x): A member function of `GG` that deterministically maps
  an array of bytes `x` to an element of `GG`. The map must ensure that,
  for any adversary receiving `R = HashToGroup(x)`, it is
  computationally difficult to reverse the mapping. Examples of hash to
  group functions satisfying this property are described for prime-order
  (sub)groups of elliptic curves, see {{!I-D.irtf-cfrg-hash-to-curve}}.
- HashToScalar(x): A member function of `GG` that deterministically maps
  an array of bytes `x` to an element in GF(p). A recommended method
  for its implementation is instantiating the hash to field function,
  defined in {{!I-D.irtf-cfrg-hash-to-curve}} setting the target field to GF(p).
- RandomScalar(): A member function of `GG` that chooses at random a
  non-zero element in GF(p).
- SerializeElement(A): A member function of `GG` that maps a group element `A`
  to a unique byte array `buf` of fixed length `Ne`.
- DeserializeElement(buf): A member function of `GG` that maps a byte array
  `buf` to a group element `A`, or fails if the input is not a valid
  byte representation of an element.
- SerializeScalar(s): A member function of `GG` that maps a scalar element `s`
  to a unique byte array `buf` of fixed length `Ns`.
- DeserializeScalar(buf): A member function of `GG` that maps a byte array
  `buf` to a scalar `s`, or fails if the input is not a valid byte
  representation of a scalar.

Using the API of a prime-order group, we assume the existence of a function
`GenerateKeyPair()` that generates a random private and public key pair
(`skS`, `pkS`). One possible implementation might be to compute
`skS = RandomScalar()` and `pkS = ScalarBaseMult(skS)`. We also assume the
existence of a `DeriveKeyPair(seed)` function that deterministically generates
a private and public key pair from input `seed`, where `seed` is a random
byte string that SHOULD have at least `Ns` bytes of entropy.
`DeriveKeyPair(seed)` computes `skS = HashToScalar(seed)` and
`pkS = ScalarBaseMult(skS)`.

It is convenient in cryptographic applications to instantiate such
prime-order groups using elliptic curves, such as those detailed in
{{SEC2}}. For some choices of elliptic curves (e.g. those detailed in
{{RFC7748}}, which require accounting for cofactors) there are some
implementation issues that introduce inherent discrepancies between
standard prime-order groups and the elliptic curve instantiation. In
this document, all algorithms that we detail assume that the group is a
prime-order group, and this MUST be upheld by any implementer. That is,
any curve instantiation should be written such that any discrepancies
with a prime-order group instantiation are removed. See {{ciphersuites}}
for advice corresponding to the implementation of this interface for
specific definitions of elliptic curves.

## Other Conventions

- For any object `x`, we write `len(x)` to denote its length in bytes.
- For two byte arrays `x` and `y`, write `x || y` to denote their
  concatenation.
- I2OSP and OS2IP: Convert a byte array to and from a non-negative
  integer as described in {{!RFC8017}}. Note that these functions
  operate on byte arrays in big-endian byte order.

All algorithm descriptions are written in a Python-like pseudocode. We
use the `ABORT()` function for presentational clarity to denote the
process of terminating the algorithm or returning an error accordingly.
We also use the `CT_EQUAL(a, b)` function to represent constant-time
byte-wise equality between byte arrays `a` and `b`. This function
returns `true` if `a` and `b` are equal, and `false` otherwise.

# OPRF Protocol {#protocol}

In this section, we define two OPRF variants: a base mode and verifiable
mode. In the base mode, a client and server interact to compute y =
F(skS, x), where x is the client's input, skS is the server's private
key, and y is the OPRF output. The client learns y and the server learns
nothing. In the verifiable mode, the client also gets proof that the
server used skS in computing the function.

To achieve verifiability, as in the original work of {{JKK14}}, we
provide a zero-knowledge proof that the key provided as input by the
server in the `Evaluate` function is the same key as it used to produce
their public key. As an example of the nature of attacks that this
prevents, this ensures that the server uses the same private key for
computing the VOPRF output and does not attempt to "tag" individual
servers with select keys. This proof must not reveal the server's
long-term private key to the client.

The following one-byte values distinguish between these two modes:

| Mode           | Value |
|:===============|:======|
| modeBase       | 0x00  |
| modeVerifiable | 0x01  |

## Overview {#protocol-overview}

Both participants agree on the mode and a choice of ciphersuite that is
used before the protocol exchange. Once established, the core protocol
runs to compute `output = F(skS, input)` as follows:

~~~
   Client(pkS, input)                     Server(skS, pkS)
  ----------------------------------------------------------
    blind, blindedElement = Blind(input)

                       blindedElement
                        ---------->

    evaluatedElement, proof = Evaluate(skS, pkS, blindedElement)

                  evaluatedElement, proof
                        <----------

    output = Finalize(input, blind, evaluatedElement, blindedElement, pkS, proof)
~~~

In `Blind` the client generates a token and blinding data. The server
computes the (V)OPRF evaluation in `Evaluation` over the client's
blinded token. In `Finalize` the client unblinds the server response,
verifies the server's proof if verifiability is required, and produces
a byte array corresponding to the output of the OPRF protocol.

## Context Setup

Both modes of the OPRF involve an offline setup phase. In this phase,
both the client and server create a context used for executing the
online phase of the protocol. Prior to this phase, the key pair
(`skS`, `pkS`) should be generated by calling `GenerateKeyPair()`
or `DeriveKeyPair()` appropriately.

The base mode setup functions for creating client and server contexts are below:

~~~
def SetupBaseServer(suite, skS):
  contextString = "VOPRF06-" || I2OSP(modeBase, 1) || I2OSP(suite.ID, 2)
  return ServerContext(contextString, skS)

def SetupBaseClient(suite):
  contextString = "VOPRF06-" || I2OSP(modeBase, 1) || I2OSP(suite.ID, 2)
  return ClientContext(contextString)
~~~

The verifiable mode setup functions for creating client and server
contexts are below:

~~~
def SetupVerifiableServer(suite, skS, pkS):
  contextString = "VOPRF06-" || I2OSP(modeVerifiable, 1) || I2OSP(suite.ID, 2)
  return VerifiableServerContext(contextString, skS)

def SetupVerifiableClient(suite, pkS):
  contextString = "VOPRF06-" || I2OSP(modeVerifiable, 1) || I2OSP(suite.ID, 2)
  return VerifiableClientContext(contextString, pkS)
~~~

Each setup function takes a ciphersuite from the list defined in
{{ciphersuites}}. Each ciphersuite has a two-byte field ID used to
identify the suite.

[[RFC editor: please change "VOPRF06" to "RFCXXXX", where XXXX is the final number, here and elsewhere before publication.]]

## Data Types {#structs}

The following is a list of data structures that are defined for
providing inputs and outputs for each of the context interfaces defined
in {{api}}. Data structure description uses TLS notation (see {{?RFC8446}},
Section 3).

This document uses the types `Element` and `Scalar` to denote elements of the
group `GG` and its underlying scalar field `GF(p)`, respectively. For notational
clarity, `PublicKey` is an item of type `Element` and `PrivateKey` is an item
of type `Scalar`. `SerializedElement` and `SerializedScalar` are serialized
representations of `Element` and `Scalar` types of length `Ne` and `Ns`,
respectively; see {{pog}}. `ClientInput` is an opaque byte string of arbitrary
length. `Proof` is a sequence of two `SerializedScalar` elements, as shown below.

~~~
SerializedScalar Proof[2];
~~~

## Context APIs {#api}

In this section, we detail the APIs available on the client and server
(V)OPRF contexts. Each API has the following implicit parameters:

- GG, a prime-order group implementing the API described in {{pog}}.
- contextString, a domain separation tag taken from the client or server
  context.

### Server Context

The ServerContext encapsulates the context string constructed during
setup and the (V)OPRF key pair. It has three functions, `Evaluate`,
`FullEvaluate` and `VerifyFinalize` described below. `Evaluate` takes
serialized representations of blinded group elements from the client as inputs.

`FullEvaluate` takes ClientInput values, and it is useful for applications
that need to compute the whole OPRF protocol on the server side only.

`VerifyFinalize` takes ClientInput values and their corresponding output
digests from `Finalize` as input, and returns true if the inputs match the outputs.

Note that `VerifyFinalize` and `FullEvaluate` are not used in the main OPRF
protocol. They are exposed as an API for building higher-level protocols.

#### Evaluate

~~~
Input:

  PrivateKey skS
  SerializedElement blindedElement

Output:

  SerializedElement evaluatedElement

def Evaluate(skS, blindedElement):
  R = GG.DeserializeElement(blindedElement)
  Z = skS * R
  evaluatedElement = GG.SerializeElement(Z)

  return evaluatedElement
~~~

#### FullEvaluate

~~~
Input:

  PrivateKey skS
  ClientInput input

Output:

  opaque output[Nh]

def FullEvaluate(skS, input):
  P = GG.HashToGroup(input)
  T = skS * P
  issuedElement = GG.SerializeElement(T)

  finalizeDST = "Finalize-" || contextString
  hashInput = I2OSP(len(input), 2) || input ||
              I2OSP(len(issuedElement), 2) || issuedElement ||
              I2OSP(len(finalizeDST), 2) || finalizeDST

  return Hash(hashInput)
~~~

#### VerifyFinalize

~~~
Input:

  PrivateKey skS
  ClientInput input
  opaque output[Nh]

Output:

  boolean valid

def VerifyFinalize(skS, input, output):
  T = GG.HashToGroup(input)
  element = GG.SerializeElement(T)
  issuedElement = Evaluate(skS, [element])
  E = GG.SerializeElement(issuedElement)

  finalizeDST = "Finalize-" || contextString
  hashInput = I2OSP(len(input), 2) || input ||
              I2OSP(len(E), 2) || E ||
              I2OSP(len(finalizeDST), 2) || finalizeDST

  digest = Hash(hashInput)

  return CT_EQUAL(digest, output)
~~~

### VerifiableServerContext

The VerifiableServerContext extends the base ServerContext with an
augmented `Evaluate()` function. This function produces a proof that
`skS` was used in computing the result. It makes use of the helper
functions `GenerateProof` and `ComputeComposites`, described below.

#### Evaluate

~~~
Input:

  PrivateKey skS
  PublicKey pkS
  SerializedElement blindedElement

Output:

  SerializedElement evaluatedElement
  Proof proof

def Evaluate(skS, pkS, blindedElement):
  R = GG.DeserializeElement(blindedElement)
  Z = skS * R
  evaluatedElement = GG.SerializeElement(Z)

  proof = GenerateProof(skS, G, pkS, R, Z)

  return evaluatedElement, proof
~~~

The helper functions `GenerateProof` and `ComputeComposites` are defined
below.

#### GenerateProof

~~~
Input:

  Scalar k
  Element A
  Element B
  Element C
  Element D

Output:

  Proof proof

def GenerateProof(k, A, B, C, D)
  Cs = [C]
  Ds = [D]
  a = ComputeCompositesFast(k, B, Cs, Ds)

  r = GG.RandomScalar()
  M = a[0]
  Z = a[1]
  t2 = r * A
  t3 = r * M

  Bm = GG.SerializeElement(B)
  a0 = GG.SerializeElement(M)
  a1 = GG.SerializeElement(Z)
  a2 = GG.SerializeElement(t2)
  a3 = GG.SerializeElement(t3)

  challengeDST = "Challenge-" || contextString
  h2Input = I2OSP(len(Bm), 2) || Bm ||
            I2OSP(len(a0), 2) || a0 ||
            I2OSP(len(a1), 2) || a1 ||
            I2OSP(len(a2), 2) || a2 ||
            I2OSP(len(a3), 2) || a3 ||
            I2OSP(len(challengeDST), 2) || challengeDST

  c = GG.HashToScalar(h2Input)
  s = (r - c * k) mod p
  proof = [GG.SerializeScalar(c), GG.SerializeScalar(s)]

  return proof
~~~

##### Batching inputs

Unlike other functions, `ComputeComposites` takes lists of inputs,
rather than a single input. Applications can take advantage of this
functionality by invoking `GenerateProof` on batches of inputs to
produce a combined, constant-size proof. (In the pseudocode above,
the single inputs `blindedElement` and `evaluatedElement` are passed as
one-item lists to `ComputeComposites`.)

In particular, servers can produce a single, constant-sized proof for N
client inputs sent in a single request, rather than one proof per client
input. This optimization benefits clients and servers since it amortizes
the cost of proof generation and bandwidth across multiple requests.

##### Fresh Randomness

We note here that it is essential that a different `r` value is used for
every invocation. If this is not done, then this may leak `skS` as is
possible in Schnorr or (EC)DSA scenarios where fresh randomness is not
used.

#### ComputeComposites

The definition of `ComputeComposites` is given below. This function is
used both on generation and verification of the proof.

~~~
Input:

  Element B
  Element Cs[m]
  Element Ds[m]

Output:

  Element composites[2]

def ComputeComposites(B, Cs, Ds):
  Bm = GG.SerializeElement(B)
  seedDST = "Seed-" || contextString
  compositeDST = "Composite-" || contextString

  h1Input = I2OSP(len(Bm), 2) || Bm ||
            I2OSP(len(seedDST), 2) || seedDST
  seed = Hash(h1Input)

  M = GG.Identity()
  Z = GG.Identity()
  for i = 0 to m-1:
    Ci = GG.SerializeElement(Cs[i])
    Di = GG.SerializeElement(Ds[i])
    h2Input = I2OSP(len(seed), 2) || seed || I2OSP(i, 2) ||
              I2OSP(len(Ci), 2) || Ci ||
              I2OSP(len(Di), 2) || Di ||
              I2OSP(len(compositeDST), 2) || compositeDST
    di = GG.HashToScalar(h2Input)
    M = di * Cs[i] + M
    Z = di * Ds[i] + Z

 return [M, Z]
~~~

If the private key is known, as is the case for the server, this function
can be optimized as shown in `ComputeCompositesFast` below.

~~~
Input:

  Scalar k
  Element B
  Element Cs[m]
  Element Ds[m]

Output:

  Element composites[2]

def ComputeCompositesFast(k, B, Cs, Ds):
  Bm = GG.SerializeElement(B)
  seedDST = "Seed-" || contextString
  compositeDST = "Composite-" || contextString

  h1Input = I2OSP(len(Bm), 2) || Bm ||
            I2OSP(len(seedDST), 2) || seedDST
  seed = Hash(h1Input)

  M = GG.Identity()
  for i = 0 to m-1:
    Ci = GG.SerializeElement(Cs[i])
    Di = GG.SerializeElement(Ds[i])
    h2Input = I2OSP(len(seed), 2) || seed || I2OSP(i, 2) ||
              I2OSP(len(Ci), 2) || Ci ||
              I2OSP(len(Di), 2) || Di ||
              I2OSP(len(compositeDST), 2) || compositeDST
    di = GG.HashToScalar(h2Input)
    M = di * Cs[i] + M

  Z = k * M

 return [M, Z]
~~~

### Client Context

The ClientContext encapsulates the context string constructed during
setup. It has two functions, `Blind()` and `Finalize()`, as described
below. It also has an internal function, `Unblind()`, which is used
by `Finalize`. Its implementation varies depending on the mode.

#### Blind

We note here that the blinding mechanism that we use can be modified
slightly with the opportunity for making performance gains in some
scenarios. We detail these modifications in {{blinding}}.

~~~
Input:

  ClientInput input

Output:

  Scalar blind
  SerializedElement blindedElement

def Blind(input):
  blind = GG.RandomScalar()
  P = GG.HashToGroup(input)
  blindedElement = GG.SerializeElement(blind * P)

  return blind, blindedElement
~~~

#### Unblind

In this mode, `Unblind` takes only two inputs. The additional inputs indicated
in {{protocol-overview}} are only omitted as they are ignored. These additional
inputs are only useful for the verifiable mode, described in {{verifiable-unblind}}.

~~~
Input:

  Scalar blind
  SerializedElement evaluatedElement

Output:

  SerializedElement unblindedElement

def Unblind(blind, evaluatedElement, ...):
  Z = GG.DeserializeElement(evaluatedElement)
  N = (blind^(-1)) * Z
  unblindedElement = GG.SerializeElement(N)

  return unblindedElement
~~~

#### Finalize

`Finalize` depends on the internal `Unblind` function. In this mode, `Finalize`
and does not include all inputs listed in {{protocol-overview}}. These additional
inputs are only useful for the verifiable mode, described in {{verifiable-unblind}}.

~~~
Input:

  ClientInput input
  Scalar blind
  SerializedElement evaluatedElement

Output:

  opaque output[Nh]

def Finalize(input, blind, evaluatedElement):
  unblindedElement = Unblind(blind, evaluatedElement)

  finalizeDST = "Finalize-" || contextString
  hashInput = I2OSP(len(input), 2) || input ||
              I2OSP(len(unblindedElement), 2) || unblindedElement ||
              I2OSP(len(finalizeDST), 2) || finalizeDST
  return Hash(hashInput)
~~~

### VerifiableClientContext {#verifiable-client}

The VerifiableClientContext extends the base ClientContext with the
desired server public key `pkS` with an augmented `Unblind()` function.
This function verifies an evaluation proof using `pkS`. It makes use of
the helper function `ComputeComposites` described above. It has one
helper function, `VerifyProof()`, defined below.

#### VerifyProof

This algorithm outputs a boolean `verified` which indicates whether the
proof inside of the evaluation verifies correctly, or not.

~~~
Input:

  Element A
  Element B
  Element C
  Element D
  Proof proof

Output:

  boolean verified

def VerifyProof(A, B, C, D, proof):
  Cs = [C]
  Ds = [D]

  a = ComputeComposites(B, Cs, Ds)
  c = GG.DeserializeScalar(proof[0])
  s = GG.DeserializeScalar(proof[1])

  M = a[0]
  Z = a[1]
  t2 = ((s * A) + (c * B))
  t3 = ((s * M) + (c * Z))

  Bm = GG.SerializeElement(B)
  a0 = GG.SerializeElement(M)
  a1 = GG.SerializeElement(Z)
  a2 = GG.SerializeElement(t2)
  a3 = GG.SerializeElement(t3)

  challengeDST = "Challenge-" || contextString
  h2Input = I2OSP(len(Bm), 2) || Bm ||
            I2OSP(len(a0), 2) || a0 ||
            I2OSP(len(a1), 2) || a1 ||
            I2OSP(len(a2), 2) || a2 ||
            I2OSP(len(a3), 2) || a3 ||
            I2OSP(len(challengeDST), 2) || challengeDST

  expectedC  = GG.HashToScalar(h2Input)

  return CT_EQUAL(expectedC, c)
~~~

#### Verifiable Unblind {#verifiable-unblind}

~~~
Input:

  Scalar blind
  SerializedElement evaluatedElement
  SerializedElement blindedElement
  PublicKey pkS
  Scalar proof

Output:

  SerializedElement unblindedElement

def Unblind(blind, evaluatedElement, blindedElement, pkS, proof):
  Z = GG.DeserializeElement(evaluatedElement)
  R = GG.DeserializeElement(blindedElement)
  if VerifyProof(G, pkS, R, Z, proof) == false:
    ABORT()

  N = (blind^(-1)) * Z
  unblindedElement = GG.SerializeElement(N)

  return unblindedElement
~~~

#### Verifiable Finalize {#verifiable-finalize}

~~~
Input:

  ClientInput input
  Scalar blind
  SerializedElement evaluatedElement
  SerializedElement blindedElement
  PublicKey pkS
  Scalar proof

Output:

  opaque output[Nh]

def Finalize(input, blind, evaluatedElement, blindedElement, pkS, proof):
  unblindedElement = Unblind(blind, evaluatedElement, blindedElement, pkS, proof)

  finalizeDST = "Finalize-" || contextString
  hashInput = I2OSP(len(input), 2) || input ||
              I2OSP(len(unblindedElement), 2) || unblindedElement ||
              I2OSP(len(finalizeDST), 2) || finalizeDST
  return Hash(hashInput)
~~~

# Domain Separation {#domain-separation}

Applications SHOULD construct input to the protocol to provide domain
separation. Any system which has multiple (V)OPRF applications should
distinguish client inputs to ensure the OPRF results are separate.
Guidance for constructing info can be found in
{{!I-D.irtf-cfrg-hash-to-curve}}; Section 3.1.

# Ciphersuites {#ciphersuites}

A ciphersuite (also referred to as 'suite' in this document) for the protocol
wraps the functionality required for the protocol to take place. This
ciphersuite should be available to both the client and server, and agreement
on the specific instantiation is assumed throughout. A ciphersuite contains
instantiations of the following functionalities:

- `GG`: A prime-order group exposing the API detailed in {{pog}}, with base
  point defined in the corresponding reference for each group. Each group also
  specifies HashToGroup, HashToScalar, and serialization functionalities. For
  HashToGroup, the domain separation tag (DST) is constructed in accordance
  with the recommendations in {{!I-D.irtf-cfrg-hash-to-curve}}, Section 3.1.
- `Hash`: A cryptographic hash function that is indifferentiable from a
  Random Oracle, whose output length is Nh bytes long.

This section specifies ciphersuites with supported groups and hash functions.
For each ciphersuite, contextString is that which is computed in the Setup
functions.

Applications should take caution in using ciphersuites targeting P-256
and ristretto255. See {{cryptanalysis}} for related discussion.

## OPRF(ristretto255, SHA-512)

- Group: ristretto255 {{!RISTRETTO=I-D.irtf-cfrg-ristretto255-decaf448}}
  - HashToGroup(): Use hash_to_ristretto255
    {{!I-D.irtf-cfrg-hash-to-curve}} with DST =
    "HashToGroup-" || contextString, and `expand_message` = `expand_message_xmd`
    using SHA-512.
  - HashToScalar(): Compute `uniform_bytes` using `expand_message` = `expand_message_xmd`,
    DST = "HashToScalar-" || contextString, and output length 64, interpret
    `uniform_bytes` as a 512-bit integer in little-endian order, and reduce the integer
    modulo `Order()`.
  - Serialization: Both group elements and scalars are encoded in Ne = Ns = 32
    bytes. For group elements, use the 'Encode' and 'Decode' functions from
    {{!RISTRETTO}}. For scalars, ensure they are fully reduced modulo p and
    in little-endian order.
- Hash: SHA-512, and Nh = 64.
- ID: 0x0001

## OPRF(decaf448, SHA-512)

- Group: decaf448 {{!RISTRETTO}}
  - HashToGroup(): Use hash_to_decaf448
    {{!I-D.irtf-cfrg-hash-to-curve}} with DST =
    "HashToGroup-" || contextString, and `expand_message` = `expand_message_xmd`
    using SHA-512.
  - HashToScalar(): Compute `uniform_bytes` using `expand_message` = `expand_message_xmd`,
    DST = "HashToScalar-" || contextString, and output length 64, interpret
    `uniform_bytes` as a 512-bit integer in little-endian order, and reduce the integer
    modulo `Order()`.
  - Serialization: Both group elements and scalars are encoded in Ne = Ns = 56
    bytes. For group elements, use the 'Encode' and 'Decode' functions from
    {{!RISTRETTO}}. For scalars, ensure they are fully reduced modulo p and
    in little-endian order.
- Hash: SHA-512, and Nh = 64.
- ID: 0x0002

## OPRF(P-256, SHA-256)

- Group: P-256 (secp256r1) {{x9.62}}
  - HashToGroup(): Use hash_to_curve with suite P256_XMD:SHA-256_SSWU_RO\_
    {{!I-D.irtf-cfrg-hash-to-curve}} and DST =
    "HashToGroup-" || contextString.
  - HashToScalar(): Use hash_to_field from {{!I-D.irtf-cfrg-hash-to-curve}}
    using Order() as the prime modulus, L = 48, `expand_message_xmd`
    with SHA-256, and DST = "HashToScalar-" || contextString.
  - Serialization: Elements are serialized as Ne = 33 byte strings using
    compressed point encoding for the curve {{SEC1}}. Scalars are serialized as
    Ns = 32 byte strings by fully reducing the value modulo p and in big-endian
    order.
- Hash: SHA-256, and Nh = 32.
- ID: 0x0003

## OPRF(P-384, SHA-512)

- Group: P-384 (secp384r1) {{x9.62}}
  - HashToGroup(): Use hash_to_curve with suite P384_XMD:SHA-512_SSWU_RO\_
    {{!I-D.irtf-cfrg-hash-to-curve}} and DST =
    "HashToGroup-" || contextString.
  - HashToScalar(): Use hash_to_field from {{!I-D.irtf-cfrg-hash-to-curve}}
    using Order() as the prime modulus, L = 72, `expand_message_xmd`
    with SHA-512, and DST = "HashToScalar-" || contextString.
  - Serialization: Elements are serialized as Ne = 49 byte strings using
    compressed point encoding for the curve {{SEC1}}. Scalars are serialized as
    Ns = 48 byte strings by fully reducing the value modulo p and in big-endian
    order.
- Hash: SHA-512, and Nh = 64.
- ID: 0x0004

## OPRF(P-521, SHA-512)

- Group: P-521 (secp521r1) {{x9.62}}
  - HashToGroup(): Use hash_to_curve with suite P521_XMD:SHA-512_SSWU_RO\_
    {{!I-D.irtf-cfrg-hash-to-curve}} and DST =
    "HashToGroup-" || contextString.
  - HashToScalar(): Use hash_to_field from {{!I-D.irtf-cfrg-hash-to-curve}}
    using Order() as the prime modulus, L = 98, `expand_message_xmd`
    with SHA-512, and DST = "HashToScalar-" || contextString.
  - Serialization: Elements are serialized as Ne = 67 byte strings using
    compressed point encoding for the curve {{SEC1}}. Scalars are serialized as
    Ns = 66 byte strings by fully reducing the value modulo p and in big-endian
    order.
- Hash: SHA-512, and Nh = 64.
- ID: 0x0005

# Security Considerations {#sec}

This section discusses the cryptographic security of our protocol, along
with some suggestions and trade-offs that arise from the implementation
of an OPRF.

## Security Properties {#properties}

The security properties of an OPRF protocol with functionality y = F(k,
x) include those of a standard PRF. Specifically:

- Pseudorandomness: F is pseudorandom if the output y = F(k,x) on any
  input x is indistinguishable from uniformly sampling any element in
  F's range, for a random sampling of k.

In other words, consider an adversary that picks inputs x from the
domain of F and evaluates F on (k,x) (without knowledge of randomly
sampled k). Then the output distribution F(k,x) is indistinguishable
from the output distribution of a randomly chosen function with the same
domain and range.

A consequence of showing that a function is pseudorandom, is that it is
necessarily non-malleable (i.e. we cannot compute a new evaluation of F
from an existing evaluation). A genuinely random function will be
non-malleable with high probability, and so a pseudorandom function must
be non-malleable to maintain indistinguishability.

An OPRF protocol must also satisfy the following property:

- Oblivious: The server must learn nothing about the client's input or
  the output of the function. In addition, the client must learn nothing
  about the server's private key.

Essentially, obliviousness tells us that, even if the server learns the
client's input x at some point in the future, then the server will not
be able to link any particular OPRF evaluation to x. This property is
also known as unlinkability {{DGSTV18}}.

Optionally, for any protocol that satisfies the above properties, there
is an additional security property:

- Verifiable: The client must only complete execution of the protocol if
  it can successfully assert that the OPRF output it computes is
  correct. This is taken with respect to the OPRF key held by the
  server.

Any OPRF that satisfies the 'verifiable' security property is known as a
verifiable OPRF, or VOPRF for short. In practice, the notion of
verifiability requires that the server commits to the key before the
actual protocol execution takes place. Then the client verifies that the
server has used the key in the protocol using this commitment. In the
following, we may also refer to this commitment as a public key.

## Cryptographic Security {#cryptanalysis}

Below, we discuss the cryptographic security of the (V)OPRF protocol
from {{protocol}}, relative to the necessary cryptographic assumptions
that need to be made.

### Computational Hardness Assumptions {#assumptions}

Each assumption states that the problems specified below are
computationally difficult to solve in relation to a particular choice of
security parameter `sp`.

Let GG = GG(sp) be a group with prime-order p, and let GF(p) be a finite
field of order p.

#### Discrete-log (DL) Problem {#dl}

Given G, a generator of GG, and H = hG for some h in GF(p); output h.

#### Decisional Diffie-Hellman (DDH) Problem {#ddh}

Sample uniformly at random d in {0,1}. Given (G, aG, bG, C), where

- G is a generator of GG;
- a,b are elements of GF(p);
- if d == 0: C = abG; else: C is sampled uniformly at random from GG.

Output d' == d.

### Protocol Security {#protocol-sec}

Our OPRF construction is based on the VOPRF construction known as
2HashDH-NIZK given by {{JKK14}}; essentially without providing
zero-knowledge proofs that verify that the output is correct. Our VOPRF
construction is identical to the {{JKK14}} construction, except that we
can optionally perform multiple VOPRF evaluations in one go, whilst only
constructing one NIZK proof object. This is enabled using an established
batching technique.

Consequently, the cryptographic security of our construction is based on
the assumption that the One-More Gap DH is computationally difficult to
solve.

The (N,Q)-One-More Gap DH (OMDH) problem asks the following.

~~~
    Given:
    - G, k * G, and (G_1, ... , G_N), all elements of GG;
    - oracle access to an OPRF functionality using the key k;
    - oracle access to DDH solvers.

    Find Q+1 pairs of the form below:

    (G_{j_s}, k * G_{j_s})

    where the following conditions hold:
      - s is a number between 1 and Q+1;
      - j_s is a number between 1 and N for each s;
      - Q is the number of allowed queries.
~~~

The original paper {{JKK14}} gives a security proof that the
2HashDH-NIZK construction satisfies the security guarantees of a VOPRF
protocol {{properties}} under the OMDH assumption in the universal
composability (UC) security model.

### Q-Strong-DH Oracle {#qsdh}

A side-effect of our OPRF design is that it allows instantiation of a
oracle for constructing Q-strong-DH (Q-sDH) samples. The Q-Strong-DH
problem asks the following.

~~~
    Given G1, G2, h*G2, (h^2)*G2, ..., (h^Q)*G2; for G1 and G2
    generators of GG.

    Output ( (1/(k+c))*G1, c ) where c is an element of GF(p)
~~~

The assumption that this problem is hard was first introduced in
{{BB04}}. Since then, there have been a number of cryptanalytic studies
that have reduced the security of the assumption below that implied by
the group instantiation (for example, {{BG04}} and {{Cheon06}}). In
summary, the attacks reduce the security of the group instantiation by
log_2(Q) bits.

As an example, suppose that a group instantiation is used that provides
128 bits of security against discrete log cryptanalysis. Then an
adversary with access to a Q-sDH oracle and makes Q=2^20 queries can
reduce the security of the instantiation by log_2(2^20) = 20 bits.

Notice that it is easy to instantiate a Q-sDH oracle using the OPRF
functionality that we provide. A client can just submit sequential
queries of the form (G, k * G, (k^2)G, ..., (k^(Q-1))G), where each
query is the output of the previous interaction. This means that any
client that submits Q queries to the OPRF can use the aforementioned
attacks to reduce the security of the group instantiation by log_2(Q) bits.

Recall that from a malicious client's perspective, the adversary wins if
they can distinguish the OPRF interaction from a protocol that computes
the ideal functionality provided by the PRF.

### Implications for Ciphersuite Choices

The OPRF instantiations that we recommend in this document are informed
by the cryptanalytic discussion above. In particular, choosing elliptic
curves configurations that describe 128-bit group instantiations would
appear to in fact instantiate an OPRF with 128-log_2(Q) bits of
security.

In most cases, it would require an informed and persistent attacker to
launch a highly expensive attack to reduce security to anything much
below 100 bits of security. We see this possibility as something that
may result in problems in the future. For applications that cannot
tolerate discrete logarithm security of lower than 128 bits, we
recommend only implementing ciphersuites with IDs: 0x0002, 0x0004, and
0x0005.

## Hashing to Group

A critical requirement of implementing the prime-order group using
elliptic curves is a method to instantiate the function
`GG.HashToGroup`, that maps inputs to group elements. In the elliptic
curve setting, this deterministically maps inputs x (as byte arrays) to
uniformly chosen points on the curve.

In the security proof of the construction Hash is modeled as a random
oracle. This implies that any instantiation of `GG.HashToGroup` must be
pre-image and collision resistant. In {{ciphersuites}} we give
instantiations of this functionality based on the functions described in
{{!I-D.irtf-cfrg-hash-to-curve}}. Consequently, any OPRF implementation
must adhere to the implementation and security considerations discussed
in {{!I-D.irtf-cfrg-hash-to-curve}} when instantiating the function.

## Timing Leaks

To ensure no information is leaked during protocol execution, all
operations that use secret data MUST run in constant time. Operations that
SHOULD run in constant time include all prime-order group operations and
proof-specific operations (`GenerateProof()` and `VerifyProof()`).

## Key Rotation {#key-rotation}

Since the server's key is critical to security, the longer it is exposed
by performing (V)OPRF operations on client inputs, the longer it is
possible that the key can be compromised. For example, if the key is kept
in circulation for a long period of time, then it also allows the
clients to make enough queries to launch more powerful variants of the
Q-sDH attacks from {{qsdh}}.

To combat attacks of this nature, regular key rotation should be
employed on the server-side. A suitable key-cycle for a key used to
compute (V)OPRF evaluations would be between one week and six months.

# Additive Blinding {#blinding}

Let `H` refer to the function `GG.HashToGroup`, in {{pog}} we assume
that the client-side blinding is carried out directly on the output of
`H(x)`, i.e. computing `r * H(x)` for some `r` sampled uniformly at random
from `GF(p)`. In the {{!I-D.irtf-cfrg-opaque}} document, it is noted that it
may be more efficient to use additive blinding (rather than multiplicative)
if the client can preprocess some values. For example, a valid way of
computing additive blinding would be to instead compute `H(x) + (r * G)`,
where `G` is the fixed generator for the group `GG`.

The advantage of additive blinding is that it allows the client to
pre-process tables of blinded scalar multiplications for `G`. This may
give it a computational efficiency advantage (due to the fact that a
fixed-base multiplication can be calculated faster than a variable-base
multiplication). Pre-processing also reduces the amount of computation
that needs to be done in the online exchange. Choosing one of these
values `r * G` (where `r` is the scalar value that is used), then
computing `H(x) + (r * G)` is more efficient than computing `r * H(x)`.
Therefore, it may be advantageous to define the OPRF and VOPRF protocols
using additive (rather than multiplicative) blinding. In fact,
the only algorithms that need to change are `Blind` and `Unblind` (and
similarly for the VOPRF variants).

We define the variants of the algorithms in {{api}} for performing
additive blinding below, called `AdditiveBlind` and `AdditiveUnblind`,
along with a new algorithm `Preprocess`. The `Preprocess` algorithm can
take place offline and before the rest of the OPRF protocol. `AdditiveBlind`
takes the preprocessed values as inputs. `AdditiveUnblind` takes the
preprocessed values and evaluated element from the server as inputs.

## Preprocess

~~~
Input:

  PublicKey pkS

Output:

  Element blindedGenerator
  Element blindedPublicKey

def Preprocess(pkS):
  blind = GG.RandomScalar()
  blindedGenerator = ScalarBaseMult(blind)
  blindedPublicKey = blind * pkS

  return blindedGenerator, blindedPublicKey
~~~

## AdditiveBlind

~~~
Input:

  ClientInput input
  Element blindedGenerator

Output:

  SerializedElement blindedElement

def AdditiveBlind(input, blindedGenerator):
  P = GG.HashToGroup(input)
  blindedElement = GG.SerializeElement(P + blindedGenerator) /* P + ScalarBaseMult(r) */

  return blindedElement
~~~

## AdditiveUnblind

~~~
Input:

  Element blindedPublicKey
  SerializedElement evaluatedElement

Output:

 SerializedElement unblindedElement

def AdditiveUnblind(blindedPublicKey, evaluatedElement):
  Z = GG.DeserializeElement(evaluatedElement)
  N := Z - blindedPublicKey

  unblindedElement = GG.SerializeElement(N)

  return unblindedElement
~~~

Let `P = GG.HashToGroup(input)`. Notice that AdditiveUnblind computes:

~~~
Z - blindedPublicKey = k * (P + r * G) - r * pkS
                     = k * P + k * (r * G) - r * (k * G)
                     = k * P
~~~

by the commutativity of the scalar field. This is the same
output as in the `Unblind` algorithm for multiplicative blinding.

Note that the verifiable variant of `AdditiveUnblind` works as above but
includes the step to `VerifyProof`, as specified in {{verifiable-client}}.

### Parameter Commitments

For some applications, it may be desirable for the server to bind tokens to
certain parameters, e.g., protocol versions, ciphersuites, etc. To
accomplish this, the server should use a distinct scalar for each parameter
combination. Upon redemption of a token T from the client, the server can
later verify that T was generated using the scalar associated with the
corresponding parameters.

# Acknowledgements

This document resulted from the work of the Privacy Pass team
{{PrivacyPass}}. The authors would also like to acknowledge helpful
conversations with Hugo Krawczyk. Eli-Shaoul Khedouri provided
additional review and comments on key consistency. Daniel Bourdrez,
Tatiana Bradley, Sofía Celi, Frank Denis, and Bas Westerbaan also
provided helpful input and contributions to the document.

--- back

# Test Vectors

This section includes test vectors for the (V)OPRF protocol specified
in this document. For each ciphersuite specified in {{ciphersuites}},
there is a set of test vectors for the protocol when run in the base
mode and verifiable mode. Each test vector lists the batch size for
the evaluation. Each test vector value is encoded as a hexadecimal
byte string. The label for each test vector value is described below.

- "Input": The client input, an opaque byte string.
- "Blind": The blind value output by `Blind()`, a serialized `Scalar`
  of `Ns` bytes long.
- "BlindedElement": The blinded value output by `Blind()`, a serialized
  `Element` of `Ne` bytes long.
- "EvaluatedElement": The evaluated element output by `Evaluate()`,
  a serialized `Element` of `Ne` bytes long.
- "EvaluationProofC": The "c" component of the Evaluation proof (only
  listed for verifiable mode test vectors), a serialized `Scalar` of
  `Ns` bytes long.
- "EvaluationProofS": The "s" component of the Evaluation proof (only
  listed for verifiable mode test vectors), a serialized `Scalar` of
  `Ns` bytes long.
- "Output": The OPRF output, a byte string of length `Nh` bytes.

Test vectors with batch size B > 1 have inputs separated by a comma
",". Applicable test vectors will have B different values for the
"Input", "Blind", "BlindedElement", "EvaluationElement", and
"Output" fields.

The server key material, `pkSm` and `skSm`, are listed under the mode for
each ciphersuite. Both `pkSm` and `skSm` are the serialized values of
`pkS` and `skS`, respectively, as used in the protocol. Each key pair
is derived from a `seed`, which is listed as well.

## OPRF(ristretto255, SHA-512)

### Base Mode

~~~
seed = a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a
3a3
skSm = ca8b6aa354a5ab1411b883acd5608a3ab12ac687c93623274bcb579fd6506
b0c
~~~

#### Test Vector 1, Batch Size 1

~~~
Input = 00
Blind = c604c785ada70d77a5256ae21767de8c3304115237d262134f5e46e512cf
8e03
BlindedElement = a2b0b915f0404735ff6cf21f729e44d37123d0d1f5566e24207
7866d965d1913
EvaluationElement = 64f72c596342ab874077e8f98c721468c7332b1dc07a9595
f22a10875f05762a
Output = 4be865be03c4f560a641fec1cba70bda79891252c68f4ac65bff70e3f36
ee63bbd7009f3456f648c6d2944f8691bf6119574694e16bfc8104947d7b41882211
3
~~~

#### Test Vector 2, Batch Size 1

~~~
Input = 5a5a5a5a5a5a5a5a5a5a5a5a5a5a5a5a5a
Blind = 5ed895206bfc53316d307b23e46ecc6623afb3086da74189a416012be037
e50b
BlindedElement = aab54258dcf735c6decabb723ddb12cf50894332f19c503d0b3
b570a8e845235
EvaluationElement = 9a404e12c29070fc20943210c78278a1ea13fe59c842261d
0af113448eb3b747
Output = c0f6e64639df3ea0ddd3173422aa9170c244db220fedb544091b609e868
33eada85df7bb19c4add278b1a7c252bd505b9ccb3f1e1cb6d54cb5906eb1ac4c71a
8
~~~

### Verifiable Mode

~~~
seed = a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a
3a3
skSm = efd78e96dca2700086d815f1362a496ea5ed287756a8c043dc83479d7b8cf
907
pkSm = 6ca010a0a065ba492548a4ca08d0d4aa2f33e584b430bb8879712eb95668c
b00
~~~

#### Test Vector 1, Batch Size 1

~~~
Input = 00
Blind = ed8366feb6b1d05d1f46acb727061e43aadfafe9c10e5a64e7518d63e326
3503
BlindedElement = 2a841be9f7f476849893d3e48a40a9e87664fd1875e001fc581
79144f783d536
EvaluationElement = 2a2fe8b7a2178562da30aec8b4b803e7f9d83deacde1a2ef
3847f26c9d016347
EvaluationProofC = bf0c18848a399595f0600e35f19094d1142039a66624319a4
f95fb69e88c800a
EvaluationProofS = f2a45d5c704a71b475b7067cf14578b15d55ad941b6478e3e
5ff94bfc7538c01
Output = b9ed37f840e339872c56ccbe95b5f3bd515418500ad65a30630093a3ecc
e30b2ad85b9e815cc2056b0c168640e8d87c2ae0e47ed4f61b886bc3a699e2ce9d27
c
~~~

#### Test Vector 2, Batch Size 1

~~~
Input = 5a5a5a5a5a5a5a5a5a5a5a5a5a5a5a5a5a
Blind = e6d0f1d89ad552e383d6c6f4e8598cc3037d6e274d22da3089e7afbd4171
ea02
BlindedElement = 8c7cef545576473ebd47d6e241540531f6b1672486f0dda26b5
c41c24d7eba2d
EvaluationElement = ceb6b4c7037ff3fff4800aea833163ce1d68ea2d1fd12b5f
a34506c645054e18
EvaluationProofC = a1685f5b91d9c24dfa435deea96a4006a4aa117dd007d05b5
5c13a4c13c17b05
EvaluationProofS = 6aadc512cc835bb5426a4c4c44311a1b5852a65d73ae6632f
34b661d031e9906
Output = 8e2e8c97b685f0d425ae8658d798f67293092deb823242a9417ec846a8f
d7ffac89228b05ace8b65fe759686a1162e00829ef9e803634a296861dcb656f36a3
4
~~~

#### Test Vector 3, Batch Size 2

~~~
Input = 00,5a5a5a5a5a5a5a5a5a5a5a5a5a5a5a5a5a
Blind = 80513e77795feeec6d2c450589b0e1b178febd5c193a9fcba0d27f0a06e0
d50f,533c2e6d91c934f919ac218973be55ba0d7b234160a0d4cf3bddafbda99e2e0
c
BlindedElement = b80b7f73c89291a5a72e1c1c24054d96aa2ef7163465ef3bd9c
9a58907b92e44,4018acf2df5a491516a32f8fd5c0fbad92436e061ce86f02998179
4c6a1bba4b
EvaluationElement = 40cdf4e58d652bc4e6ee6eeafaa88b868a3bf5dc76493e5e
25f03578dc5b5857,fc812a7b3f068807146b59ef9f80542f3a19cf0c1f34e9e96ce
3cb8833100d7b
EvaluationProofC = 540f5c2ec83d6858ca0520ed662ff3122992e771670bbb2e4
bf4def36cc74e09
EvaluationProofS = 82132e1e158b30d180c54089bebedd7024051d3617b0480fc
fabc47582f7cf0e
Output = b9ed37f840e339872c56ccbe95b5f3bd515418500ad65a30630093a3ecc
e30b2ad85b9e815cc2056b0c168640e8d87c2ae0e47ed4f61b886bc3a699e2ce9d27
c,8e2e8c97b685f0d425ae8658d798f67293092deb823242a9417ec846a8fd7ffac8
9228b05ace8b65fe759686a1162e00829ef9e803634a296861dcb656f36a34
~~~

## OPRF(decaf448, SHA-512)

### Base Mode

~~~
seed = a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a
3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3
skSm = 1c776d9cea1190c8e36b574b78f89715197bc3af604f9935e6491c6b48b61
82aecb2a7b2e28de593e3cb3b35f6a4ecab99e5e4eca8a24a08
~~~

#### Test Vector 1, Batch Size 1

~~~
Input = 00
Blind = d1080372f0fcf8c5eace50914e7127f576725f215cc7c111673c635ce668
bbbb9b50601ad89b358ab8c23ed0b6c9d040365ec9d060868714
BlindedElement = dcd47f1697514034d9fc0d312d6049e018bd6bac9b3259dd399
b51df670300e13b3a5e74b2794cbbecfbc0b7c90482d8a6f4750c82a4af04
EvaluationElement = ecae5e32d4517da739ed0cff4b299e433d2a7f783729cd13
5ade609a82ca030b0d154719992b8222033295591d53c0a03ac66fdcd069e0a9
Output = 84304d7f48095c5aceb09911644d663fa9161b4bf49b50e648f8b09f91b
1c5746a0ed1d2a48990dc4e9c3be33603b5ef2c621518cf82c11919e34c918a76f84
7
~~~

#### Test Vector 2, Batch Size 1

~~~
Input = 5a5a5a5a5a5a5a5a5a5a5a5a5a5a5a5a5a
Blind = aed1ffa44fd8f0ed16373606a3cf7df589cca86d8ea1abbf5768771dbef3
d401c74ae55ba1e28b9565e1e4018eb261a14134a4ce60c1c718
BlindedElement = 28e25219cfdea084e0349e5249f842b6d117f9512191263e1d7
125ca740b31971ca99a2d346fec0b3c5afd29562ba02fb078dfac2d319e07
EvaluationElement = f03bf99457cf666af4271b748ebf632e763e25030aabb6c7
13db1a844b92a8dc8f6842ac164c07fd10432b8e23acb348a44e6248725ee457
Output = 91b830c8e8ca95d5143c02ec147506c6515b6bf51e45f26321cb9ee6581
242d9df876084a247a733be14ccf3dc16f0a92e6985f9e98976ab68436bd8e92ca9b
1
~~~

### Verifiable Mode

~~~
seed = a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a
3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3
skSm = eac0b641d6df0e27348bd3bfe81591f736ec321fcefdd345c5f8200c8338b
e3cc1bb094cd579f311060f8a3aae54457d18a86b7380952d20
pkSm = 9cdbd627f6b2e5fb2d35469d3d762d5044dbc5bc8f1067bf516d035c76227
11c990c584e127350b5db66eef098b707b34901cdfe68de3c52
~~~

#### Test Vector 1, Batch Size 1

~~~
Input = 00
Blind = 4c936db1779a621b6c71475ac3111fd5703a59b713929f36dfd1e892a7fe
814479c93d8b4b6e11d1f6fe5351e51457b665fa7b76074e531f
BlindedElement = e81533d1a9225aac867739832c5f2c0fe3bf4245e9fc261beee
b3804ec701e698a586c3aa88896f49901cebe03e170ba79edf0890b2ed668
EvaluationElement = 789257cd7b3b30115d941096a2ead8f4f1fa8e2d926c067b
175fa40aca074dc968bc56aa290f964b4bbaa7ba8fc245fd6aa4a9feea87e4e9
EvaluationProofC = 336e2a97ef380c3051c10eaffe100945d534f9987976a0f74
5f4f079d33a733710e2696a6477dfaaf9011843b2105ad4ed7e1efe9cae752e
EvaluationProofS = 1b524f1198e85d4b682d9d4335c9f2c7ee68bf6430971a018
d7af5a406122336f2fe49487a7b864d60f829b00d585678c443b40aec58990f
Output = d85d1dc1017b7e7f865ab526bd533e808f9d38969cd1d1e17394c344c8d
2b3fcfd5fc1fe1cc2d2abf087c047d455070e609bb156f5232b93fb5507a8de559c0
3
~~~

#### Test Vector 2, Batch Size 1

~~~
Input = 5a5a5a5a5a5a5a5a5a5a5a5a5a5a5a5a5a
Blind = 15b3355179392f40c3d5a15f0d5ffc354e340454ec779f575e4573a3886a
b5e57e4da2985cea9e32f6d95539ce2c7189e1bd7462e8c5483a
BlindedElement = b285637ffb0d97fa253c3efc1369a2c7ccb84c250f03a698c97
091036f33d9ec789867c0897e80cfdfc5a10abe9e8e8a5f9bd3636f2b27e5
EvaluationElement = 6ec28350fa245117edb450dcb3ec62b83515b7065aad30f1
6c690c19256577c72ce4945fd295abc77ff92a1959486e7290313b2dfebd0824
EvaluationProofC = edb482cc0eaf8c3b3c081c04a0ac4dcfa38ed4b949a3134a4
ea41656461de9e155bcb9a8b75554cfe15878825d2a579453b7b16127774f21
EvaluationProofS = 6582f9841ea5665d7833d458a2cb8a5aedc43df58087e5920
8d24194365fae89eb841ff918453bb357b61e28f6c18833638edebc30b1e022
Output = dfaf18c8c14dae6e442fe99fb0574b5ad895e826c157cade571a9542671
8be1fb23b3f0defddb9627c8723af69ee053fb9cdc236295bf01f361b4084a30518d
d
~~~

#### Test Vector 3, Batch Size 2

~~~
Input = 00,5a5a5a5a5a5a5a5a5a5a5a5a5a5a5a5a5a
Blind = 614bb578f29cc677ea9e7aea3e4839413997e020f9377b63c13584156a09
a46dd2a425c41eac0e313a47e99d05df72c6e1d58e6592577a0d,4c115060bca87db
7d73e00cbb8559f84cb7a221b235b0950a0ab553f03f10e1386abe954011b7da62bb
6599418ef90b5d4ea98cc28aff517
BlindedElement = 5eb7b615250eb20511c6fe4e3ad8f401f96acf64a8a8bee5cc8
deee97469d4730ff376675968ebd69f7bc0cda262057dddeb695aa7912954,8e1426
8f71e8abdf1df956a32b09110dfe6d83f4b89a2ba2a869f69adf675b661fc2dcaf36
dfcdf5223cdbd2d31ff3142d7cfd172a312239
EvaluationElement = 9c5be5c5342bd27f35b69b9d5f637e8711b2a5a9a3c456dc
16b8efc4f11c276dec8e08c763fb5084166e16e338214e78cf814e596caefd5a,f25
daaec5624137d831760dd5417b61afaef965c8f6367e663064b18d612821119b41c9
3123b215e0ff9dff56538decea44d686e1c9bd68e
EvaluationProofC = 203791f8214bda1e904b4eeb1cbf8ad5b5a0b9929384dd09a
f13838dcf31c3107c6fa6c107683c0b9f6615dac8fcafdb8baf0a8053cd8f3a
EvaluationProofS = d5289d3b168e4dd8e7dd9c66e63bfcb021ccc49285a7da2d6
611892e390115020d97af97bd090755bd0fc77e46f187cf9f362e52af26db25
Output = d85d1dc1017b7e7f865ab526bd533e808f9d38969cd1d1e17394c344c8d
2b3fcfd5fc1fe1cc2d2abf087c047d455070e609bb156f5232b93fb5507a8de559c0
3,dfaf18c8c14dae6e442fe99fb0574b5ad895e826c157cade571a95426718be1fb2
3b3f0defddb9627c8723af69ee053fb9cdc236295bf01f361b4084a30518dd
~~~

## OPRF(P-256, SHA-256)

### Base Mode

~~~
seed = a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a
3a3
skSm = 815a77cf92f333bdf6869bd9166bf4fb820160e0d9703236ac3e42b0e44b3
f11
~~~

#### Test Vector 1, Batch Size 1

~~~
Input = 00
Blind = 5d9e7f6efd3093c32ecceabd57fb03cf760c926d2a7bfa265babf29ec98a
f0d0
BlindedElement = 036fe34ec785f565c67188575fb85e45dcae1a01ecd7574839c
ff353eb3f9bc094
EvaluationElement = 02185a9448067c7ea4cb566288f52de635c73f85b94adb83
08e797fb5742c77e92
Output = 1dd9f4380834ce07f964b4a0e9407df2619403b5c0435e86842d46ff312
2e43f
~~~

#### Test Vector 2, Batch Size 1

~~~
Input = 5a5a5a5a5a5a5a5a5a5a5a5a5a5a5a5a5a
Blind = 825155ab61f17605af2ae2e935c78d857c9407bcd45128d57d338f1671b5
fcbe
BlindedElement = 0357489736f9da62cc104facdb909eb95e5cd556ca4a08b2793
732f5e3504c2c3c
EvaluationElement = 03ad96ee687c9a4d57b55f46698c9e2796acc78c05a0d0e2
2bc30e753f73f03b88
Output = fe345d391aead5bb83f01f048ba5d901f92271faf9c035d0baf5434a324
5eb1a
~~~

### Verifiable Mode

~~~
seed = a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a
3a3
skSm = 029ec0bf2e6153ece839125e15d76a9f054ce2945bfb57e32e39f43c0b5e3
280
pkSm = 029667382da3efb1d8df771ec1ca7f0dd598dda9bbd4c79b920c2d1e3dec2
20885
~~~

#### Test Vector 1, Batch Size 1

~~~
Input = 00
Blind = cee64d86fd20ab4caa264a26c0e3d42fb773b3173ba76f9588c9b14779bd
8d91
BlindedElement = 03351578a5acd9c1c7b939de59b600cb8198c8ec46c5e2cfa8a
c6861f0a5c25490
EvaluationElement = 034d809cef29116db99e8d8ed51ff00126a40debb0dbac31
9907042ea0fcdbd383
EvaluationProofC = 534fd3a99fc6a17a2d4007cfe51eb56354b949d82c797628a
3c04fe97f88b81f
EvaluationProofS = 64120fcfc300c3d402aa049fe4c1bb839163da54c21bce6e9
f055f704c3614af
Output = 847440cc811af42db433390156665727cd9d4b77bb5f780247be57b8a4f
db109
~~~

#### Test Vector 2, Batch Size 1

~~~
Input = 5a5a5a5a5a5a5a5a5a5a5a5a5a5a5a5a5a
Blind = 5c4b401063eff0bf242b4cd534a79bacfc2e715b2db1e7a3ad4ff8af1b24
daa2
BlindedElement = 02548fc2774c7f17b925002a290ea9e4099f1ac7851aaa66b50
d32c66b1b82102c
EvaluationElement = 03e1cfd84c5679496bd94a15ada49da764ab54fd02aced91
0db467a00e02148535
EvaluationProofC = bec34bf1db082165364a2f24b75483492432b4f4f7a3d2bc4
527f64a940201eb
EvaluationProofS = eb55ee0b51c0d93d63cc83f59b5102f20fb14f968cac8e3e9
351a90a281b3877
Output = ac42adf5facc79dd1ae9e93581d86fbd7d592a46ceee555008c62436cf8
5ed61
~~~

#### Test Vector 3, Batch Size 2

~~~
Input = 00,5a5a5a5a5a5a5a5a5a5a5a5a5a5a5a5a5a
Blind = f0c7822ba317fb5e86028c44b92bd3aedcf6744d388ca013ef33edd36930
4eda,3b9631be9f8b274d9aaf671bfb6a775229bf435021b89c683259773bc686956
b
BlindedElement = 02dc084fb8f639cb035cac90a4b3b5a40e3af937857160ae3ac
612f37de29b92c8,03cbcae5447ed1dbefcbf52edf18d0db934a20f6dcfb25c319f3
db98a00d4ca0c1
EvaluationElement = 03511ff88dde54885bf82101ffa64f09bc0c31c930c7de7f
54fd35d851b91846e4,03da54a39f37b3352ad143424a8aa4c371a753ebca847cf0e
8c71b466e884a7199
EvaluationProofC = 35be6a33623779c97d276021979d4b7550cb8d23d3708f6b2
a83f9a26ad86c1c
EvaluationProofS = 500614a8c6e389ae0dbb31e92ceaa123ebea7c4d184134946
c4beb2de8e04b28
Output = 847440cc811af42db433390156665727cd9d4b77bb5f780247be57b8a4f
db109,ac42adf5facc79dd1ae9e93581d86fbd7d592a46ceee555008c62436cf85ed
61
~~~

## OPRF(P-384, SHA-512)

### Base Mode

~~~
seed = a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a
3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3
skSm = 8d075563c385b0a7fe52fe36cec021c7fd338aa47f672b230b01248362669
06b3431e2cd99ae52b569707aeb86b7f2d7
~~~

#### Test Vector 1, Batch Size 1

~~~
Input = 00
Blind = 359073c015b92d15450f7fb395bf52c6ea98384c491fe4e4d423b59de7b0
df382902c13bdc9993d3717bda68fc080b99
BlindedElement = 0264775b50549d9e7aa07ecf0ac0dba04068352aeefc2086095
46cec329f74de77bcdbd84dc6cfcc0064e1fd0cc967e299
EvaluationElement = 025df8ac4c4d0b3477109263a671f73feaec0f07bbe09c8a
605a8903b600d95c0efd1e111fa00c343657d91a36d9b42c94
Output = 1eacc030ccdfd1d8fbc9cac4c794124056a013f6efa3621fc18ab52ddc5
bb440c63f8146a1db7266e1b04465eca212267f146beb248f551723537b9f6967501
9
~~~

#### Test Vector 2, Batch Size 1

~~~
Input = 5a5a5a5a5a5a5a5a5a5a5a5a5a5a5a5a5a
Blind = 21ece4f9b6ffd01ce82082545413bd9bb5e8f3c63b86ae88d9ce0530b01c
b1c23382c7ec9bdd6e75898e4877d8e2bc17
BlindedElement = 0382fa4dd4ccee8c7acb95d9db71298f830ab6ec16a4e07b411
6c5800b67e4687af57eb240b4d01b3efa1a0ad67889f3a6
EvaluationElement = 0234c7bed4271c83d17a399cdaefdd35bff967cdf84fda31
0eaedb15e2cf4337be68eeba23c80619ed83ab11aa691a51ab
Output = 569da7a3a183234f2ca5dc54145acd2450189161c93c9e6993fe9a913ba
dba1d80c5fcae2e6d466757ce294ea5571545b81ed5f80cc11ea7891a97069f26b8e
4
~~~

### Verifiable Mode

~~~
seed = a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a
3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3
skSm = 978dd82d645ae675237d855fb6cbb370573fa9e23328859777aaea11e2e25
eefce619f6d45f2eed992c4ec4660603dd8
pkSm = 03efdc97c396c9be4912e5171eec99cbf32342316f4e2eb0b955da6fdc9a4
15299cb13df4f9c8afd9d17e276beee3afc2a
~~~

#### Test Vector 1, Batch Size 1

~~~
Input = 00
Blind = 102f6338df84c9602bfa9e7d690b1f7a173d07e6d54a419db4a6308f8b09
589e4283efb9cd1ee4061c6bf884e60a8774
BlindedElement = 03adbf16df115a7efb99128ea11966de000c314d962aebea6cb
a0fdbd77edaa09fb43569fbabf17ec6e06e3673e8688fa2
EvaluationElement = 03c278ada58af2fae2c649977ef3f2851b384c7efbbe4e6e
66c5a3813b346e95a09ba782ca711ac97b6ee8479fd81ef596
EvaluationProofC = 7a137a342835c456a2833f7314ce860484a456c26b6eb93f7
6c28f108b40b29d75774a267b1b170932bebc016977a8ab
EvaluationProofS = d8dcdf7ee959802954005faff43ff4f3a1862b7de175cce98
0a8d7de2d890b4f495799c6a5d01b0a3f202a57020beb38
Output = fbeb018ae74510e4851d0bb7190ad25491ad62f180304c79c5c587c3f4b
05d4eafcdf03fa1e5c4518f14df71c9b4099e3251a40d59bf687ad971f94211a499a
c
~~~

#### Test Vector 2, Batch Size 1

~~~
Input = 5a5a5a5a5a5a5a5a5a5a5a5a5a5a5a5a5a
Blind = 8aec1d0c3d16afd032da7ba961449a56cec6fb918e932b06d5778ac7f67b
ecfb3e3869237f74106241777f230582e84a
BlindedElement = 02c066dce4710aa5982160a4640755404a4b660a1e1c35ace1a
c8b71d207b952cc9d0e739b489e1d901bd1c332643c07c4
EvaluationElement = 0223e312414d01fd49c450812713793d832335c53dda2c41
f0d7c913dce3ee7458a4ae4fcad4d85ad3d891e2074c216539
EvaluationProofC = 9c6ecd99051cf72e72a1467372d4297e743fd499a0af7d95d
622eb342e4dbfe98e0b693f276604c6969c8883da94ba36
EvaluationProofS = 6407181b2726166be08a3e2d8761284f9ef2f20a852434a1e
73fb39fcd6499c423404c7b69ef7d2e551256ae655bf628
Output = 9d2ca080f75bd14180e2ef39b172bd4ae5b935d6fcb83871fce36833d41
90c800460f62f2fe61e497aaa1c7870ef847d804dff38e6fd861d1377abc21a5b1b6
9
~~~

#### Test Vector 3, Batch Size 2

~~~
Input = 00,5a5a5a5a5a5a5a5a5a5a5a5a5a5a5a5a5a
Blind = 41fabd4722d92472d858051ce9ad1a533176a862c697b2c392aff2aeb77e
b20c2ae6ba52fe31e13e03bf1d9f39878b23,51171628f1d28bb7402ca4aea6465e2
67b7f977a1fb71593281099ef2625644aee0b6c5f5e6e01a2b052b3bd4caf539b
BlindedElement = 0239df2cd191ece93ee7fdc47eb5b07e5958b9d6634ed772e81
281a5a4950fd09cc3640f9ad41f1619716818c28887e81c,034c761b4299f569ef7e
bb42a552046127b6a99cbc358f4a0f0918f12c26445becefb4878c12b435bfe6b4f7
fbdeccd25f
EvaluationElement = 029ef67e3d67aca7fa1141c0923d695bdc9e562aa2acc217
a85d76f29a727a178d313ae91e693810a08ce740a5c3c98fae,034fcdf12b94df06f
0ec38bcca9a53619935e29dc7f7fa677a7a82ee6df773e11511edbe205d21b0377f9
4b2a20a88aea0
EvaluationProofC = 44e50e7dd50d64aefc4f5ecf7cb4570a77ecc082bd209ff92
bc41f6b3faebb249ba9fb7d13dc1a613ad4dcef68fb2163
EvaluationProofS = 3a12717749e31a607902ef076161643fdce1d78c2938d0fee
bae12d9b768c478f1f6b4f95a3120a0d31ced40dbeabad4
Output = fbeb018ae74510e4851d0bb7190ad25491ad62f180304c79c5c587c3f4b
05d4eafcdf03fa1e5c4518f14df71c9b4099e3251a40d59bf687ad971f94211a499a
c,9d2ca080f75bd14180e2ef39b172bd4ae5b935d6fcb83871fce36833d4190c8004
60f62f2fe61e497aaa1c7870ef847d804dff38e6fd861d1377abc21a5b1b69
~~~

## OPRF(P-521, SHA-512)

### Base Mode

~~~
seed = a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a
3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a
3a3
skSm = 004b6d726577fb03f271cc2019225dba2c95c30f3db9ccd3f7c73e72e5b88
dd414aae248363968cdec91eac95aca69d1bf76cda42213ed965a0218b0c9d4c9e70
639
~~~

#### Test Vector 1, Batch Size 1

~~~
Input = 00
Blind = 01b983705fcc9a39607288b935b0797ac6b3c4b2e848823ac9ae16b3a3b5
816be03432370deb7c3c17d9fc7cb4e0ce646e04e42d638e0fa7a434ed340772a8b5
d626
BlindedElement = 030050fc7061af80cb04426077ca987bea12b02e00e3b889b5e
57d399711d99e2a106083bb018ad9a7bc1207d402280e3ab2ee07551781a7263e931
608255550d1c0b7
EvaluationElement = 02003e638c43939c6372833e9cc58ff8f8d1b524c775f2be
4fee700316e9f16402ca62962d95efbbc17a85d340a811a0a64084c710c0a3e19e3e
ebb756e790a43e2c69
Output = 85ef5bc3cc7d6229fd45d276939834e8e78f680b5b9f602ae4a41bfa096
641621a0cb0ac4afdd09037a4a6d07846855c3124af5a1bdbc4f4345bf0381695710
1
~~~

#### Test Vector 2, Batch Size 1

~~~
Input = 5a5a5a5a5a5a5a5a5a5a5a5a5a5a5a5a5a
Blind = 01a03b1096b0316bc8567c89bd70267d35c8ddcb2be2cdc867089a2eb5cf
471b1e6eb4b043b9644c8539857abe3a2022e9c9fd6a1695bbabe8add48bcd149ff3
b841
BlindedElement = 0300f0b27a82d48b5f64f48a861c0a9d2387bfeb49e861e3eca
4e0ce2a2068585db53a00eea5a8925dedd1fdacc5caaa77fd30119eb4aa179704a62
0f637fa93aa1199
EvaluationElement = 0200b178d00273ef32bf1fa3725b172908dfe0b0a5449791
cb692a04934e9c0f92cd7f8bc74e02524eb3ea16ce86a06faf4104cf113a01f6e996
0f9baf0d786b2dead4
Output = c690a31df7904a928ddbfd431d4609dd1e81698276d57dfe9dd7334debb
75dd1153390c19145c54281099a8a18e00172b0cbe19a28501e698f35a30c514b855
0
~~~

### Verifiable Mode

~~~
seed = a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a
3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a3a
3a3
skSm = 01f26b83346e4b549ede2c8687c06de412cdb70d74574dd8e47711f583474
c1a0390b51e140b127a2442b3c97ed24c4d5ebc571c95552a31c37885e3d35294243
343
pkSm = 0301baa9fe02e50c9919c927a51d33d9a2546d37a61c1d3978c1a8e9370bf
6ef05670362c54c7a6579f243c9f21e9c27f6ae36c6eb05c5ff7aadd8a5b866a2204
a6e59
~~~

#### Test Vector 1, Batch Size 1

~~~
Input = 00
Blind = 00bbb82117c88bbd91b8954e16c0b9ceed3ce992b198be1ebfba9ba970db
d75beefbfc6d056b7f7ba1ef79f4facbf2d912c26ce2ecc5bb8d66419b379952e96b
d6f5
BlindedElement = 0301cefc78648a4adbb34d5818671ef4afd48d401cf4c6820f1
af958c720a470311375afa4b102a5ceccc232f2c33d7fffa19cec809563c21095306
32f592c88ce9ddf
EvaluationElement = 0200f8936e0ece051dcf24826c6cb0fbcfa8a682cdbe34a1
1b7ff1c68fc8c3168c6585fa3645285ffc997e91da1b029ce9300f3616ab3aa2b9c7
54e04c434c9f76543f
EvaluationProofC = 00b5d62e501182a16e02513a8dbd7222574cf76f4b4adc0ba
ac6ffc4e595b35e93d37c6fc52dbdbfea0876e23da47b304fd1c442a2260a460050d
a7b9fc2e0d2e213
EvaluationProofS = 01441af299dc1946a42a941acb829037bdbe75ae867aa46af
857773b29c7116f05bd244fd2ede1b9c7efdb151d6e74cb38fb21ff2cf7638ece3c9
75e114b591c872c
Output = b34400c99f49ce09dcb72bff46f5847fe168c7c738cf2887ed101acf9a9
259f9fc96e33418dbe235075500b5c304d3b553fbd2723610c0a2a1cc2bac5b47983
7
~~~

#### Test Vector 2, Batch Size 1

~~~
Input = 5a5a5a5a5a5a5a5a5a5a5a5a5a5a5a5a5a
Blind = 009055c99bf9591cb0eab2a72d044c05ca2cc2ef9b609a38546f74b6d688
f70cf205f782fa11a0d61b2f5a8a2a1143368327f3077c68a1545e9aafbba6a90dc0
d40a
BlindedElement = 0300a30fa0d5bc4981cf7b28cfea8329bb1eacf8f8ac6e79de3
9efdab7842b1ae428e3d5b396a2db3d12cc4259ef6127b6bd2fd4f352e87a05302c5
00548ca157fdf9a
EvaluationElement = 0201d10a84dd8adb70a835a6cc6b9ae53517f7dc9f7725c0
d5fe0311b87992597d795f4bd2c744525e2120102e26482dfae8055b05989a2aa124
d307a4823bb0173575
EvaluationProofC = 0079b52bc69ae3f3467b49d27bf938e679d3290c72a4540bc
66440ba0a479d2e5e3ccde5450c459cdc4fef0dbf37c9453b82d83280a3481bb3b82
b71724634a58414
EvaluationProofS = 0123d92d6ce3516524cd2ebf19fa92aff74c8cbade990b13d
6ec2d42d598adb9230045304446fad9e375d3e075f45b93a122f2ed8684a0d254b49
daa2c8a5906b8c7
Output = 2b5eb016c7dcd66b77826e2da167a69026adfc0927b6a3da869c4c73d3a
e7873f010ea8f1ad5f9f9a076928839ec25882cab7b66909a1187d9329fef71fd25b
c
~~~

#### Test Vector 3, Batch Size 2

~~~
Input = 00,5a5a5a5a5a5a5a5a5a5a5a5a5a5a5a5a5a
Blind = 01c6cf092d80c7cf2cb55388d899515238094c800bdd9c65f71780ba85f5
ae9b4703e17e559ca3ccd1944f9a70536c175f11a827452672b60d4e9f89eba28104
6e29,00cba1ba1a337759061965a423d9d3d6e1e1006dc8984ad28a4c93ecfc36fc2
171046b3c4284855cfa2434ed98db9e68a597db2c14728fade716a6a82d600444b26
e
BlindedElement = 0301fb42226d9b7857bdfc16c361541670b7fcbda3e7fbf8814
4594b658d3af94da54bd88b4f039a6e884b0e45a684fd2cdc5458d954cd77c648c95
485f792df0d9957,0201999e9f44430750033c96057447cd2266f42b92d674566f42
379df904357ca2c7ae3f25990601059494596539abfaf011516f50750fc61cb40fb5
f506d18a041df4
EvaluationElement = 02015cdec5d9884cf32625abc06f4507a0edea27e37e4d46
ac239875b4bef3c463bf2c4f7889ecbeba553437c08513d9fbc4796489f240fd6a72
e464048a84e070d143,0300802b98f06394ef0a757d097b64333fd82709d8d39bc1a
b4db1b583f1a1ce5d07fa092c6ff9a5585f364da62d435176080a4a957c3aabf7879
ec4598caa99fe2447
EvaluationProofC = 000fdc7eaca12bbcb142211090e6cf1f6382fc40f27bfeabb
db065f703cb4ba39b8468fe0094487ccddb1c313c1aea86bd8bddcf81f93db66def8
a7b8ccb1028cd4f
EvaluationProofS = 00edc6ab49eb6516e9e59d223d1d0bea1445c4b9ef1134e8a
00939db61fa1dec30fd7197f8995bb2b5ad7a885d2c5c93cabf134d374316543d0ba
7702ab74f91b404
Output = b34400c99f49ce09dcb72bff46f5847fe168c7c738cf2887ed101acf9a9
259f9fc96e33418dbe235075500b5c304d3b553fbd2723610c0a2a1cc2bac5b47983
7,2b5eb016c7dcd66b77826e2da167a69026adfc0927b6a3da869c4c73d3ae7873f0
10ea8f1ad5f9f9a076928839ec25882cab7b66909a1187d9329fef71fd25bc
~~~

