%%%
title = "JOSE and COSE Encoding for SPHINCS+"
abbrev = "jose-cose-sphincs-plus"
ipr= "trust200902"
area = "Internet"
workgroup = "COSE"
submissiontype = "IETF"
keyword = ["JOSE","COSE","PQC","SPHINCS+"]

[seriesInfo]
name = "Internet-Draft"
value = "draft-ietf-cose-sphincs-plus-latest"
stream = "IETF"
status = "standard"

[pi]
toc = "yes"

[[author]]
initials = "M."
surname = "Prorock"
fullname = "Michael Prorock"
organization = "mesur.io"
  [author.address]
  email = "mprorock@mesur.io"

[[author]]
initials = "O."
surname = "Steele"
fullname = "Orie Steele"
organization = "Transmute"
  [author.address]
  email = "orie@transmute.industries"

[[author]]
initials = "R."
surname = "Misoczki"
fullname = "Rafael Misoczki"
organization = "Google"
  [author.address]
  email = "rafaelmisoczki@google.com"

[[author]]
initials = "M"
surname = "Osborne"
fullname = "Michael Osborne"
organization = "IBM"
  [author.address]
  email = "osb@zurich.ibm.com"

[[author]]
initials = "C"
surname = "Cloostermans"
fullname = "Christine Cloostermans"
organization = "NXP"
  [author.address]
  email = "christine.cloostermans@nxp.com"

%%%

.# Abstract

This document describes JSON and CBOR serializations for SPHINCS+,
a Post-Quantum Cryptography (PQC) signature suite.

This document does not define any new cryptography, only seralizations
of existing cryptographic systems.

This document registers key types for JOSE and COSE, specifically `HASH`.

Key types in this document are specified by the cryptographic algorithm
family in use by a particular algorithm as discussed in RFC7517.

This document registers signature algorithms types for JOSE and COSE,
specifically `SPHINCS+256s` and others as required for use of various
parameterizations of the SPHINCS+ post-quantum signature scheme.

{mainmatter}

# Notational Conventions

The key words "**MUST**", "**MUST NOT**", "**REQUIRED**", "**SHALL**", "**SHALL NOT**", "**SHOULD**",
"**SHOULD NOT**", "**RECOMMENDED**", "**MAY**", and "**OPTIONAL**" in this
document are to be interpreted as described in [@!RFC2119].

# Terminology

The following terminology is used throughout this document:

PK : The public key for the signature scheme.

SK : The secret key for the signature scheme.

signature : The digital signature output.

message : The input to be signed by the signature scheme.

sha256 : The SHA-256 hash function defined in [@RFC6234].

shake256 : The SHAKE256 hash function defined in [@!RFC8702].

# SPHINCS-PLUS

This section defines core operations used by the signature scheme, as
proposed in [@!SPHINCS-PLUS].

## Overview

This section of the document describes the hash-based signature scheme
SPHINCS+. The scheme is based on the concept of authenticating a large
number or few-time signatures keypair using a combination of Merkle-tree
signatures, a so-called hypertree. For each message to be signed a
(pseudo-)random FTS keypair is selected with which the message can be
signed. Combining this signature along with an authentication path
through the hyper-tree consisting of hash-based many-time signatures
then gives the SPHINC+ signature. The parameter set is strategically
chosen such that the probability of signing too many messages with a
specific FTS keypair to impact security is small enough to prevent
forgery attacks. A trade-off in parameter set can be made on security
guarantees, performance and signature size.

SPHINCS+ is a post-quantum approach to digital signatures that is
promises Post-Quantum Existential Unforgeability under Chosen Message
Attack (PQ-EU-CMA), while ensuring that the security levels reached meet
security needs for resistance to both classical and quantum attacks. The
algoritm itself is based on the hardness assumptions of its underlying
hash functions, which can be chosen from the set Haraka, SHA-256 or
SHAKE256. For all security levels the only operations required are calls
to these hash functions on various combinations of parameters and
internal states.

Contrary to CRYSTALS-Dilithium and Falcon, SPHINCS+ is not based on any
algebraic structure. This reduces the possible attack surface of the
algorithm.

SPHINCS+ brings several advantages over other approaches to signature
suites:

- Post-quantum in nature - use of cryptographically secure hash
  functions and other approaches that should remain hard problems even
  when under an attack utilizing quantum approaches
- Minimal security assumptions - compared to other schemes does not base
  its security on a new paradigm. The security is solely based on the
  security of the assumptions of the underlying hash function.
- Performance and Optimization - based on combining a great many hash
  function calls of SHA-256, SHAKE256 or Haraka means existing (secure)
  SW and HW implementations of those hash functions can be re-used for
  increased performance
- Private and Public Key Size - compared to other post-quantum
  approaches a very small key size is the form of hash inputs-outputs.
  This then has the drawback that either a large signature or low
  signing speed has to be accepted
- Cryptanalysis assuarance - attacks (both pre-quantum and quantum) are
  easy to relate to existing attacks on hash functions. This allows for
  precise quantification of the security levels
- Overlap with stateful hash-based algorithms - means there are
  possibilities to combine implementions with those of XMSS and LMS. For
  example, both have the same underlying hash functions and utilize
  existing HW acceleration. Furthermore, an API to a XMSS implementation
  can be directly used by the subroutines of Sphincs+
- Inherent resistance against side-channel attacks - since its core
  primitive is a hash function, it thereby is hard to attack with
  side-channels.

The primary known disadvantage to SPHINCS+ is the size signatures, or
the speed of signing, depending on the chosen parameter set. Especially
in IoT applications this might pose a problem. Additionally hash-based
schemes are also vulnerable to differential and fault attacks.

## Core Operations

Core operations used by the signature scheme should be implemented
according to the details in [@!SPHINCS-PLUS]. Core operations include
key generation, sign, and verify.

## Using SPHINCS-PLUS with JOSE

This sections is based on [CBOR Object Signing and Encryption (COSE) and
JSON Object Signing and Encryption
(JOSE)](https://datatracker.ietf.org/doc/html/rfc8812#section-3)

### SPHINCS-PLUS Key Representations

A new key type (kty) value "HASH" (for keys related to the family of
algorithms that utilize hash based approaches to post-quantum
cryptography) is defined for public key algorithms that use base 64
encoded strings of the underlying binary material as private and public
keys and that support cryptographic sponge functions. It has the
following parameters:

- The parameter "kty" MUST be "HASH".

- The parameter "alg" MUST be specified, and its value MUST be one of
  the values specified the below table

| alg         | Description                          |
| ----------- | ------------------------------------ |
| SPHINCS+128s | SPHINCS+ with parameter set of 128s |
| SPHINCS+128f | SPHINCS+ with parameter set of 128f |
| SPHINCS+192s | SPHINCS+ with parameter set of 192s |
| SPHINCS+192f | SPHINCS+ with parameter set of 192f |
| SPHINCS+256s | SPHINCS+ with parameter set of 256s |
| SPHINCS+256f | SPHINCS+ with parameter set of 256f |

- The parameter "pset" MAY be specfied to indicate the paramter set in
  use for the algorithm, but SHOULD also reflect the targeted NIST level
  for the algorithm in combination with the specified paramter set. For
  "alg" "HAS" one of the described parameter sets as listed in the
  section SPHINCS+ Algorithms MUST be specified.

- The parameter "x" MUST be present and contain the public key encoded
  using the base64url [@!RFC4648] encoding.

- The parameter "d" MUST be present for private keys and contain the
  private key encoded using the base64url encoding. This parameter MUST
  NOT be present for public keys.

When calculating JWK Thumbprints [@!RFC7638], the four public key fields
are included in the hash input in lexicographic order: "kty", "alg", and
"x".

When using a JWK for this algorithm, the following checks are made:

- The "kty" field MUST be present, and it MUST be "HASH" for JOSE.

- The "alg" field MUST be present, and it MUST represent the algorith
  and parameter set.

- If the "key_ops" field is present, it MUST include "sign" when
  creating a HASH signature.

- If the "key_ops" field is present, it MUST include "verify" when
  verifying a HASH signature.

- If the JWK "use" field is present, its value MUST be "sig".

### SPHINCS-PLUS Algorithms

In order to reduce the complexity of the key representation and
signature representations we register a unique algorithm name per pset.
This allows us to omit registering the `pset` term, and reduced the
likelyhood that it will be misused. These `alg` values are used in both
key representations and signatures.

Sphincs+ targets different security levels (128-, 192- and 256-bit
security) and tradeoffs between size and speed. For each security level
a small (s) and fast (f) parameter set is provided.

| kty         | alg           | Paramter Set |
| ----------- | ------------- | ------------ |
| HASH        | SPHINCS+128s  | 128s         |
| HASH        | SPHINCS+128f  | 128f         |
| HASH        | SPHINCS+192s  | 192s         |
| HASH        | SPHINCS+192f  | 192f         |
| HASH        | SPHINCS+256s  | 256s         |
| HASH        | SPHINCS+256f  | 256f         |

## Using SPHINCS-PLUS with COSE

The approach taken here matches the work done to support secp256k1 in
JOSE and COSE in [@!RFC8812].

The following tables map terms between JOSE and COSE for signatures.

| Name            | Value | Description                      | Recommended |
| --------------- | ----- | -------------------------------- | ----------- |
| SPHINCS+128s    | TBD   | SPHINCS+ with parameter set 128s | No          |
| SPHINCS+128f    | TBD   | SPHINCS+ with parameter set 128f | No          |
| SPHINCS+192s    | TBD   | SPHINCS+ with parameter set 192s | No          |
| SPHINCS+192f    | TBD   | SPHINCS+ with parameter set 192f | No          |
| SPHINCS+256s    | TBD   | SPHINCS+ with parameter set 256s | No          |
| SPHINCS+256f    | TBD   | SPHINCS+ with parameter set 256f | No          |

The following tables map terms between JOSE and COSE for key types.

| Name       | Value | Description                          | Recommended |
| ---------- | ----- | ------------------------------------ | ----------- |
| HASH       | TBD   | kty for hash based digital signature | No          |

# Security Considerations

The following considerations SHOULD apply to all parmeter sets described
in this specification, unless otherwise noted.

Care should be taken to ensure "kty" and intended use match, the
algorithms described in this document share many properties with other
cryptographic approaches from related families that are used for
purposes other than digital signatures.

## Validating public keys

All algorithms in that operate on public keys require first validating
those keys. For the sign, verify and proof schemes, the use of
KeyValidate is REQUIRED.

## Side channel attacks

Implementations of the signing algorithm SHOULD protect the secret key
from side-channel attacks. Multiple best practices exist to protect
against side-channel attacks. Any implementation of the the Sphincs+
signing algorithms SHOULD utilize the following best practices at a
minimum:

- Constant timing - the implementation should ensure that constant time
  is utilized in operations
- Sequence and memory access persistance - the implemention SHOULD
  execute the exact same sequence of instructions (at a machine level)
  with the exact same memory access independent of which polynomial is
  being operated on.
- Uniform sampling - care should be given in implementations to preserve
  the property of uniform sampling in implementation and to prevent
  information leakage.

## Randomness considerations

It is recommended that the all nonces are from a trusted source of
randomness.

# IANA Considerations

The following has NOT YET been added to the "JSON Web Key Types"
registry:

- Name: "HASH"
- Description: Hash based post-quantum signature algorithm key pairs
- JOSE Implementation Requirements: Optional
- Change Controller: IESG
- Specification Document(s): Section 3.1 of this document (TBD)

The following has NOT YET been added to the "JSON Web Key Parameters"
registry:

- Parameter Name: "pset"
- Parameter Description: The parameter set of the crypto system
- Parameter Information Class: Public
- Used with "kty" Value(s): "HASH"
- Change Controller: IESG
- Specification Document(s): Section 2 of this document (TBD)

The following has NOT YET been added to the "JSON Web Key Parameters"
registry:

- Parameter Name: "d"
- Parameter Description: The private key
- Parameter Information Class: Private
- Used with "kty" Value(s): "HASH"
- Change Controller: IESG
- Specification Document(s): Section 2 of RFC 8037

The following has NOT YET been added to the "JSON Web Key Parameters"
registry:

- Parameter Name: "x"
- Parameter Description: The public key
- Parameter Information Class: Public
- Used with "kty" Value(s): "HASH"
- Change Controller: IESG
- Specification Document(s): Section 2 of RFC 8037

The following has NOT YET been added to the "JSON Web Signature and
Encryption Algorithms" registry:

- Algorithm Name: "SPHINCS+128s"
- Algorithm Description: SPHINCS+128s signature algorithms
- Algorithm Usage Location(s): "alg"
- JOSE Implementation Requirements: Optional
- Change Controller: IESG
- Specification Document(s): Section 5.1 of this document (TBD)
- Algorithm Analysis Documents(s): (TBD)

The following has NOT YET been added to the "JSON Web Signature and
Encryption Algorithms" registry:

- Algorithm Name: "SPHINCS+128f"
- Algorithm Description: SPHINCS+128f signature algorithms
- Algorithm Usage Location(s): "alg"
- JOSE Implementation Requirements: Optional
- Change Controller: IESG
- Specification Document(s): Section 5.1 of this document (TBD)
- Algorithm Analysis Documents(s): (TBD)

The following has NOT YET been added to the "JSON Web Signature and
Encryption Algorithms" registry:

- Algorithm Name: "SPHINCS+192s"
- Algorithm Description: SPHINCS+192s signature algorithms
- Algorithm Usage Location(s): "alg"
- JOSE Implementation Requirements: Optional
- Change Controller: IESG
- Specification Document(s): Section 5.1 of this document (TBD)
- Algorithm Analysis Documents(s): (TBD)

The following has NOT YET been added to the "JSON Web Signature and
Encryption Algorithms" registry:

- Algorithm Name: "SPHINCS+192f"
- Algorithm Description: SPHINCS+192f signature algorithms
- Algorithm Usage Location(s): "alg"
- JOSE Implementation Requirements: Optional
- Change Controller: IESG
- Specification Document(s): Section 5.1 of this document (TBD)
- Algorithm Analysis Documents(s): (TBD)

The following has NOT YET been added to the "JSON Web Signature and
Encryption Algorithms" registry:

- Algorithm Name: "SPHINCS+256s"
- Algorithm Description: SPHINCS+256s signature algorithms
- Algorithm Usage Location(s): "alg"
- JOSE Implementation Requirements: Optional
- Change Controller: IESG
- Specification Document(s): Section 5.1 of this document (TBD)
- Algorithm Analysis Documents(s): (TBD)

The following has NOT YET been added to the "JSON Web Signature and
Encryption Algorithms" registry:

- Algorithm Name: "SPHINCS+256f"
- Algorithm Description: SPHINCS+256f signature algorithms
- Algorithm Usage Location(s): "alg"
- JOSE Implementation Requirements: Optional
- Change Controller: IESG
- Specification Document(s): Section 5.1 of this document (TBD)
- Algorithm Analysis Documents(s): (TBD)

# Appendix

- JSON Web Signature (JWS) - [RFC7515][spec-jws]
- JSON Web Encryption (JWE) - [RFC7516][spec-jwe]
- JSON Web Key (JWK) - [RFC7517][spec-jwk]
- JSON Web Algorithms (JWA) - [RFC7518][spec-jwa]
- JSON Web Token (JWT) - [RFC7519][spec-jwt]
- JSON Web Key Thumbprint - [RFC7638][spec-thumbprint]
- JWS Unencoded Payload Option - [RFC7797][spec-b64]
- CFRG Elliptic Curve ECDH and Signatures - [RFC8037][spec-okp]
- SPHINCS+ - [SPHINCS-PLUS][spec-sphincs-plus]

[RFC2119]: https://tools.ietf.org/html/rfc2119
[spec-b64]: https://tools.ietf.org/html/rfc7797
[spec-cookbook]: https://tools.ietf.org/html/rfc7520
[spec-jwa]: https://tools.ietf.org/html/rfc7518
[spec-jwe]: https://tools.ietf.org/html/rfc7516
[spec-jwk]: https://tools.ietf.org/html/rfc7517
[spec-jws]: https://tools.ietf.org/html/rfc7515
[spec-jwt]: https://tools.ietf.org/html/rfc7519
[spec-okp]: https://tools.ietf.org/html/rfc8037
[spec-secp256k1]: https://tools.ietf.org/html/rfc8812
[spec-thumbprint]: https://tools.ietf.org/html/rfc7638
[spec-sphincs-plus]:
    https://sphincs.org/data/sphincs+-round3-specification.pdf

<reference anchor='SPHINCS-PLUS' target='https://sphincs.org'> <front>
    <title>Sphincs+ Stateless Hash-based Signatures</title> <author
        initials='A.' surname='Hulsing' fullname='Andreas Hulsing'>
        <organization>Eindhoven University of Technology
         (NL)</organization> </author> <date year='2017'/> </front>
        </reference>


## Test Vectors

### HASH SPHINCS+256s

#### publicKeyJwk
```json
{"kty":"HASH","alg":"SPHINCS+256s","x":"C_NSiMVMN6kpAv\
\1O21izzVkl7cN0ls-tX3xL8VeIOHWzfwmMJ37LvTLVXZQMFyXFTilBcLjcbPNqRCMXi\
\z_c1A"}
```

#### privateKeyJwk
```json
{"kty":"HASH","alg":"SPHINCS+256s","x":"C_NSiMVMN6kpAv\
\1O21izzVkl7cN0ls-tX3xL8VeIOHWzfwmMJ37LvTLVXZQMFyXFTilBcLjcbPNqRCMXi\
\z_c1A","d":"-gEaHimlK26FRDpf33I6BKsfT3muN8xdOyonuYSgHtEoDxDBsTe30nU\
\O8OWZrfwJxvKyVxo6HCThjbMJX9LqSQvzUojFTDepKQL9TttYs81ZJe3DdJbPrV98S_\
\FXiDh1s38JjCd-y70y1V2UDBclxU4pQXC43GzzakQjF4s_3NQ"}
```

#### jws
```jws
eyJhbGciOiJTUEhJTkNTKy1TSEFLRS0yNTZzLXJvYnVzdCJ9.OTA4NWQyYmVmNjkyODZ\
\hNmNiYjUxNjIzYzhmYTI1ODYyOTk0NWNkNTVjYTcwNWNjNGU2NjcwMDM5Njg5NGUwYw\
\.YHQAElZyZybZFn_t59KRVgl0AjA5g-Jp10exC8JGFsv8kMeyWoroy-T5ANx2lkE-cz\
\jnnGMZdyODF_-ykpLqGisuXJwaHjZtelsN9-XwzAHp7wUR7GMw3Y0YLlNWfRSSz7cq6\
\Vx9IfzmVgfc3LqG5OKJZQ0LIjliQGEoKiIm4dDZDurJgN7XJACWBFTd0Ashktv5IC_n\
\JE8DGqDOoPuqYVciWmnD9ljFZKVniEM1PcFAsuQZ66OfPsEF-0ZXSMvWyHwcznw50o6\
\RPQXlbwF0IYyOrhBitNtoKDZ3f-hJ7NleYVLbuztFxw1ewf7iEA1oxRQq6fiE2J7YWf\
\qQO67KdeH2f5jzsiwrKUqodc43emKyTlW6_ckqaYfrtSp1JN6JnoE5vbKFeGD08fCW9\
\02MvGXSJKHwj8dRi8D4-JuGwJ6u04XGj3y5sBc0VnwMAFJ7nSemX1uRzr1teFp_WhRy\
\z2u6xpMhNJudKi0aVgE2Yk3U7ae3yabwQqi5euxLCbHVYHrr5XfJHgRTuiZcHAxERMf\
\cpy6lEfHHtp6wQJWJz56mtovuhO8a-H7RWjAUnIBy-I7lokJyHbFAisR730qsVDgsEF\
\Qv6CyFEcB3zvzb0C2uPXzlKOnc8tu1hSBt4fGV1k9DOJ8Fy6YFZM780O8i0bSWdel4-\
\F4lV7ZWfYhPSfiCYI1nasM6ghQj1DhWRY1izr9NHcNxS2aQmQ4BfDtfooYIsvRidfKz\
\V8QG3p7aCoLUexS8qiQfZ86U3ROjgStSH1FZlgQ67Dj1pUifCTlqfdoxaosE8tkYD8t\
\IO-zYwZ2uGvWbqHevCbV2GGwJUmnjL5zkMh8ilNbDGaK2ZJ4s-3cu0Q-NY6e9ot-IHT\
\1whbSzWNUpvEVNd4780rRKcGRtydP-vJ8yOCqvSecWxvnJH02ib9JzymIEtUF0mkdpn\
\Wk8KzUpX79C0snMvj0KeKp3T6aK7xZK8pxbHoqFzXPD_Jl8ffUDVbQ-QQOT1aMqfQyM\
\P7vn2xLdYCgBKAch8AY_UxZR1QnnfWXV70L4GenM-xW8fo6kKrieqhb7dnId4jZyjot\
\KNaEppowLnG-nw99EH4zrR5MLOYavvQwxi2XOOfuPeq_jjZC0IgpliRIVoysR851W80\
\olsvCOFY-6dWrvSQ5bPttFnfXTDkjmOpZPDUt72F2JCjb8f_Cp15s7tFlQ0RCDYmBLX\
\UvcqOmHrbx9vqMackDoc00FOQXAkfmDSQM3DeV7FRHASBS8tHILgVGtnY8t20ZaBJ98\
\O1WVOd7bHpz0bFX9Tmx5tBqb6Vi15J-KfJtvG2iL8D8Cg5EkfaoVwA_0-tNUdKIvMuA\
\EPC3vdkXNHwWQ_4GOf0fSmNzfda2mLbjbMRFlKyOZvNw9OC11SP-fpo_m3pD70qtyUg\
\GtIs4CuQyV9-PaVelivFtuozvYgloDISDcB57oGtPr4CenMT5LZT4XpwnlFgvv0qi2-\
\-tbxiOOrvBicACQAVZ0m-H5k7mz2fKSO7_CNCKOxD3tY75Av1Mg62qdaf_BNT3ajkbH\
\jkOJx53Ua9NmX3H8AFLYdvUTraQ3NtXQ016jCTniEaPwDQingrJk7F-ITKuPD0tMGSJ\
\AZDSssmokG_BrcMQ78cxv9jB1k_vejN298uE9blH9fs7Sjssmppt3htBQQ3nQ2KVHdz\
\EmMcAh93qS909BbCVuzbm1idgzf2rQoET9I2-1840m23fP8wD8WlBTyIK0dtDM50Pth\
\rNX4CVUZBX2xTsAoweCMcQEbJF8JRpdqpPB0VdWWzOnACUStrwRBf1DhgveQ9jQuHYN\
\YlnvtPBITqVCFtEFyHhI3y3a53IKfY14GVw2fheWTs92fXHzCleQ4BPxL1DnCY4NFIx\
\uSai9Gbs_0ozowx-Ye2CgbEq8ZSvE9xR6wntSP5ls_AUXBPlVNTxYGVT9QixqqpWLPg\
\RQMtBnuUkgbLYMHxCzNoPEv6qhrzRHq91VVHCgJG7AyRaLrZXPruoD7inByh1s1Q3n2\
\Rj30WKTKfohX4hX2LDOvwIeVHHA7Kwg1qT0-iC-CKGZ5TcWSP3kkdXN9pTMseD8-qEN\
\XBpb7KQsaFlxvE1Hx7KwAEJIojkvLeoIQJuYWXrtsuN05LA_iNJgA36T1siC8qankLe\
\EZpojLyYT3yeu1A0zjZlZ3-qE9kdrPqxcDpcpO5voJgtf-SM0bkqZT-H7o6izX1ayEs\
\LGzMNJdKZUKP18aoD8u9SpaXTZS1-feXln7ibWQdt5khqCWSpszryb8KtJ4S1vhEnU7\
\cYbx6X3Z6KkAH7_XTpD5jfei5_BvciIimy5cBcsp8mxO50miNto8031ODd5d5FoBr5r\
\xt5_6vICoC-Pu42WPIQCHx0WZljKu-6VcV-BIIA79BOwK4RoFNRRu7J0kY_YbO3WNdr\
\kJzj2U0UFZZLZ1FtfNMe6P3A8MlLOnYcQtHD7da03ieM1jmA0wgB_GcngOnk93fYj2Z\
\NT-5lllPEYke0ZDfiK205nuIbI276RlC_PrnmxvQrb1MH_f7rDXIEV3nZ31w7M8V-P2\
\zATU-NeUnQKdiNBoM5J7_zJWoief6--E1DBUSQqCc8qIDy1QMKqm7frexH5hzS9pwK7\
\YOi8LkRGDI_sVpbbZPDR4tv7xV5VZySndXbhCYzCyYsRY9N1IUvp4iXJqhELzpRw1GB\
\CrId4AReCqDkyoV7Byu7i86l76m-1a3YR7clsFQpTEkWxMeUdDZjCUdi_GxQfj_2w7v\
\HYzdIwZdxK0OxVXd9x8KlZepTXeoDqqhN0DDfEL2mbAx2l9Hi2_CHNGRKf7560Sd_zt\
\q7k239-cfpSwHBEYPKwJspntVvYuXxarGWc4efY1QPFvYHJtolc9DlkPQ3rP7jHtpJE\
\nHPJQZ2dhP2flG6NPl4cCxBzJ5aXY6a-tEbjN1sCpKexLP5LssLJ_NPUPKHPmClqlbA\
\SKlbirgVrBZ8ZhqNKeJYCgjnm8ePNtXKxcSfNuF9qinuLpiA1JymXi482eahnBWZETW\
\HIHdTEc6t2Tx78y6xhgjo-EhgFyZFAN56fnZNSA3LMLtd7hDmB39nR7H8pfE0MQqpKM\
\ly5HYHjx1UgHvwOUGfBt5QVWIQoh-NE_7tdT9DLvEcibRxqQMA_b80YYrSQqtrxUf7s\
\kmtgDMQKbUCaTWHlZPJweG2tGHJHlSxLSy2QR0ToNydni2g9ho3GcyocB2ECFOCne2Z\
\uIDiWxYuRQH-1VefZq9iQ4Piq8q-vyyTubb8ggASddroyAePF9R4fzRGtianrrQ8NR3\
\3lIjC7QcfNhjCi0XXjvqP07xUiu8MklJDfhbjTA2ZXzKu1PkykNUBxMDtwMXt5tfYH-\
\lrNsbvhOgFh2S8eyzd82yAb-oFYa2bZFuVyIVFkB8xIa1huMCgQDDRBk8HjGm8mfKtc\
\8cwh5dr9QYM6_z-id4rddRebN70GKwxuFqQoLSsNCjRos-xd-Es-uP-pJG21XxBVaB5\
\2IQFly587eszxddDL-k3mnZ6tDybDI-B0_kupXZB04N4uEHlWe-3MKv833VGSbsDZoS\
\6tOV9rrKST__MoUBmDI0AiP1pRzKGUIeMpeiG6td1KbaRCcJS-_nhjrgGX9OrF-ADvN\
\EUl-_gkOt0UlkcEcPED8BKFH2ZvTzKzKYJjYdu5RS72ihlJE3TJjy1bg8SMy5CbBo5y\
\DLF-QZ1vaZpbps3QlbF_Ch-6oHBa0-wuUUVr6Z4SUERLVg3pqVjI5xXNY26LYWViKcf\
\93TU0EAMjWkaVJFsWiPx8NVDBr5z-sic08YP1188sJD8RMBB8AR5ztHOl3sEutrTQT0\
\ifa1ni1hrn58wsmy75diavaGlhr6Illvw2qqUTLtLNDxh7Ocwxzztw1_NG5CdpHDyie\
\4l9aAp-DlK1NxzYRGh3jwpF46YVEegsTJsqRr2gkZItDSSowbG6XC6S9vcusS_mlKqP\
\UXmmSnkKx7O4vztv_hxOOIMDYLj3UQ141QPtBqktZbY8PeUhJFhLwYgET5opR7XEhpb\
\s32al1Ndnf1cUvgsGitG7CCfBzpgnokj1X2J7rnNZItBrZFLL-VC3dU0QaEfi4Njv4G\
\aouAo_QpahzNX2JlueYpP1Jk3alb3nPOGQSeeBfC6hxf9V9ycMC8uTAc7ocWlXcQOOF\
\igMK_3nVF3hxpoqggsFjHeHbSdWufdXyTWhzzHZSdgAi_HegEEjX9z2xpjoK6wodCY-\
\-jZolJtY4v-8-Wz_Da8-KRsJZ9rm8PI_uE5f25QOsilThZxYotTDK0YiTMVbGOF9iu6\
\nJmFPc0nneYcxlEKdIiQjncrkS5biy8JbOi7jzTU1BeHxEebpTYYHhg_x3Uo6JxYywP\
\LAlrSHQAQBu5jl_X1utf-J23FhuObJ_qsVLX_WvpIHCC-BkUZXqFVslzRoViCW9ieM1\
\QNVv51KqCP52fbNvqt01KUQq40l7dQY9ra-M9l-5ZtW9tGbKx7ABEHu0VSDWK0GVUx1\
\Cf68EZxo7enX4cGVryI4F6XR02RID5qrKYFGC8GEi166fGYJ_5WtzP4x3sYOAfURcyf\
\N1AXMCwelARC0_PRjwCGi48WDCVD4vv9L89bmShz16kAFQndX23eAZFfuI2xeE3sLfZ\
\mlSOGbCaeWKY-TUVydW7WA0QS_ZmovMxIgjGYMul07lzVPEdtduONei6chmMKVg19ih\
\44pzfRFInVw97IaegYTCaOIZZnrm0xacfOdP6CpnzvrIQUlL9lQpk3en9y9C7PdbOr7\
\5IKN99dRgQu4wLtSquAsfPs2hm3owhF9J4y1xxjrtbss0-XMV9IV4LDQ_Mcn5mbMuDX\
\iM_537rnsjZyTxxBLW7CQkF4SDzcEewhLIlPoDrI0cLyNjU0I9gu7imXWqHVMjwpaz5\
\VzATGxR_t0lnIkoMtpPJpO043ySpzRd2z__SFa7K35L8j3TnQImZzwq85Q8b9NJcwu4\
\ZeVbcUIdBaQ0ZCERdfOhPARSjY1Ksh4hjDL2MvFRfriA6owP4pNNl4vTNDbk009Cx1r\
\b5nAh-ZiD5ciUO0Bv2o_kFT4v32QZATCv3uR24W4G9IMYOb4gy9ebctix4zgmMM2WNX\
\tKAllSlBHFh_VpSh85X1dOuiowXVILy5rdopqr4UZ9UUHO_BSAkf5rshuxUUYJoyebl\
\ZOFMAWMaO_vtC12ipLetGIqBqV8I6TxM6u-Z3fKLqFWT-kN1lnnG_0ZMcuEyG44kHoG\
\oZy6-9S3lu7lPxNBYIJ0ZW_Gs6uJj2pIDBJHNxoibsHXLGGdmHOPrgmiMP9ZTXav6P9\
\woz_Xzkah4nJ7lqF_0vxGji4-u84wW0NpAIV9E8aOuoM6kCYFbPZJgUk4SQetVAdrYV\
\IrDGPxyz0m7SEQy7hqLLzSHCciO2uHlK8nKrfEIVu4ksTyKCNxYUR3E762bjfiJe2xv\
\a7S-5KQAycPIAqIEmzzkVlqyE0VDX__T84oPveatwSOoeD6bbMhn3nre-qtrznKh39D\
\_Y5u7KZ2RZZKkHsNII8etDBQlktETHJugNXykdYVd6JmlTzBWfXjFBL2Oty-S5I_a8R\
\6Auanp60QmcXoguPNjGbHQnJfeCe0V4c4nhe0nED3Kv3LydR0VfpigcOz8-HIL_Vmft\
\26KbqMwR85Torez0wlI8Mq1IF7ybSoKnjnpz8Mn6XTntqM-4GMxcfgoBVliMd5WmmZF\
\HEYgBdbddX2jtyglRwlZZdI7lc6URL9obvEgjMjEWmCXlHWxg0diRehr0vDvawuRyaj\
\TkXpvU7HD4fZfqNYy-JbmdEIGVpH5p5gbefzRLrZ0ljCd0enT6FM248h4X9MWO1XwFB\
\pL9yJXVaiDtI-lWmT544SrD2mSzqc_XumTrFkB5c95SpYwDba6TFpLehrtGcliZYt2t\
\0CShjHPm0PYK4z6NUiNETHQ2LIjkHTfCcNKIf5HobmhcMk7PD8Ed_Lxv9AllTPnOyr1\
\UHm0KFAuRz58tVhlcgmHMnpZ8V8AgEdDQ8f8ml4FzPzvseYoYHByeBCVGVYX_DwZOyT\
\_ANH9iqebJeWBjowSf0CgSZunvK_o4qOJdg70cZ6DbyKUFfhcRaS7M10ekkZpoxRLRY\
\EfwZ8U9OXx68Vhw9H9IPSJ1D4cp5Gxu-Hk9l23XTMsoYeGfUAy920mo3Zfw3DsxwYvS\
\KLMXKyqYF6yFqd60tG3OIuJvYbzD1ccQl7n2cO3AlDnN8PpJTWuBaj72dDrff2ovrKF\
\mJrYd7L2a4ikx9f6WY1oxvg7kJt-8SQ77oqzwJgCLT1-kiRyhBEomAantO-opEtpKJU\
\eTFnQJD785M7j0P74-5kpuIBhcQ0xseXLDgSjW5XjoAs92CHE_C0n1oXtqEd6lpQuqI\
\mhEyH47EBesknBmMQFe1O1Y808HlA81orOQS5qeJUtMxbo577joC5iHOKBZyAzJaYtG\
\bUNc7zd7L8ZZ5Aqa_AtaRYRLyb6yQCg3QT7QsPvjg7jpyAIwdxymChBsrqArA-zYs6t\
\EtNBMMuJsimhTmxZH-xNfyn78FjGq73d55fCPFyPo7ImSXn0i0lTSahJIk_zjczHgUV\
\bEwm_xEG1HdEMES5C9GnetuhhyW--pRyKeluIkSVtbbGnzUEUHjaLnV5B9PNbkcqP--\
\4UhvZJGy-nSHcz4ke-bQ3ElvkHdvSWYMWlxhGgfAmUTMHqfPFuY8tylkzCTix9_28H8\
\tkVGQjO0ag4ytYnD8eQ5b9lPwB55E9VlnTIbAOW1J1rixK9szX1ybM-lHVWQiraVN5F\
\Q-UkxW9Wdgx-U0Ms6FXK2J0tZFE7RdJUQBfni1fkgkj7RUAs5gELsCgoykXUyGHt75A\
\R2JSGSNhWBfm7PtrqO5zk7rNCOIjAyUgRnk0HA32satspJZb1XAagRvI2-fhWEiAmX_\
\DM31wRTmF8rzVidudr55IjDQxUGWOkx8Hm2jWdkZiLCeYUaTLY7aFCL05z8EZj76y4r\
\oi2tjmfoI2wPkXoSdFCTIpadCyNtR6uzHDGNy5uOGOHJIsyT4fM9RiPs6qOGVnod-r8\
\Gm49IP7tqjBVbHPLLJ30oFT1XM_LvW6gKYzoh1WNuguO9Wwf3hFlFYcn3qotB8tCe4G\
\7aPQH9bnmrGrkTQKA36wvZVcNicLba2j79iDR-nwVoahnJSkeoUsARfTYIYWVtU__SM\
\E4mx5z75rJIglgYnxZWrLrwKfzD8piDmtHkV45cI59HJv7b6PfTDgdlUAp-sAEv7hEQ\
\f-u92rnVHtNuvYuNovqGqQ0XERuky4dWNLGbGhjXwKtyDVSuvbdnMjKaUaZZlZYBTY_\
\dgM_UgvWTWV8Fo7OIZVLAXKId3RNVFFcep78JpopACIoyzxPQSxwPYCsRR1GX7J5NVl\
\nxzTaEIa2E8m3c00Ky1sTyIFVsuttwepiqm3l3pTm8K76IY152ugc5oF4Gf4Ib51k-m\
\GgnBV2OvJPV8ZmXD_OshtjWofFrwFiiOPCWfvTwuzD6az0iiZFIOGsT5lWIQz9eIJJg\
\3eDZ6rJmV6SPT2mH-F2900y6v4MTpCPeItCEYMQ4LVXZ0lGSh9OcFubbhPDMZJ1Auey\
\hcD5CT16Xok-QuDoepyWMJILo2I_cTNdm6E9KqNXSKIBxydrQqkEYTB2MPr1UB9HhSA\
\ebkkJFxKKH1tom7OpSBgCRSU0GXHHOUySyElyFeaXSz9JJtHHRXlBh5S39b5JV5YClW\
\YO-L9CqWNa5h-dvZ0wRGcLcRjub6O9guAbo4aUrh3Rtpy6CeB0qPFiqk05giWwTKM1r\
\wFAH6KhEIb6NNbYeTvfIIeVZ9CfMTE6Sx-dKC4yvehO2Q1S2FC_rZW8yW7ohcc9FDg2\
\0bKK8zby-JrxO7NSej6LjEHwPQkkdVSGkdLD0Rwr6hC1bKEu_13VGVJUgiKe489tk4v\
\V_2S6dA5vFhTv9iKnyOyD7lhim0lyVCYnRry_rfLQj9qvvPQP8EasXK5ljK-Qj1BJj_\
\D8BBgVecLiYnovPYhe9EZ9IG17X6qOVeGcRcXDT9L7DHP_nwE5sDvHNorvHjyY13AK3\
\yv86T1DNr9pY7GybmvgaG5bYxSTOO0EemkkNUlVLX8rTTDZL7Ciml6IrGZO8wioccPH\
\VfvGoi07bIeA5i1bodamBiHAlWTSX4gEZQ5DX5uIAkmYa5ryt0ObkLWHYkeEaEQqv3L\
\H5TMSRZs7dJHD0Sr9HSrX-Kk4dOKNOe7WKsfepjkERFMEf6ndAdwCtcW9HWsPVplnW4\
\JTUNhw1FuzBuHCQ44YHzRW-UYLtK7gE0vYmrNjvrNfze2zWKzi7Dpt_EojM2tYZ0Otb\
\_iTEn8kdAxGXhuguG12OwTy1JwPBF0tv9u-EqP93OVFmiFi5LaXX-KiJACoffdrfSTl\
\JbS0-xZJeXxUuKufNEUQebRlLOE5osYgcPvwhLYFn9RFqR-2xre4NvQfbBLI8cc3G86\
\8Taf7NU83fXAgQg52z4qVS1VjldTZ0y3aa6RiFqu3cVNqKuq29XCHdfjFfD2JbT8WLo\
\Unp2VScOr0ERmnVylBRXDtcsQr4POFLA3u3iCmMCFKRQv1s5i7EZ214GpqeHZcfu-nq\
\ODcEeOtHQHdN1ZWRXoM0ru7LukFgy24iDX0vG1OyBAPbgzlAIbQpju3uaXkUTGjt7Nm\
\WSXiGhQX49Gaa8-ySWm7q75Dvz4c_OnZcaLFW3Qtaky75qS7feF1G6RB-lyH2MWvlJd\
\UI-3q7WVagdIO4c-HzwjxVXI_SmVcOw_Pk8paUDSnWXvtMrXYDf5fPL2xNt2OVlXNhM\
\TuGH8z2sqt3Eij4VDQtHlwhb01OwI-yN6D45rUvTaVBoSLS-tAwFOenPkmY9-s_Kbke\
\p5s3lSP-D-QaUc7tInceXgusnAEz0ysJTDVmRyoqc8BvN8bBk-MjQ3ETSWfJ3R_UA9g\
\_d8CKpVG3m_Vuz0A1lucYIfC6KNsCu9ATt69TZHE5VCE_GFLPiZiQGAFTroVAYSZ-BI\
\Rd9EI-oj9jmWeoS_4LLe39h8Fb2Vd8YgGNfjVgSnWd_PzQnwFYgoRf8lYbvb278y7le\
\OY-Piz8_s4BKZt4eu21MRrFpcBgyy8m_1_AAv7ShqeY6Yu7HIonFzSRWJGUAiQ2shy4\
\Lge6N-SZr2bOU9zjnpqxM4UjqciylTLy1_xt9Dm5gLBICL4QdjbwhWktC4VSOvvNHFK\
\uroTRpyQKosoaN5NY80nMR5quSyfX522djGWnY-xBy274QWc57WoXUIwOYQVmhOd03v\
\-alAJc1pgskd3kj1EF2xlW3j3pMwTd5Z24bBcqossJ-SGmtJEFfLgTIH-rPZD9qqTqh\
\1nSeOuS2cpBvNfGMmCTIn2HOwIdyyOAzXeWu4YzHdkpT2fruOMpJnQdriwmB_B_PgWW\
\JThK1DyknoX4OeMBAv1hRvxlC1zGwkK_WyJfWrgNtFNswL2-_VMWMe_LnZGLLn4lY-v\
\J4CMzGA5sWuDrjEljv2lrbXQC0rocov_kXG4KzaS-OB_GddA5woNPbRVkpo0ste0d1J\
\RlhU4V-DOokb5E5kKDRARy0VCYhKmxDeURqu7POGTwQchf05aQFNiGa_9d8d1pJGnUc\
\KvALh_QjoCa3rU8w0PnQDCXoDtGDGb79TCVjavr4A0r640fol13xUJZuwTykpffRWPd\
\61dsiYs_HLs3sZmoESw3kE5jrnvJ-TGcXnwf0-QouTNIjoGhwB2_cyV6I2HAjxpFpzf\
\JY9LiBP-svHzQQxmz1Cba5SWxAf0g4mVLTkbeMBzIltc-o7fLvQ5vbcJNTBoPVOBi9q\
\_W8RY7PFJF9ovF9mJxy_Ya2UPRG2kNl71yDR37RcDJMvdgXkNcXd8FyG46ygqvg7xYH\
\dJbdrIITbBHl_JwQmooYTxULKTORg1my2gFO-5Ix3MUYRRsrcn1Dj8kVdEZwYc0nSK2\
\TG5_xCTo8IPwPlMpUOplPps9azE9BFuVz_2lf8ckVTOc-AfFSdJfBrhITSVEhRMwqIH\
\CR_yr6vxfomoyYVKFbilaNLEkVL9ckAmU6WnnpksgPxUu17ifNO7POY4UYrjDAaunxb\
\Nutr0H9oBgJbATZ-PYVlHNYGSSByflN2gQ4zERMXDwSdPd9JGw3kB7QidWWRFa-tMjX\
\LXye30WEpoTeV1cGDaBnB1wzq7uFfq5V19ajR8acLMoGepsXlm4HaDN2AE5U2mbQwwI\
\npav8wb26RdPoPt2Bby4kWrAzOKZUONDbQeBNmGQkItvWkSNbKSgFEqcwEgHKbDzLZG\
\UNFsKKZfW3HhP_oiAZCYTQ0QYBuL_6QeGZYvzmDQPDVSf-kKGRxt2HELcfyi8FoLwgE\
\LlMCLaGB8dDovm5zbo8Y42UMP_l-ZmmgaXVNwp-tzEJRkzwYQLwV2mvuKBJHCgueLoV\
\GnQKoYH6JYqcphBnQgYKyfzIRxkx49rnJjKVrRoHVTmKyMjGRvkPnOkWeO_WLrLP0jy\
\HpnyUE2X4Ib8jqxyMnPxKb_X0t3BGNz-NRwdpqhN0m9MKa4lOU6UifxkRXTkhWWXH2j\
\TWAtqihHh8GtcB8njXYZ1fRNv5YRbhMnf71bF7UHMKlBHaX4fhEet_3cLL3uLV4tSrd\
\HVZY7VXUShmYvkX6P37MNVzCo93tBIpfEGP1z9jzY_F5_-SoFKFY8VIxnsc2_k5JywS\
\Z7ZdYoJPOZXeE8T51-ySE4Cfv5DWNu1czO9ft9HQnRZhLMobhTVqAa_V8XErVItVVhw\
\zR7EnK8bR3o6S7EyV5e2LSQICdMtu3fBKhijAcCU6-BeS4SlzBFKOnOY7cqRk6WvQsr\
\3ZkoHom1zyNeTZ2u5Zj7xTKoT458g6uajKDj0pnpboegUhMFazddWQiLhEGDgt1UfWN\
\vV1fJp-uCGtS38AIXhOx9NCf7jDzrVpF4LaZ1DxDy64ktZR7OSfkhVuonXswBzsC4e-\
\u4ObOp63pvdgnEmaUMuS15Yyrm3s8iQvl3bFXHFb7TxwbVSrRhT9EnwmcP88x7hULPG\
\DGrqiFvzwNHWxp7SVh_WsjhqCbaBRA0LKKWTd0Rc8xsEfjZFIHgsyIei6LS-fQL6Fmr\
\uzZ8K75VKjSkv6Q9hMfQHX-TDXQdIg67MuE2Ae7fwVOVVgNy1UHWMuESbIj6XeXDUGs\
\Gj419_7L3uoUtMEKodXxhuidWxYpDq10ymQxRDxbCKE3CAVNPt_n0AwTXAtFaRQhHjd\
\cgKPUsS0MXUwb3SMC6glz5I4MkYS1sGupNLed7j7_nrv0gcvz74z6O11HGxxNqRz6Ts\
\TULxyDex9cUg8mc1uH6dsml1kamYRiCBqN45N05UgnopgmCq3TM7Mc8TQasheAwiTyG\
\lqmwn6-Dz-soMJBQiz3VitofkOQHw9PD4Z9n2krHygIvxx0-s9x0glCmjmX6r2f0ZOg\
\57Gh5GMR_DlZAzpaENi5CqJQQEL4kYomn1YEaiHV0iqq9j6n4FG_Z3KPRMLfHfWu7w7\
\zdFTAto2OUBEtVSkng6lwCOiUgmqucVoOKRvx3SaqAJAesHXLbYuORHdTePKDtNLXci\
\CZn-rdqPgN6OQiHX_UjakCSieFu_2LfGZXfiuu63UH_gIvTlGNdtDwf3q62QGCJlXxA\
\R4ZdgeJuz9hHlg-PtgHj0aknC3QnkoYB4UOikmVaDoYMWXWI2ajrXWt_45YDGHRvwNQ\
\Erroyd-wwz-yX7zXU6DsEVDiAEutRbgjgLN-4CHiHv3XOnKBe0_Q7CuuQztW8PAblmf\
\D482lGgXFQvQ6sV29v47me4S2MU-D_Wcd3eMCRQNi_hRrJQT8F9xupJ6plYhhFfIPUD\
\Mkgqvg0ekO_hZ-M6yeVXZfSfc9micnMnkyBRViA7RpwM0O_z_L6yBzAv5xlRm3CXE8i\
\1pRx3ec55Ts8_EPO-38WZyIudcLuQtY1FwJX7d67zSvoN9H0H4R3c1mra86wn0ujWgI\
\NXga_kXWn5vKP-WgGrBoviRE2ZaScSvQI_3EoxGNLraY3Z2w9w84uP2jOe-WGzxIT3A\
\7s-eLIq18p0fspiMsl-GDiBCQV1a5MDNV2en1EPImbCS8luUmvV9pXDLMFRTpH1ClLe\
\Lx7aJr0SLJJC5FeO8oEHnJC-J_NPYU8aT0TYfvFtV7e2JZXAm_BoBLEpWn7MIGK7HrD\
\6PgaS3xkIbToJwvpX2tSfW-yXLyaepdKptcCd-ZExlqPaY9fCJP_BKvHDa_CTP_zrQz\
\PQOlN7lZKPegro39XpWuLyk_DxgfCVeaWAJ_Ni6bggBWLMuVhYfZqXFcYKbLorjG1Fu\
\nwxpZqnAyjF3sqfTziHIvGY0V7SJycjBMYLetR9eenJS_GVx3g_gUX1uXfghDG2gHm3\
\mClr-SIOAsUBxoiUpWsHiVny9vdULRJBLAHF5F_ITUlWtuYWm6YMhrQ2SDE5lWYbdOu\
\tg7lUGU9hN7iPTSjHNqItc182aL7YIxl0UfQTq8HyETMvfV2LzbO2UpWErVNEqkSxGf\
\OAMsTt2OF05O7k1Pfko_5DkjlRmYxh8n_Vl81iiR0XXPbtpvEjssJOu-gD-OnDSCsfO\
\dwwjoEtaFB07Woz-33ek6RPRIRfim8hErzhbLVcgv9gN6Yk8gMUraMFZapciWAdNinf\
\xyrpFlm-LI2WEbqzyd2ObUnRSyTV_ijxZvU3216spIzbElzwr5RVE3UEY8s91UlAsjG\
\lZ_75AWRWMaZ5mo_phJEZkK31ICUJGQABPgMU0eWrs-5FWucJh6AzRe4mUqmXo8VkNE\
\f1hAXDaIFdYlbbprhkdZBUN4adMl_OzMtk0mDmqO8j8AckYecZ6Io7TuyvRWDf8iMGc\
\H9-YxeBAqBv4cvhhiUvTsaMYlOOY0_wzj38TlSnTRAll1BFeP9D0gyi41X4L7GgScJI\
\kGsCOt2Nme4R_hXBIruV7XNCG0ShAIows-mkuhntGvqirH4IiCsKXpYIu8nA1eo0YRG\
\SZVjAMMsZHY95ba6KyJImdiTlEQYnmlwErJp9KncBBhMs7ue7SC_hyqx5jY1ezwAS4_\
\rCFM2IkHyoagwQIiGATc0AIHvMpQlbbDGuwqW2xTR11jQXLcM07oO2hLQUgmJuT3pD4\
\J4p9tLuBnmBNkKFLnRYV0PztbKM_GHVUVPFPC4I3qr9YpHm8vZocFVkeO6ZBEuz568z\
\sGqAiNiVu7gTR3B8o_JBi80kSJRVGA9tbJ8ULEtGR31mKjhyfhK1sca__7Hep-6sR2v\
\prg5cIl3xCooRa_dp--HW3KyqtqVrR4KY4LGrfP0lu1uQpzqnn2I0nFQS1iL6pI5od1\
\_mShLeX91JA8rTkFUKDIC4QnTzlieCqaA34rTcfqx0kFVyV5kCOBcOXu4GxbIcuyDv4\
\qsi3FSmFQRSxjk2Xt8sevZx3onBSXUPCWzp-cLg5TBvW38ZxnEon3oKY1nnWM2-eR_i\
\e80BnBUOi5GIWHw_scGJjgVeM6lxwWS8PCR7l02YzRioJZuSONusI_IzWg4SjiOLajw\
\mzdjYSqeAiYJKO-BDUMt_bYggHPHNyHsYzRjvouaizrGLxMwamIHkPvjgDzhJ0UF0wL\
\aAXo9ujZtk0GI9mIAOy-squRxiPc7vuoC8vJoXisKVE6d1D9fl5CW2914DxOqoymiwZ\
\vCCg2pBe4flJAiWLAmG5h-Q6lFUc13rMdJYL679t3OmSoCQ1Xky7cIXb1x26jSXOp4p\
\8_yTmiBru13PYPb0n7_YJ3O3fQ63I8C4tDwRsSz_p9LcmGFemzYS9qsXA-H5YnpsN9R\
\boq_ubxuOtayWrr3aN0RCu0cexQva781Ga-65w9pMxak3mfjrFWI33zJeYH68qg-e-g\
\YZJWvvVYmNHJ_xGOKQR4XIa3l5WI2GAilSZ1Fla5jaPcv2LUFOXYKl5In-jWC3YgY5M\
\VDO_qZ26urBZcw5btBWWTS1fFX-lP3Ai6Dx_vdo5x0cUcetsV16E7_yQFIJ8vpt15ls\
\d0nCI1XT7kZwtYgNvS7pADPs5xMYtM_42hrucLdMfnJHlKY3DlS0jmbIFVslmgjAuws\
\OrCG1ziMkC3kOvvljgWrQWDl5jFAjQ1lX8k7l5hejynao3GeVxnne-ERVskNoIXDnEw\
\C7xCmPcaF-f1hCnXjKTTol4o1SuN-HZdIS47zgZr2OyzyiRQVMG0pQqLPdqBLmuAFuY\
\hlr8mqrvgOQ8Ty25zikECnlNUMPCzCJ7O8hgna6SN4bzxq6IyB9l6ToUhhLQt79DDUa\
\htDwZO4CqkLnamMatp4Xlc6uLx2DCMV0__zYnfsg1Sc8TLLGBLa1OgGwQwXzZMrQn5Y\
\CzB5Q0aiJBCucHXy3E2tu8Df0bjPv3--dUM1PcPDEEyXmA63I6j_Ib96HrYiKYXBAuv\
\AL5yO6wrvnYHE4K5TrMp4P4p7i6pVZpGsbHfp69BnQ99zEkX695RhUqh7d3CT-DmJ89\
\JfYus_yBt70rEYXs_1fekY3LZjXY6ElPtqLMCpJhRtMiAEdkP6HzlbYTvS7EiRvoEjr\
\Uyh-Zv1D7lM9sWOJUYH-SCrHng4TNSCiYAlcJ-h-liZ6eLJhDHGZuX5ErVeiL5YoreM\
\rWvnVXVtif7DaGxAVsMSlDFMppU69LPKArm9y5xaeNBAjmd3Hky0i77qgXlLvVUL-L4\
\fVK9dwXCKAocAj9rIx3k5yJCvoVtdDWTfziip1JWYO6FzVwuu61KfpzJnlIPHtcb4He\
\-rYCSKncs0_-vrQXXaWAHqW4Puswunvm5ZZJqjB21IRyrMXKy3t99If3htJm41eZ3xF\
\-dl1NUoiL3hZU_u8qf67IUSsu12MRg4BLQTFpYkFrR85hnPW3gx7K5-WdiusiTzdMYZ\
\oCMcfhYVZDaGu5aZFvNqNsNe5fY6HuxfY33Q8nJBiIueZzWwCSJpay-yuzM85AWmhsY\
\_1KBZ2rtI4Y_DpIj7mfT2i2w2xiD6Lzx0OP3mC8R2amjDSUNqYVY_NJOkir73PfTTf3\
\kDg8Ip7i2KsuiZj7KoYJxhloxC1Bc-Bj8DOBUi1K_JDdvr5KvPs_y8hb3u6ejffwkiG\
\_R5SFV6bfGh6pgNMcSjPkO5Xai9QHlArFFmWcHukyy2VA3xfp1wxVYTMZ5zECUl_SFF\
\7vqRoLha2a3Qu2rm5y-qKu4ERxYA6h6te2oRzajFiaoy8vlLEoCKoiq4d48q_naB0Hv\
\6WM29u0ofdsgRF-E4usdN-b9BuCwgldRFGT9UzxY35O3FjrljMbMgVIbf-yFS_1uYqR\
\zXV3kxW7A9MnF4UJ1TRLH2eYV7kFz6gTcBBo0YtI3jybkkf_qzil0uCzLu_tFVk1ETU\
\bbFR5DW0DBY2_KpOv1sw4XPZYToeIyiCGm0W3qSjnOvIvJQPsoTyn1RAKEAJtPHiaTE\
\nnB0H72ZYVNbNod1UWWs5K99ED14nJeVSkiEiBQdtltJkUFzPWXdo5Y9I7NRdesRHma\
\6-L58s0jFaKoYryW7tO46QAcnXmBi3XTVLP78tOR0UmHqTP7kQCdIMcccyDXt_dnnjl\
\-C0mDyThVzYlMVmbZNIHld5TgXrnwSSZ0dO82YzOcxAANmGzFW3ZfbEloZwwqZQOpEl\
\CPdOuE4MqO2SwXlyWXukvlaAI7pfp_ZkceNSRmQQZ3SBGRgJ2ecJoHwG1nH-1P8IhnX\
\DuRc4qH8VUhGoyfOELnwiyOkESEeBwodbvTGrm3nzdbk7dxkSgMOZ-EDpigq0xWA1wD\
\JaQ1p21p8izGroiPtkXGSkaRa5aIor8aWYPEeOG2HENAqqHp_n5MtMxAr84uxaDVQSd\
\NyVyFzXFpBepE8n0Fvtn1aVcYgsa4IS71KEbBhSIEzi1BwtJUw65C5mB4p35grAouwO\
\W0bQn1ixPbN9LxqUb5a_BifZ4kDeTg2IXB6NO9MEim_KTrO_mJPEldcbDNqrRCZog7u\
\ZUEW72Zh6xGWntNVoa5TbtO2QDW5X2_QX34EPU1yvWABb-tEPmVFwcGo9PaMQhJtOlX\
\MP_FWiy5I2Hr2BZhDD2amscOeTAdOw-aKMnIfZdR_zu-L2Zwga75B7v3rQ51AQvPvGk\
\My6b6JXIFmYFNPzSZ2oCFRQMVFgXT0mBf57lOt6axwubSim03ANeLOrBve6EvDFZRUG\
\Fl5VQDOXdfIzboIV0vaEkThKd9N3AcyWwxa44rz5J86VYIWyVohlg27CNs4papd1OWM\
\AhrLnUObaE9jBr6LK0divmfxQp_A6oyRTcu3toDVVzq9f0Ae668DApsxNc6aUgVxyh9\
\_I4ZHpzkoD1x6d2hWFWgJUVS_pS4qYZ0xK1bvvEjkmDZnH2QyJxTc7arwureP5kI36_\
\OIH2vvtu6BaNgiwnu-GyzaneGsf-6wTk3XGJ9nqfzJsbP9NJm8u5P8u-kFw2GUsVxXT\
\A63aw_TN_S4OXTZnkW4OYr5PLF-DZNcsoLqZyPbnv59fWys1PAGZpNKAQAkobMj1gsM\
\QvZJc0zVIQ0Pq7o1vPZPusIwpuGilG-yhpiJdj7QL-LUCmqROIG42wOrz2cN0TZSdBP\
\MKAl2SY5X87Uwns8zk_v4B-qv8brqNL7C4uPBDX2_Api5O1J1WQS_hS8iD1qCSRq0bo\
\GYE_ZvEWmFZzriaVlfsja8Ee6gyl0jI8sduABAxNsQKvqaz_9JQY515xoH4F5DjQt7s\
\v_RTbucpV3uGSVkQXaxs6teYEJym0Zy8_PO0qoTk52Jj9ohEwv4ohqXNBQsimhMH0g6\
\CLHr-5lO3-JU2H4_R5eGEmV16CWsKhmdgGLf3PshANQ_pdabw9yfWtEVif-vBglndZ_\
\126615okK_fadTKRVBFrGUYSpCG_b3HJpAl475rmULWV_-oYuviZ3YLahdlQGpOVyEh\
\xFbFHNUPeYJ6FFlxpb1vXq-9tHvoxb-YN9DlLZl3e8kyj01m5vVY0efsFkx9Z9qcfgz\
\bTH353Lessv0NcprRKzSeq75QykDQuxDdJVOZrVLDuYTsj0jNFECzzzqMeQrCAClxOR\
\8Ofo4CkseLHqr3BWzno4t2jLW4RpfTss2YmRbQKuJBCaehqjOVOIx7EtsPM5AuAymoz\
\EXUS1R7kpovI6Vm3i4OvyygG810m58bSEWr-MwRSgtgaVpe8kRFFFQ7YagIqLMuq7By\
\uY3vc00lhKdDDuqwXb003dHAmENIxY-PDseHsVTmsb2QGo2PT_L-ab5v0aWHyAi3evF\
\K-RDKTLXK2RRC9mE2KQpPOQXNI5ZQgZw7PmaxtSOlqSuK2xw7flJZV0ARnDD5IjFzH4\
\V2HqZS_G2tttE1J-EcSB76fe7wY_TFmkBAHNMc055x4a9WZ2f87aOb3thOYUHNc-zJK\
\eyn6EHs2UmANY91n1F27s1SdtF8Anrfkz9NArLCGX1BZ70OJzJGZaX4ZtQ-631k25RX\
\dBLCo-yStrSMmY9JYA2vJWtZceA8fkvCz5r7ZNdiPGet8nvibzahXBqvh1jzgtn7PFM\
\7f3eI7fGArOPGGyFPNrwmEzOIiwIUg9oVvhrVh6LeYYC0P5tXiofmqqcP4ypGuGwb48\
\UOeYG7Ue0bVXEPeCzluVh638feCnRnj2czo17oVD27ILaboIhSfa3ZVt2dsCLGBfTJ_\
\7DtLiH_ujl2dE3SkSyE7PN_qGpDzvJCsqb8QtNookMD0akFc33zFDi2w_mnp1Ocluvu\
\M1qT1qIPArFme8djvfEidT8QdV8zK0T9xbiz7znp6p4ds2I3kdc7eXs2NDUlsr8IpDo\
\3JqJw9qf4oLVz2rBfEcDcojwWJUhX8HVVS0_Um9lJUh94-75JdrnWBdylYlt32Vf6H-\
\XATGg4HAIGvhfhWOrK8lnRFfY-rj4czTAyJizb4LwCm1bBpoHV2azTK7qcfHgWKnJxu\
\5_FxK1ubv4kWMv2GH4R4UPUIWC6Em8ZfaKbMZ0lo332WvBBxmHMBh-SBikvt8KAYqTJ\
\c9YyUuphfwl2rQwPFPtGg9bnaIIcijSNL7Crujyi2y45p92cpOY55mKunugqOXsk5B5\
\JkZexQCPiT_XfmkDGZLj-kKP0vS6CQK0KwMRxMsKxaEYn2v-OCoW1uTBlvwfqG0UYPv\
\zuEfpl3vGAWPOjOYmOWBEhA0BkXgI1Qoi5dRZ0iQd3OLmDJ1u8GK3PfqP7uKDppeyKH\
\tQaphqXBPD6HEBRenbbWJFRBwnWLQkCSMy5zE8TdZIWqehblh7hWp2HiHl5r1SCNG51\
\9bc7R6JxLFSgG1fJn-u44A24VudlkYD5bmpCXjiN--dY0DBjFz2pCcue8NMwRD9F8ir\
\Xa9BBojwmpnz9hd-ZNjYn-kgxOyTXsOd5Rk2P3zily8RfbSYaGzF35g0o7GdmzGxYaO\
\-HqMMMxBmHoKG8tYyPx3Sj3UYxO5N2MD5uRWDdycsJk_c86FRXQt-j3b6WxZn-Aw4Q9\
\-_ZDzg-j8eCEJ9DSvKAvVsxSHW8bMmfSGU6pNK2dTEDdbzo-ot5EBcDaQ7pvcXlu3yI\
\s6BRvVuLc1jGqp6IAgCeuYxTIfpl3VkKPzsWZFZpQJMyByKPUWJdsJAw-goyRIvpetp\
\ujbYgz6K1dEWQ71o2AWXONLm_-uBW56JiOtHe1l6jYPbuhnS9jvE_YPIByUWaN7AH5U\
\CVEZioodxAnTbUPYGkf9LJaM0-sdU0LHMBqAEHVQvr9zGYhjxZ5rU2AOUEZkFlKWDbi\
\RQlDEhxaXAJrWhYQxLdxj9vpEGMmXVvfhJknw9Wz42pYy-iERxH3AV8wrzGmuojEivG\
\KIsV25ILTxRCubqyzREzwoEaFIL35uXI4o6-qt1JJfnjQzomLn0_fu2Vub4EacIgEIq\
\H7nBjc8DXcWrFGnOVFjRTnVzHmOCEWsGZnHQaYa0_nUIQ1QFkSYHIp8ZrW9L-HsMX7Z\
\ptZUqsgFz5lphspjAioICBmGTWsyQQAvmMJcEQlh79ediVD1wCciPecIVzFXoMNrtuh\
\DITJJyL3EDKZ92zC3PsrDS0TiGbXYCvyDZCH_Ffok6VszejaaDfq2TlzSE8ij5B9YwL\
\BNBRO9glScoiPTb5NWgmGTz1f8V_r6nnKf3LkcO3JgP6brFnp5Q3weFjf98mmoTADOa\
\58FZEwzZG6tee21mQmAcL1YhTTfgvLDHs6aBEy-5yyQl4orYZWLehesffzNOHrHfII7\
\NoNBFxMWUJ8128EwpVOECwrpWucl9jNiMLGQEZzkmdDk7ypODLE5DR3RXEfboR6eMJs\
\X7LshP3BQxgUYjQvSD96qlCYmHren9y6sVzIYxSfCERonG7hj_tzUtv7KbpZvOeH4As\
\YgIJkmcU3RADO3FwFlepw4Ug4s-vYQPN8Zv-7mSsvl2EpZlZWpC4Ay905G_2itedYOZ\
\ZXq_aCRIrSFfrbvFCW1SIVuWRkXP78DJkpfEtqLDW2MFo7LeZPmpizoFLrW450u4FzP\
\EQs2HOjlIQ6usYKPatG9ERy-8RevFsO7ryKsE5C1sd5KojQom1dmKgkqxDv12aFqFag\
\PFKtieJOaSM9d4tdNd4vHmuUcxYNL4SRGqXbokdvgWlvt8o7HnSSOLHJjNPLCcdb_rh\
\T7p2OMR0JiUYlWQZzCGqN7dqg19F__NugLty6sFNBF_86hO4DfjGsnfRzOO9QYPjgPo\
\7ddClw2sEBEIlSspuKlqWKroRGGHY3NSyv83A7KOdAbpxlpFz0qjYnpBrCXGSc9FP4-\
\4q81Bhu2OBn1eFR74ND4jqZirl_0fMHAYzrpY6cN_ZhUyAFkcaMy-eUUUWUkcFJjaxd\
\xreVSCZaYzf8LQkzHqibP3BPD60dSN1n0YQSikgHSfY1q5SqVv9xLzLT4qOxxE-LiT8\
\V8bBdBxDJD7nHtP1UFqcFmeYn9Sjntg5P3JCD-dMhcxcDk3ZsS9CSrsxjnvK1ocwrp9\
\azt-0Z7KdEr2Gr4MNvi78zWTH7DIG359VALiq3Z1yntb4UwZz_SnSHyfm_bN-GPmSrQ\
\ZMH_h54yyNbIm6atI96z56V9IYbhuOoO0ujm8o5_lJ-EbOWdb4DIYw3URcnAIe0dqUP\
\FIwe_HbzB0jjx9tonNrz9BeQTOdTxjgjAe3kmf_nR9DmJ6UecTkFiLb_ZstGqWtIdMK\
\FO4zr94ZoKnIU76oXPpidr8JHVVQKXnT8xZ3_GiVuN-N63IA95F-F7YWy4aRtQtLb5E\
\SF-noSkzMzNgpwZozhafh-sm71AALQav0mh4T0iLggpHeh2EEZi97HsINF9UjDO6t99\
\AvWbo9IFsV3GWR-0rSgnvg2p6yGcWdga_fuMlHtBS_hAVCf4EC3NxeCTnU9WWbehxoa\
\Z-2cyb1vTdPs9mUCY4bwSoEVo_IM6lORT7FVdFRrfsZXfNxEOOLgQp_uSN1LiMWAe-c\
\EnLTxZERcQNdZUoXNYs3x_M-q-B28vSaKAicb2TPAeFmQo1otq7wx6YyCTlUTM8tvo6\
\nBg5mpjWpHoTmDpnVp7WL-clUf0r3BUFozI66v82fFZqW3R9j1AT3Z5P9nTiV_GbHHO\
\2lK-4kX4aMO5jKjrZd6j6TCZvfGslZU9HQedECj8erKQDgCK02t8xOHtu553JAwLjPL\
\xkzeasYq5e_UOFbRG6S-dfWWV7lI7vMw94k2nEQfZnHLPuKwUUe-wi_0wRX_CGUnq8C\
\Kklz8RrnNLYd8YiLqQ3VO7iUzrYAFTzemYlf4rHencJ7xdGw6TQO97UqgMkEdUa8meR\
\uSRsng0dd8_mJmFsvB5cMiUwP3laEFhlH3PCtKH2-2g3tVi54fFonD-v0CDacAsfxMe\
\2cntL6-M6BRaT5wsjFjhwQsKff9BQz6TCHBYat4fqLq7DGVJt97sKtNJ6xvfMUNeCoP\
\ejC04IQTjWsriDalsDo2sIpR0zbFsJfc58N2cjI9FzZqSkw4rm1Yd9YAH3FQw9BXgPe\
\O1LbS4Ph2-Dffdyom8gqEf_qId5xNO43H6wmugZllGr-ZAD1E-g2HDdkwnwwQrKP_nt\
\PLuIvFbb00B47dqQ2Vfy2Zyr1vZko3Ji_OpJjaYOlKHJJP3JV6iHjZrdWK5MFOB0Hto\
\umy8VTIDkPEmsj5GM5Hf53qmU3cTt4gIPEEHvr0q0v_rmSxQA1ejZAtskqco4UYW9zU\
\6OSgmtolEoc2j80ZQPLdtpfJ6kRbx-3QyQmS7ivnXiTv369Zngt_zC6t_O_rweYRZeE\
\akSstJiibX1ncvadoNhwSKQdGSCll4MzbI8et2kBrWoqI3zNbkPR-dyUQmiz01YKWz-\
\_gt-ycAYyJStswspncSLLpNXACVDCzX13aPhxUk-n25X4ZFlYlbzxwoN1TLeo2FGluc\
\50Di1SBzyu08TiVJ9asnPw9HGuMdlyqK7gwGdUmHQr99jIeI5aD0lJQskIuuDJcFpXi\
\iZZqhIZjKVzTNs_WOHUGPFuBhVmqxKwcDaaDfH5cQn2Hqi_SfZZH0nXC2Wyu8GtBPIo\
\Vh2jYtrUM6ltX9l3DB9yRKZkx-rZKRgvqTIpf5gPh4RmNjJ46P3KClUphL06qsJtQ1Q\
\XvzmA0bA-rYKwz63100imj9iAlnZadz-muL-kucY-w3R-waKbq4klttfzvkPgLytyCc\
\ucQOAXKpomcSShduS6Z22zTC0-TSycdViTXLxFSoSWE6hMbbhBHtEzSqcbif7ZhEXT1\
\1Ya50jHupF2SH1Tz_BkZI9dktnwUMufsgcMoKKL_duh1TaCNSd4qDVC2x5TgizKRMEQ\
\rPztild0vPePgsLeLL3GoHHtwQwKR4fHbbTTlPLb-2e-VQvZAHEdVC5qcfMbr3FrU81\
\pHcrFsrJ9AxVZuspAt3oRchWkdg4VjnEAvgL8S8kox7Mh8bmYbFXEguAIZjG2bd0GDY\
\m1i-8Pbu-BeGi4GHo1gB_lskTCSbBCh7nd1tpSQS4T-uf8GcZHLWVGlwDYfg7_PpI7X\
\jmaIJ7yApfq46AUTxaU8sDKQYefD_zH4sP3v74ULs21ofksmcjEjZi45Pwygal9X4tj\
\uAHAeD31a3ISPvllwLY3xPo_7gzZ9v0PZNcBl-o9AXeIq1Ejy5b7aTzYuLSbdAC7WPv\
\F7Tx_ER5sWpXPtlxaYQ0PPQdL-PEUvp6tdV52jThnsJiow5mIVasR5P-tPulwPZVF30\
\bYr_c9pjvk5gn8ueVx_dnhaF8eaH-SOjm6C1mvbSzIaQtSM1k_dbjnZCK6cF_xkIU25\
\r7Z9j0otCEytHzCBEOdLjPTaKDY7cpKUT-oiRSGc448N6GRPt_XxQPe6tReIL3TkjaO\
\p_JXFMyJMRLTsWUC6ezTEd3Jdl_jbD-iGtxpFMWtVA4skkKrBHS84Ph5B859jWT0KCA\
\gKl56dR9VdC2umQIX7bsuS3zfe_MY-y3ITdzCDcCeIMDXklGsXxIf0M4SweSKQkAwmV\
\KB1xaZtrV9lR5a--cY2abWF3tdPKYqeaWxH6f1kvqDdmDLihSOeQrFZLgMoKrKEvZL8\
\aN_HS8rieM2TgZaWHFo1tD0oIOs8KUZnXEmbk_YBxJ0PNj4DRpvGSIZn0qSymoL2zga\
\lL_ubUQpg7XINtXwvNh4Af_jKyjnZ9AdnLTidltRhXHMo6-Nyci-DIC9TIdhedi_Pb0\
\JrA0R0RH6T2iItunckn7qFmEu1mBHyEHYleDLdcwZZdu3T-6ljqJHfm3FZ0Hc9oTg6w\
\4x_s1OfJZ5rJvutiV_Gh9bVcJudItUw8-y8EczF47w1Mrq5THUVhwSJn_dLtpARWOkR\
\o--mnKPdFXxb7aSifHtKbgWFs61Cx3OGdaRbVyfMGD4b1SCTO9dYxmvM_OqTpsvIynP\
\avnNxFU5U-whJo-EYbJAaQ2DJfFwJQ-ulYFAllDQZLQzI9KRHxuk_JNILr0ddArcw9f\
\qCn73z9izFHZ5ScmBzdGQE8-JUJBkuPF3mLWJkVwh5_firzp1CKb4hwyrjAOKQO91SX\
\QAwHb-d9KeFGSO8a0DP_aGmbzLUfTPMpPuRi7p7KqGC9y--K5deV-lhXf7o5UJlhXZU\
\JWBNYMoAcdzGhQjp6Rmgotg4v5FL76vKFp9H-5iRQbnTxwFdAG_epJeQ6LjUc0p6-E_\
\rBYBVRBv9Lp_ltRdlbtBTzgfl5ExVGtJzpwr_qXLLffO0R_ntfBw1oEtAn3ZaNukVK2\
\-0syppR8hq2CbmeX1emvDOYBZSkS2f8UQ3nT6x7WAoKARbXLVpj6YOBXh7E5KjRpwKE\
\b_PiixJ99I6JGUu3Jyk8A7ZGD-R0G3pI64ph3Qjq3GC466VuuNpbByTBFh3L4Ifwe96\
\qpL-p0Y1vN3qcZt_kukEP9JaB-RF5BCiEb55M6J_olB6gGbX_OEXSsT3KHK2t-5RbJp\
\na8T1tBeBo0WcV-EnOU3H3GpRQR4ZlgNUGR_UDN4EwlCrK2PV2F0PiDGmH8xKMUgKjs\
\K2Fg19yqB0-HbAgeELOjcgmOk3dwYZ0bR-IUt46mgxKwCDlfp5PsumjCs79lRDuWzyO\
\o5YQsXF0WQewOlUmi5HFbVY0eNue9rXHql2mPjT80hXOX3miYrJU27tMKcMkhSWAmhl\
\qHvvHH1Jm4qMGxBOS18sB8it8RcdYy8woiheURxlU9-OZqG7nMam7R12mvvNytIfpds\
\eHG3zrFlkHs3Xse1uXQiiVFooOGyb-2szxKxBtbnxbMfeyKSaLTTZyyizGEbrzoZ5Rv\
\P3CVYJ6Nh29YLTejagzH96aI6ackJ7pX3Q7TwIXPILEh48Qt7uxe9suhZAjHMS6CsN7\
\19axp_YZPZ67yDhnDqOCMb_pUqiekQu-xs681Z0ZDJ_3DQeuqMBSWaL8u1AVcGOLUa9\
\KGSa-N2VSMSUyVBHly6pKQWhe3YmmyjJQbb3teZDSPU-AMDy0aWUbfIo8udFWdmoJrB\
\ClVdC75qYyECQHM1RfhK2zpBdK0lCnTgrJIqRlpCl1_odx9fEuJd2UPAzH3PhmkNnlu\
\HVeHFfyDWKfiDsT1BM_oN1rW29ds2RfdYHxZVYrlf45XAyPlUB223h7ngpai5uAjCEF\
\kK4vzSfmwZNpMY3JhFSk_7IcEZuZzDs4sxWUtZosEdQQsfRNM0LhpaOgf9-kX7GV9Y7\
\jAQczEj0vtvYK6Vj86PAcLMRfMri-ShBEriFdltr0a_jDjVT3Bu2W8AUTh3vqMMrND_\
\F5zT5zBws0nPIzgD8YXa39zGvBvU4fUZusB83IZ4AYphcxIVsF4vl7uOpk81YvZsL63\
\wE2Zif-9dbL9L56MSJaFlASbJCb6o6p1S8_vi9QqDQJHnjcvUdXmevexgDi_PHVrJ9b\
\mO67licbb4G_wKt3O89bGi6WjBNBqyOSX-XoaAQCBYLehPa5Sf05Co_KBmPpg9P2bFP\
\TeupYpbgxBhTAMY9hMYmCByOpQT1IULZUkJjierYiPcKyOLcZ644iGU1yeu0zvhXja_\
\lkuwfgoDX384d4kw2YXvdqBIQOID195lgSpOYQxvSybDvv1NQksICu2fX0Eyi1SrpFi\
\u_tHm_u2K0gml1Ljun5DcReIp-uq__XmH6EoyUdM3kklGyRGo2H5VT5ub3JCJ4zKoKs\
\j2Se_Zo8nSCc6YLgeGTqZlODmW2U_nJF2rpBU0qfSg78Bch0I9rg9bUGjcNtVz4VT1x\
\yKkfJc29QsvLIJ5wr_w2MY36HirFAKSHWfi-fYQmublQUEJH09MJzyMspMYojNryviZ\
\OWx_ZBf7DbaIp3cMYCm5W1dOs5Ke7z6avkeju5ESgplAdeK_IOqCAoloi1CWBTIMJiJ\
\0zplulvqbaJKIJc09kX7t-3DAPLCjtjyopK1bGFArb4ot3mYHXORbmCdqjF4MCdvY8C\
\ODrnzc17LU1w2naoWZ9FSjJtTif_OWxE9KDhKsA5ZfLwMsSmqW-Z8LEB4vLIMloPs31\
\xd2CTMhTSFRDOXKslERBu5tZvKVitP3oM5W1hBzF_WtugBnx9y22xg-BQKT9lSNzlWX\
\nYw1kfCET4RulAW5FdseWu_L2U7L2eNULYGNjc7atooQpI-ixvkgVtlFfKRkS1oOJ_O\
\W27IOcd87KaTcPZzQigFy0kRsxDHwG4vcv3KIRto0KRKqkKsMWwIN_fJbe_8rGlt5QC\
\Uq4mwb7QoqVOXJrUyyAypHwVKJ7sfN_U4gEXgVbADk6gPQl6w8gi-yC_oaqLVFJnxZI\
\lUf-7iE0_DFtup1KZNp5a7Iydv31eKZZhbKTNEhR9zDTQNzYYJPGsiAlkS9NDAzSrmo\
\2MsALtSKLbx4eZW8t_5aIoSYS86o6sY8D11NP-hwcGaB9qyTVsNyvDIZfF1IHpU0eq2\
\C9EEBjaS_20u_yaCHUuzB8hWZc4I3NERmAG5v69Tg04UZtP7AFw_Jx5Nf7HJlFoePMC\
\YigFuEQAk67zdsZfkgDhqE39J4NvsAjYR_jtlkkxRf9HdOT_uWuEVuVXPc3fAwOvKfd\
\NnPjqULHa46kw_Jwj7RA42H_Xz8McUPNQuScM87FIgiK_2TGtuK7_qiJnbv_2V6Qwp5\
\UjqDmoQvWzfO2tH6a6m6juuGMjS_7xhxI_rZn_oQ51cug8wiS5nFqmegAR5QECLsfjW\
\N6iB9OxZ7rQ4QbBVZMytOTXiytLrxrlIzRztFRPXcuw_wSmr1O7eKG_SNbt4gv911l_\
\uBe_qaSZknH1MEJn9DABRHJ6Vj2Gvn4GCQnYf24ua2MpNrXZTPK7nd9wsIh9-2ZpJvB\
\kqsSC7Ggk4_aaVy5BPXlvkIIfXCudYUAT1_hW58sJxRCseEVju0h8exirR0E-4K0OlV\
\Ljq4NQ6DqICrZrkgNnlseckaH1O5l_VbnuuMBiZBpCkqgH3K3jByZDskgr7dHWnGXl-\
\Vo6v0rklSWmaR98I6EyasgqlFTw7CmhMRvTT8Db0Ei8ZPQPmCN1voLVRgN-d9s4gveh\
\LoOd65g1vCjmZHjUc-PK7tkC0d8ru8OUqBRXQwfvKQwSRk68g5JGnsZKFbw7kdaJTnK\
\6gOcDbwd9ToTS_MrT7SUX6pZdVVg7yIrxhYSdZvOjxLC63KLRnPrrTBPFdSsqLupE-T\
\uhaXBLGtGSw-n4RUBftih99-xUP811D2HO1WewPLdBU7o4WJ42hVe9V1999SUrfR_yy\
\B_9FkoAvnTOtFKYnHg3BH4x-8XTu2t4R6DjPlNJfMWd6yvIwbM70k42B2WCxrl-FnEB\
\e9r0Ty--Zid12pWbZPuS0BV_kFPwqK0NYs25m617GhXr4LcTWwXwAsHYSdW96KDkVsu\
\fFY6euC88ycsx94_QuaF6IttLw51AGaloW0YkmKntxchM2tnAZQM7OAtxwWJDyyPlfi\
\zm6OHNKI-RJY9_g3BdCawdaEvxwsEIicWD1rsXz1Q-kMmqnkPGJ31Tk6b28v1aldIgg\
\8nvHIevKq-MZnQtrclREqconaKwlLKd9RPu-bI6N3DiEamcEDNh-xOS-0-94V37F0LT\
\s6O6Q6L-U6RuH79I7zlShkoFWK4Yd1X-go3BMpEPLlDr1xAU8HAJAeKqkkuAPz_DHE3\
\sb65_3XOduvtnylRUwBX6N-ELAA-Mt0kC1H4D35UaG2x3iEPa0EbE9KDU-i9dCIxmu_\
\eMQrXT859EqAQnSq-U1nrQkLRMEr3caxH9X1iaVoT1cfuphDQmZGL_L2XRRBGABF-NM\
\klA9kkrpPUyPQBvnROFFQgNvg_JXIN-lm4IFCnq93EewrlWUdSpleIuY2Qsd2jdOsWF\
\pLsRMekcPCJ9QYtB_J3lmDwkk1175eZnNfLVC-ePuZMv5Sq8o31OI1PWi1nolQD6eh1\
\FFVRF0Ea-fJxB7Fc1LBkcj4tetS-ZG5W6BefiB9t2EdM6-eK6f_UZWmMnvhwkh0lKYH\
\DedKXX9fkXqxvFjKQMymhExPyPmyyQlHV3KrfRylV7Fl0HuBzRiWCq5z_ngmM3YQ_Z8\
\uIbH0U4UfRjSSYbVSem9qkR96qf5mI_-5Lz_3cTYdBTm4sNOBhzD3NIJeADfoHoS8n_\
\2XoM1YSsG3EOVzyArmdHCnzPo7btBMUXTVXXg6LGJVf3Kw6P4kzSVZ9NShdQCWnMB13\
\DWhELPiNtyGng-77SrpOQGPEFSbvHpYzA7Uj2HD6BNTgbEBgl8-Ij5zNytfgJxhhRgA\
\zm2DKkUZ2XzPt-640uSUjbLBZttPFr_eCvuqpLMOLM2CX70nssnjToKDzbTLPMynp1j\
\3u0jKqRdCTc_bkfRnt1mfYRMiOjRB6qg8PP9J5yn15Y1VXwkSDx--DTJgsvtTddK4ft\
\HaNpz6YXbx0SxHPufykzFpJkoBjKKTWsOAYOsR08HgfF2TuaDXLhXroXaxqFLBqKOb_\
\7FVYpY4_U0sl7mGpscnWXwotKKTxKllFzE0B2Cu6vnPPczOVLsWCPpqjs2ZmpTEo2EH\
\_06VTe-1KXej64RzJAmfBJoEBs2i3oNrPkYC-h39ZkeGzJpnc19TgMwlS6omC4uXKoq\
\oODJ_1XLt113q9n58XHOmypOWtjHLTc7HivhmB8ODIAkfNMmdhODc_Z2wlpPhSY_sda\
\idKoYpTOU7pSQrA1uvvRJQMYoO5dD18JtBumMaRNqRR43aKrtykYBNxkYA0x3out1_8\
\THkYChvdC2HALtXdgP5tY4sOS7CS_W4XZhwDNZYlsobAqQJ2MuEQayjHaUDlJIeaz1b\
\DYEJa9lcZrmzqfTz4nFeWk6PUzeQuc3YkLqOo_qOiuaJDO-sRCPPacaDM4N3RsTIQZJ\
\rcf8R2qGaOUB8umNVo8rbfzf7H9IoBGLkNfruUkrLqzoGtsoj2-Tb7zGrQRyIO5pmVl\
\Kfp7QEj21Xx1rJLUyzElnG8hsLiKMsC-PJP54bDt1RRESxHYLi2Okj_rA7hUQYpgupj\
\kVQv2MhirZbgwdPKcVNn1le14DripW_E884rvUJdt_-dyWLnDuDq03kZzRqqlPpXTXk\
\idXlRvJyKthIyFxZRlqKTFIfCHdQ8yixO94RYqz9nw6cchCo4ecuT1uzs4qlHmri3ym\
\n6vjBGJxwFNRXUdV_OryVW7cSePzm8-5Vrd5F1qYYINA_etael5dnbafJ5KeEbMxx4R\
\6cJA0FirFdAB7C1P6wiEfGfiBerHkyrcx8Bij5_yrirAGxi-RLzqAK-hcoeWaSg9kIe\
\Ku0zjD_-NqUKODwWq31hR0YPOXeprujIDrSKpRmnsTIYu8hh95haPwEMcd7wCo0ahvh\
\9EgOgvoN7KGVZcS8PxksBZBbeP3Mpl0J6nEZMWAF8Y3BKpBRZkb_NUbLFHk8QRER6Vw\
\O8yBuvTQWr5Z_F9S7vcLIvmpxfXRLz5aPIKiL4WkaRHXHbX0YaHcxXyqp4RjqNJ3BaW\
\YBr0x78_EKdmEfTzOeQ41SKNRlY-dOGP7ApzsRYtrzsQYA9d1n-Tb_2badRN7_YN5ki\
\4qgFooL-e_bKp6iM11KJhP-xE8nsbblZbK_MlW5120YppurXjupEXAzEJSTI6vbCvOh\
\WsrWJpUjU1WOkFmC-IZHL8X-HfCJ_CzxMcGoVXHMgyYKed3bR4Dx7WTFsvWfxYxHde0\
\x8qAg1w2q5V3CoAaCAou2DqOGKBILIRNVsX0MR-16FsjLzhSMibxYDSUUPso1UnrWly\
\uOWOLZE9itvMCZVz9lUU6CcI3CBuFbyxJsla99cKf-plqtZdIR8vcdJOBvfCagr9I-7\
\ylpwVMl_9BMf_SgZK36W23yHObSa1yFd35LAurL15SsQuUR0w6K4nuAqvNql7pEXOt5\
\g_tq39t2y_Hf5UjCT49Q_HVWYcEBTgX-KukrpVVuLRvp6ktlpJjBX-Bcx12hAs6IQn_\
\K8lUVoVlS4hlOECKAqab6n6qA1ClQL130b3XdhWke00QTLkyxWeqpKB_aBISKu-2S1Y\
\N7l7KsYs82o1l0ZqaVWIfx9oHobxxFZM2PB_zGqyf28oZJmruIeT-V0SRYEhlmNVH2-\
\rovSWmzQWbtMniCCXWCeXcUGJCa7un1eWFAxq6fb6x5AzD9NYDaPSE254VIBQ3P0rDL\
\yo77FojwVY42REbWM4NdqCw0uOGVIMryoH5C-Q4USFg54kcKg5I0otL_wm0Pc7wAVXr\
\l6v3PwIvoEGVmJEax9pqy1HEbhcXSwGzaiF-T1L2XW8w18jFfNa5UdC3My_zhp-icGG\
\7f8-BcN4mgbsuJfHDRxULSXsPlglnLbzNYk_DaEXIwkEOfLkHk8rbnPrdXINt78w0WJ\
\bGbB9tx8unaFdRPS2bqBYpelgBqySYI0qjdoFNSwakEjZKkBIQhHrjPcOGqj4tFUiye\
\qKZ42S-Qoj-hbYNC8b0UAjMoUTU8KqfUug_t4wGevV6U8CkD-W00sMB22RWl_DOAKMi\
\u40dpFGYlqGqbzzchiMg48plc0uk2JTXrRX2uV-2mbBqeGpKSRR_cflgcpberly7sqf\
\FMBEMZzJ6lPP0K0KH64FdBB6eMgLrRx6VemABHl2A7XiKRdIbP6H7cPeToqqhnXtgZw\
\rVSaWzFU3qKnrzXaZFeHK39VbuSdbM3raZmRhPRdeYQ1Q6joRqDhZxFdwyKhcEsiHaZ\
\jBEZ_6A39KpJVqsoJ1TEV9zQLuPaxqO9hno-xQ0wCfWIhg59hW0MpoG227adyBVYSIh\
\8jT7VhVEFcgNIbSC16fZAuXgvPM5_uEEPnRLtMtz8pZMQ8qghXmwYCHCzHO8gCkPSNl\
\AMQX8llEjwHuyoNDB29vILGaANHFhBfcU7dtntpcYH7ZQZtVlwbwE3xW2qJSe8j7PX0\
\pSwbT9IOxcMapkAZ8B1fLzYk2QH1sLHKVnCSnI_KqJFrL1FhBsthS1qrehWspH5t6EI\
\6wZyci0deX4S48lgTTgoKeEiXzBmSd16bea8V_-ORS3jc2SD-USwx5CAxsfmPvNeyhM\
\ZVw6ORWRpUZXXe5lYS0MxriK1yVoeaKYfvYDD7nr4VjfK5SZckMM1h0ZBvXeIHTY_ic\
\NRTyWqfXkDPRICq3wqWpjk22iIZZv-M4Z_pjI8e7af8R6Y8ZnIKNGcI1QNmo0BxXV0_\
\oKKeUeCZwqRDjOFu3EwcwFIuGdexvseqJ4nj7NFw_33AVQwEN2ubxxMFEmg5kOlnDCM\
\Brre4W8rZTBWck6CSjgpQLbwBPRT7Th8q3KH3-rS2gdFQPQio7719gYjiJTcSUKMkA7\
\OlAuH7FIYf0gd01FMwVjrCLu6SXkXKajL1n3-MTXvLwAiegOTZ2i1Y063b24AwS9I8G\
\Epwzk4zG0BanqQaTmBxotjbiD_9j1L3MTSNGBk7KJXCn52ShPEpwHabpBCHlsH3vCgd\
\d_tMVbLBjudv5b6sNKdqMjMLsTDZZmgh9V0v8R2lfLfZq0LOsFyvmWS145AI_rGEd3L\
\C02S0_BMDrVL6OxZRR8GbxLYfxPABIwmKcVR42I-lQp4-ZHpO9Y46fb0JUxXRUclzm4\
\Q9zKUEHG-QsDmVRDm019FFG-vTkUG3n2GXbc4muLSvHtd3k4D04eCvB1O85DxtJYQr4\
\Yb-_UOdxdmSc7MIRfknb0I0CtofY5Nr_-dLx6_i4Uq-06n2Ke7FzuXspeLY1RmO_L9v\
\Ec3GN-YQDgDShu_1OE9qg31dZ9KoUffgDhPJj8f77dTB5IibGgzAwK4zOuwTVc6Q8Ef\
\vFNCzea4F9YI7qfNp4-4hLRyL2KKxseHra-qwYT-JiUIrHoG6Ld4-Cx30jhZmq6xhTC\
\pm1zExjWD3XUglCKEHLHiI1oc8oFgfqZ1bXjXyYN5P00CtsVHZglfDUwsLFP2v7yES1\
\gK006pRIYTIRAGZ0AMd86PeLn4HhQffNj5JouKfexh8VwYdRUcDTP22vVP4MFH9sk4Z\
\fLQJkDdDhWTSSvdfYYsUmJ5jGMHMVwojHHkCg5HA1q97-ywX7BnWFUdqjUVBeFP65Jy\
\9vgze4tbfbmj7MiR38COrIeKtO4mREs8CK9z3kqHdvAfe-TzRaCHljpeffQVLwO9fFH\
\aQvL4IT2snCUB4No5_8EviyLHP2PfLHbMrcAIxzEnxeXC4yNDwrXQrLSKF3EEY7JOBs\
\Vde1HOeVkfELoeiwXVte-A9Z3cOUR5gAJ2pag2BVjBOidKV1R6y_N10ZRGylXjIKrL-\
\a1QUZpjQqFZk3TGWZ4c20zcHi_hfmh_H3Wf4ErPDj6Al08wsb6hdtZw5f1VQKXeFgEk\
\xukNQUKKvr9t3Ic7W9_rdGVtJwq4hyACFjytsKDfdFkcRm_PgMOPh3NR8DDleR2b4FD\
\DLkfhQYUZabgwpdUWBrkiOzBXPX96l2iJ3Y4sWjfwJc5WsvjCfyYhvX_a6CDa7t00hO\
\UVPm1mxdfLa1nfPMrHdcUZCBh3g19I56NYXbJvnr6l8qKLwo8wEw_Q1dNNkxcdGsCxJ\
\iBjbak9UY_qYECBM2UYCTE8jmX9U0TcE_RqmmCQ64JuZs88NAm-9r90_1HTt728VcIH\
\3CUFpzvpgmNbT1r6rM-t4Ej3iR-3Cs61PAOHDYB0uIPvMINgOjQBTvTaQl5Q17nXBzu\
\ypSjfHNiRwuXCG1mMxinMMNfms7VkHesP88vQmrGkMHOfjHxu__u3FM7xC9AjIQu7x5\
\omXmUpi-9A8oCL739lASAa3-XqcCXOilPjK8mJnapnwoEp9X-qkHVa26qcyVfZoIwQF\
\yo2FtHNkE_KnpgHNqbghP7sHMkH-eYwnLpEQtafg7Y1rQc0ZE8H9kcjmQEwkS6dE-Zr\
\IsVFkxOfIEEDQBzXG1Zs3rKxXb3uaskzsoeo1UGr3v9rUfn7v2uc5vVPcyoT6Yq9tmL\
\U0CDQdfDIONDa_olOnoUg3C5cCIiSFU8kWOkiwcYoEwNbycKoar8FfBTMpI5ft2_Wjb\
\_PUBZ4_yiyY3NGHy7TwMvyq8wrjyJ4PjjmMlATHzuBNi253PlKyhhn1r2DDBXood_CY\
\03vdoSblbnPH5RNbnfeuLi_NJFRDXP9aRmJkefU0wMnuMk7vocS4Bsneh0x-jEjh5-b\
\ZMwC9PMHoXkHp5ujXcZd5pEgLMOjMkpi3MC-4tfzzt0YqiLhw2RRItOOs_Y8URrtzNF\
\9HHEUOwijaXenns89MeU1M0lJhvc6m9QiBWGbq8_X1DlqqA2e0_Ue5cqbON37Nz8rK7\
\OZlC--iusEJOT4UoiMIWO0VS_LyRsAMJdIMwe1LhdVvmElyX3lQAfAYbilpv_A9nwna\
\rwJUApfuteCwgvMBQvxyUbXJ-vVyjOx1KASIBfwNP3uG13xb4bANzhlLj5Hrw_8Q8TT\
\4WzOFsLBgcg66rFHG7B15lvHNRU860cqAeES6NEVtSO7GrrbpP3HBSFBHbQZ_zaf4D2\
\2xvqjUTdf6SbMiCsJuJlAlW2NfvLJSeV8mrz_h2PTGT_SNMIyC8cmmmvf79R8RWtXFA\
\fUl0PoLA58u1z-wEnHRk-DSa8_8JzP5tv6GJ5Vv1G9FyfENMqFkEziLgHfE7UtP56gR\
\EQ4aGk7ASXQKMvXfnRYuArz8tXmyX2gOdggYeFputGv_NRv56JoBBVO-uDZAMa7qbOk\
\uTWm51u_BykQWbxDbsFX5Mmb-3dz9Hd_sdz82BaBl1KJEHBaFWsAOec00Z8uJZpJUr6\
\xTQZHm1Bhn_dsKQnlOUiPhk5XNVyHXUHMRy0Kn2mDbKituReiKldXC9BIQADfpcrQQR\
\o0ePnTKGuXoq9zsTV0seXtIMC6_suHYx2Vpiq5Fl7iuhmUhUHTghuqy3o2-RqL98bOR\
\QhXPddwMfpzk3FxMmCytLzMcJJdySW7RbCMaA0opnGnV27-EIKGqk6WSZd8y_mY9PvU\
\Vk1K4q9fSRug_cKyVY4NY8EoM9KKKSl6RytPvMlUVhjbTOWnoy7VkTNAW929AUfFkMj\
\em6_3DtGjdSG_hx5Fq7Qd7c-m3nG25BHidK-lQ5-tuVq5lE48pBBFgMUYpFHLZ-GL-I\
\dCH1_lW1K6714iHfAd7ishSvuLEn-hZdXdUW7FJ_A7tUzPb5cbqDV5Lna7Hlc7Z-zh7\
\6yoG693OgDxGTqUCH34nnPFRKxJi6CrQC1P0i45IxHDc8fgvDncuFPeqC_RueKQwLBA\
\45t29xgJitmeiYLDcMRQmO5qVlpUKuve_pNhH3b7bE3VNXCVk6q9Y-vqnF2yV9tatk1\
\-8Kko1fFt1a4QCTtoej2DAcGXRk7hBNrDavF4QBF1AK-aBxJdT2g9F54V8MwAv6BCEZ\
\Ofaeb91gTGslxB5nO5pkSALNi4221zaBE_HGt1vxTaOQCFEAg0Ul4969JY4TzzyVxvW\
\KNZsg_CbKJkmTnEmQIJg8ZqcygcTHpGMi_P6eYbAdhXBprviF8lMYG53WSGgWkrQwXL\
\SjY2QQhKsYkTS4Su8IKy7O7sxweAXuQqKIp55reMIOUrrE5FcdI_3A_EKEAFgfh96J5\
\A8DJ8yyMKVDLGmkE6LC2FgBx3VlZDzfNA2HB4oDTN1vu1LuJ0ynxQh2kvX3bxUyxZ1q\
\sAA6TDEM2C08aurXRh243NMQ7y3436P9DQ5qe0WMF9CxWV8grSr3hhA6FvHpNcHBVa2\
\9LFpPH6Gtd5l9302MM4sumcrlBdRPKN6nF79lAfPOE2LO2_pZy3qxRjosk0cPP0YRD1\
\YFbnIZExoHB9YgaUCT6pze5IRD_uX54qbwpxImfmC_aQZyoB1b29svP7ECiw9HVHuzD\
\PHiFfJBYSxSqGpVEMb0bzpnfGzPm7ER6BRzpKQknRIzFsmneU5oe4Lg2ZJuPxHkvvIE\
\1QdUQZkTnbhgRzAUkJfwcKcKDlMm82CWkLMTpV4wEHT5u_mmrvoFSV7l-mbeTAINnmz\
\RyPP7WWlNlj34Fr0NCJY8ebubsP0NqGFgRVrKT7a-PTfkBbvU1vtAJIwHang2vg3MuE\
\ook0IQlqKUpnJHzLMeeSWCRurs3ohjzy32v6TpVKmcJ3_4Q-LBjgU07whZ9ccvy9c6h\
\ekU9yP4PKkGlx-M_WIZYCvgIdc16vEELIclgqx3ujrqjwvrW0l1GXjASniBfDvtK6op\
\heMgAtB3Q-H4Xq4SECIPygYhjNbsQNkixrKbHrvNBe-TxCd1_T4m9E2YzVZm0B-vgtz\
\ih8oQoidXWA4HlfDJtU4O-diQnxc7QeVP0zZFTX83aQOKz4YFQRtQuwHILq_BXZJ0qI\
\isIcskEOVeOcETGJrdKSlQxtxfOPEf8zL3eUKnZrpq_66IFQGdSqer-pmCpIakTGe9v\
\Z3GN1I7z7eiGEOJ7_XRhV1Qy0vUp4-ivE4W811eyH_Gfytuj332NtQd8Nw4u6L4Jh3y\
\URCCtA7MQONEPQXmTS2NIfK2n1qVcGtBSJmSuDFAOvbZcxao4bgZ_YCBm55Y1IcR8LM\
\v6Dp-MSatk0R2Mo3CBkrVyA9V6yfezXfW4MlSKfiliuIu3OLgX2-ma0VEm6Ix6Czy0s\
\XjzFiM08SRq040XwfhNox5RgdqEc_3gG4lXSt_DKQqH-2n7DpBAsUTuCdh0fs3WTCgV\
\LYtljqyAlwzuycmB8z3f937ZN1bkxwCAqG2ccASm5Ksg5N1TW574rIqOREZ1D-S4Eug\
\fk7YSEqjkylZu-djPU4EECHzI05hibJbOy-8e02Dvgthoy5KJTJdlFBkb5rd8COTVMo\
\cueqVs8c42LGTBOC8d9Fy7qiwVikg4aERNRVVO-Oh0yo_LJTfD_7fZnTyscuqkRajMj\
\pD8sF5w09LtK0y9MMNdi4v9j0Oqo0YwxWh-9iM4i2FqLLC9j-erur1O3xwP3KsTg9En\
\E8Gle6RVZJKdS_IQfSy6Y85M2yGRRUkfdeVCFPQNFSM3vY6e3tywcJHJjZr2BPleyST\
\BNMvZb3pEDBhNycr0HHTnu2a50HEufel1pj4goFrFAyKPDRE3FrYbQGafVaT3y__n2B\
\MjIvxEueIpuZwBa3MiV2TEvuiAYQ_K8aIZNYVS7mWBigzBWgKDFBqBTtwUWwaUvTMYK\
\nPYijJ2XeBgKk6Jephx3NvZESnkYEU5nYnEtNotOQWO-9KZbzyPynWlP9GzD5N9mwnV\
\uYpe1fsTSebEwha9vsqj3sJuCrILzkJNnCofNby2SkoTJ_TGycNlYFQdVy1hvFU7jKH\
\qBpTF2etgz_o6LJtitCbmKS0_fkvWsGstxyoOmmHBKY02eqcwWtJlUoaKZmAtn10cax\
\fm8uxUY5UG-yEAqbny55ThLVB-jxreJrc614tK4niXZpPbTHNKfWswXbCjOsFAy3Fyr\
\rYsLJPkIbSCbeoU-Ftjn_7kCMnKDUkk0bI8HfMyFMdYQ-nZRt0TIcjN_kXnE92qFoBl\
\fzQmzRQyZREuwz2-dXdw2B_AlOri3Mey2sf61x5fB8WML7i01umB8VZ7Cr5GecsLiEC\
\1fewgwIKJcxlhumWBMblgpJ7CaXmFBEn_W7dTAbosjrpXHJhnw4JbniZgvb4zFFLXz5\
\hCv-v70o0uhxca36cEwIiBDIPTbFYpvqUwYEMwFN3d246Ma4-23XDoV6S5lk62TSZaS\
\Y39_3j8lAkhVQDUn-19WECv06kjF7SgJoSWtRXV9UMuk38MXlU9AKbkgeK6JrLjl6hX\
\M8RWXoO6TNuXJ6gL1HwgG-hIctTkkwUQywnqe9Nh6MrO1jFYDmn43YTogkamC3nMqHO\
\Ot8Nq6EfdPxOxsmTq-ADjt4NjsRnrhNNOu3MjCzUEFZGhNOykpeOfSqWI7sWyYMbMen\
\5YqnnGh9UWzb1BK_TiG-jH6tFhpABVr7OCEjnVIIDjSLMdEwl46TB32pWPoyH9zxyle\
\Gp8QSEJcWAdMVLibE1kIJZno95tbyVtSuiMa3eTr3w9izk0c1iNA3AJWezBSqvI4Q8A\
\5q3w3vE5AJmK2Yng6HAW3p-gjRYEQHzKs8zlpg_r5eQi0uulks5zBZRJqquHnLMeCf_\
\JKRUpHBwZ9r0Gd7FbHaiNPzZtyzBjqEjOCfbLp6BBXhqQ366gwNwTcLgh2aM0U6L-mK\
\4FiYN_S7SjNtDnBIgLVpPPwIlPHNCSdwGaLTobR_Un2xXW8y382CF98ViaieTGysigI\
\6mrDsZ26HYWqvr2q93afswz9jW4daIsAyjgLV-oM6w7u85nZKyyxLa9nIwAlNuw77D0\
\YRY7ez-RMiJf8XJGOkxyUxDb4CW3iSPNqg5YXPzIuBsVXAHcds-IgLzvxWDViit6g2v\
\KVBOE_hxGkd4oME2TMAd6fZP5QF2vzcnFOAGBp3H9WzpIU2dwRrMpKq2dGJDlWBPjMx\
\3Jkj-Y3O8Qp-QkNXxe29OSDGgDmhbDc46WmSwLC3ApdiWxUF6qKTSujWMzSkr9eEVrE\
\rO8PWDrqBXHPueYPTCZ9A0J4SSGPt_ARI3WKRzc_VyDNJV4PTp3O8IK_qRh8Aok3SCD\
\YQM21pt29__-AWaCt5lwsgb4L7no5F8jvmzhbLnCpZaHkAC6Byt6gUDCJf297FN03r6\
\H35rG1X-O6KQLh_3dpOBUehE725405XrnhG2crIBX1XoAkZRbfyuoZgOsxv-aGvmDLD\
\koa05R9AOpK_LIUg9hM4uyU95JA7KN4wUtjHvc8BM7I3sWNxk4XIUwRGNY-TlwXE19o\
\OSf2Dz6X9dbLHbCPKd1KzyPF3SRhN7FivY9DPJXCl95KEZcLRf72-VMLNkn2drA8JeH\
\yHmRHyHoU9xC0Nd-XVf08liELyvafkTHf8C5WbRwPYLPTU0KceUaPzGEcwQ7hXOFE7K\
\i2ST2YTSU-1Txg-c3lD88pxNys9BTgq30jwTr_vbEJCxJrgeFUBsqAV3Ljvpq_AT1_8\
\ur2eY87AzKnfaVXiBhuAVIdNBev2gdSsC7vi3yN9qqBxqXh1LUOsgHbmV9_1KWmIXs_\
\G4r7TmYIvhKTovaup3H4IvP36imfJPhGso40-WqsONOyzQ594C_XlQPaVW5GSACQATx\
\IgxhD-WIl9ohZ5mm3XLDpHAxCUu3ruUO1VTtwbp7bS0PbHYoClqdjLMSjwmNdWLyaY1\
\Z6-OBjlNcRM2Yjh7e_YRRtWd7Esslf1AJrX8LfBj_gKD_OeUfcy6jYKu6qL9eMn3ATQ\
\PSSxNYzIyWVFgNeVjzLwBuo7q414aomVP0PdlpI3VKQqGBhcJP_eq08UqD6hwxlnGrL\
\sX7B4bF0Tn6Fr2Q0CxoXv2gHjH-95yFbF08ai1oV59IU3wrrAkFBF7yHpYA5M3Q2tAX\
\wju3DWKxuKSiEjFblcxjokhRPe-7MLH6E-WBUklf2HvU0v7zNsm9eBMfhKDQJR8XCM4\
\YMcs01P57ggcoW_Tkfk66fymtg3LCRpJ4IVYuLmbqVUVzS2FJiBTiASG9sXgKsmMuJL\
\6TrgoEI5yHq-KXBqKz8R2vVC4zYxPnLm46mSzYaKnfDoha5KawNO6Z1DjXTN0Bdr19U\
\2yoGOEJwDcfGfswqeEyGJPHExQJ5ssu65LPMHBmco95R6xqLMWkGGiEkFNMMlq_T8-x\
\GZiL9VA8WDHpMtVGMynFBYnTLSXjrxQG7Qwq69OGzkTKuBx8i8_mrN4Z2SYwR5JfEUi\
\NS5XRnVHF-zaNIgKzZzkh4SYAeiPGSl2r-Os5ayVP7ju02Bioz0sCvLKa2ZZhP6BDlC\
\KrgIe9k-wWOrGaijui3vgMLtK0IUDJlar6TTbPiNmpT29o87ATtuvjz-rGSrBwGWjum\
\1_lGxk1y9o_KSCACvBQDxVj9tWqsQt28Si2KpcDdBG3ZlkjzzX42NwD_CgYZO8Rq9oG\
\aEELXGEaEVUvfvXDo-ziVFNtBUD81w-qG6YutK3zxhQ1SHPl2NmEH7C6gXIg2BpdQug\
\kxHFOkJaoe-87OiGIIgJStc_RHYeg_pgckz1pihA0uvIC1hHmkICCfDw_MTAmhhiLHB\
\VAAZ4qyp3YG12xXJ5MlTPOf1-ylTsFXxo6eviH-zcfq-B3CdMDH9GGngVcLMDC0NRuu\
\5Y6Z6a4B4wFeOc_GlgABg2IJuve99AcfsTcYQep1PcB_nQBnuO9pPEHP2Z6KeClEdEF\
\05Ar1w0zh58a_gLyqfF5By024x-lnBUotaLDnUF6rV1JXSnVPqwV-7P1n7fBqrrIFdQ\
\9ojrIe0cwI0jt_nj5YdcO9aqS3SveOl22prjPjo1GAxAb0V2oPVqTQEhAtzA68LiHjz\
\pGgt7AuRaQCOR3Jo7yDc27XJlq7PK9UEWfx-MzdCG6ehtFcpnPxgqs4KbSZEJD_hsoq\
\JkfEAP3cO8vCUECB_dVjQj5zXCqeHwMd15ue8ATLdJMAla6kQ9uD_GeUTghiDVOk3Qx\
\Wjy1qwKUOgodkRpdOfW8npNDH4l8s3GixQ6LBn8b47iVyKIMPl2pICdz747h1MgQv2W\
\r3y_97DysfbghM-iewTKOI-JaRcPVf0eH9ZAWORTVq9P6QXUVFcmUxTB3-4K_kWQykU\
\1aIPerjVGo_ooju2yty1-1CtInuvLMzl7dxfsHu4OZ6AS5Fh9OaNkhPHfzAUw9D43ZJ\
\KnGibhxaxFpbhie4Sf9p51kDBP_NoA_AVfS-jG7WXvpBbwAmJeeDlyeXcZBwFyW0HXc\
\PRFhkodNhviOffaj8px5KpxMAI4qgxdSrL27C4pmlZ56NWMvuBA6So0BkOzGeXyNKyh\
\p9ehEDcyKVgIToZcdy09EA4nOUl6CnVATCX9G13tfELJqcOx530SMQMLj0KX9jdYC-i\
\YPBpBhMfbjFp5h7zFg7lP5agqXAqxOB5enrD1uIHfgjaoKMptEBIPb2vP7q21_pJkPY\
\UJBAa_bsXyNjlJrU2ds6VL42aqzBd9fwk6gLJLTxXzz_w5vJAbI4laktZLu337e0tSt\
\6zviSwdJ-z7az18F46Q1l2H6nUi3Mcpv0dU9A-6WokWHZB7Tm8v6xmImfShdvpZzZDl\
\2_p3mwxiK8j0GuMiS6kD8Lognv-wwSvX7w6NCC2spzTofbbeDh6h_-AGXr0sxq__XCZ\
\rqQebAqo2w6KiUimp6HOPD19yOzuDj187W3_pE39TZyQgRu4OKXrph6hURGevqf2udj\
\9Vk4aPtk3sm-bGLD65z-qMfDvUwKqve1utKi8le0oTqDiVT92KgG4uTdbLdNZAOyCoN\
\YVLNCm-IBGAmS4pHqdbsz-p77MNdanXVQgPyLRZzxy5CKPQvgTUoi8Y9CcBa9dhxN9_\
\M88jIlJs1282BB05dYJVnYupsEhx_HBENzOpCEF2coLEZPEmy6CFLEtEgtq9k4Soq3h\
\AEpFlN_nd3RBqv0M4Q4Uve5egNMr3RHghbJsovrs1bHdNDHjZ7xsacW_aSlyEuIyvDa\
\jCLsl-GTDqb1rbarlV55zamBekyFYZzP6Demcoi7jWSlZmvvlThpZCCXtJF2n_7ozxx\
\f-L9Boi29Cx0fsiJ5z36xhOReXVpyWw4xkMJ2ybt_zt9XqsmVZ4v3RunJ025PjWcd80\
\hND6H1w2O0GqsUrQ8kFwwEIVS95oFXQ2ZTtaTp-XO7Dz9pTmF1pnHnefFTK4tC6SUF7\
\dvpFiUsYU8bwI-Zwix9S7Boeksx43NOBtGxBr3l-1WBr4qx4JLxcXptoT-vqaeRnix4\
\0kpBgxBfWvkr5w9pZ5AbL4wl42nLnTw33K6mqdW1DxVQNDTTmBuX6CjI1tTlxw8QdF9\
\KnESRSNp-d-2O0AvDcYgDCHSr7flegjQqQIQ7Nk9CJWXzFXp6vC7-bYAAGjhkabSFev\
\P97o-eIzfVDZZd3hp9IgnF2heMtZb9BAzy0R_XJ5W0H7tK_B9VWfrghJ9wGhQ51e77-\
\o67okVQ8iwSkiSrs0_njVroSrvb563wy5N-SGnbA0-V4AYeDyteZ2RTdr7ouzihoU7j\
\OHERvWFGpegIDyUOKJ4rj9_W9XznUlnzHjmIX47Wfybdn0fU3N-j9f9ZsT9YYfBSudX\
\Woa12F8ngJc8QT6zBmnCysBwRTlcjfZdWJTaLDC0gMRXezq59U1ahXKvYwlnXiWV5Sm\
\hiR2NpT2lKVFVFaEpUa05US3kxVFNFRkxSUzB5TlRaekxYSnZZblZ6ZENKOS5PVEE0T\
\ldReVltVm1Oamt5T0RaaE5tTmlZalV4TmpJell6aG1ZVEkxT0RZeU9UazBOV05rTlRW\
\allUY3dOV05qTkdVMk5qY3dNRE01TmpnNU5HVXdZdw
```



{backmatter}
