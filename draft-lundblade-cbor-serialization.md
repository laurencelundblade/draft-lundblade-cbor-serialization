---
v: 3

title: CBOR Serialization and Determinism
abbrev: CBOR Serialization
docname: draft-lundblade-cbor-serialization-latest
cat: std

date:
consensus: true
stream: IETF
ipr: trust200902
area:  "Applications and Real-Time"
workgroup: CBOR
keyword: cbor


author:
- ins: L. Lundblade
  name: Laurence Lundblade
  org: Security Theory LLC
  email: lgl@securitytheory.com

normative:
  RFC8949: cbor

  RFC8610: cddl


--- abstract

CBOR allows variable serialization to accommodate varying use cases in constrained environments, but leaves it without a default interoperable serialization.
Here a base interoperability serialization is defined that should usable for a majority of CBOR based protocols.

--- middle

# Introduction {#Introduction}

This document provides a complete definition of both Preferred Serialization and CBOR Deterministic Encoding Requirements (CDER) such that the reader does not need to refer to their definitions in {{-cbor}}.

The overwhelming purpose of this document is clarity and ease for the CBOR ecosystem on the subject of serialization and determinism.
Aside from one small change, the restatement of the requirements here doesn’t change anything in {{-cbor}}.
The restatement is for the sake of clarity.

The small change is to Preferred Serialization.
The conditional “preference” for deterministic length encoding in {{Section 4.1 of -cbor}} is promoted to an unconditional requirement by this document.
This change is considered reasonably compatible with the extant CBOR ecosystem.
Since the publication of {{-cbor}}, a period of five years, the CBOR community largely assumed deterministic length encoding was a requirement of Preferred Serialization.
It is better to make this minor change than to create a third serialization concept that would compound the complexity and confusion in this part of the CBOR ecosystem.


# Information Model, Data Model and Serialization

For a good understanding of this document, it is helpful to understand the difference between an information model, a data model and serialization.

 |  | Information Model | Data Model | Serialization |
 | Abstraction Level | Top level; conceptual | Realization of information in data structures and data types | Actual bytes encoded for transmission |
 | Example | The temperature of something | A floating-point number representing the temperature | Encoded CBOR of a floating-point number |
 | Standards |  | CDDL | CBOR |
 | Implementation Representation | | API Input to CBOR encoder library, output from CBOR decoder library | Encoded CBOR in memory or for transmission |

CBOR doesn't provide facilities for information models.
They are mentioned here for completeness and to provide some context.

CBOR defines a palette of basic types that are the usual integers, floating-point numbers, strings, arrays, maps and other.
Extended types may be constructing from these basic types.
These basic and extended types are used to construct the data model of a CBOR protocol.
While not required, the data model of a protocol is often described using {{-cddl}}.
The types in the data model are serialized per RFC 8949 to create encoded CBOR.

CBOR allows certain data types to be serialized in multiple ways to facilitate easier implementation in constrained environments.
For example, indefinite-length encoding enables strings, arrays, and maps to be streamed without knowing their length upfront.

Crucially, CBOR allows — and even expects — that some implementations will not support all serialization variants.
In contrast, JSON permits variations (e.g., representing 1 as 1, 1.0, or 0.1e1), but expects all parsers to handle them.
That is, the variation in JSON is for human readability, not to facilitate easier implementation in some environments.

Since CBOR does not require implementations to support every serialization variant, defining a common serialization format is highly beneficial for those that don’t need specialized encoding.
This is the role of preferred serialization.
It mandates a specific variant for each data type when multiple options exist.


# Preferred Serialization {#PreferredSerialization}

The requirements in the next two sections replace the definition of Preferred Serialization in {{-cbor}}.

They are restated in normative form to be more clear and so they can be formally referenced by the restatement of {{CDER}}.

As mentioned in {{Introduction}} there is one change relative to the definition of Preferred Serialization in {{-cbor}}.


## Encoder Requirements {#PreferredEncoding}

1. Shortest-form encoding of the argument MUST be used for all major
   types.
   Major type 7 is used for floating-point and simple values; floating
   point values have its specific rules for how the shortest form is
   derived for the argument.
   The shortest form encoding for any argument that is not a floating
   point value is:

   * 0 to 23 and -1 to -24 MUST be encoded in the same byte as the
     major type.
   * 24 to 255 and -25 to -256 MUST be encoded only with an additional
     byte (ai = 0x18).
   * 256 to 65535 and -257 to -65536 MUST be encoded only with an
     additional two bytes (ai = 0x19).
   * 65536 to 4294967295 and -65537 to -4294967296 MUST be encoded
     only with an additional four bytes (ai = 0x1a).

1. If maps or arrays are emitted, they MUST use definite-length
   encoding (never indefinite-length).

1. If text or byte strings are emitted, they MUST use definite-length
   encoding (never indefinite-length).

1. If floating-point numbers are emitted, the following apply:

   * The length of the argument indicates half (binary16, ai = 0x19),
     single (binary32, ai = 0x1a) and double (binary64, ai = 0x1b)
     precision encoding.
     If multiple of these encodings preserve the precision of the
     value to be encoded, only the shortest form of these MUST be
     emitted.
     That is, encoders MUST support half-precision and
     single-precision floating point.
     Positive and negative infinity and zero MUST be represented in
     half-precision floating point.

   * NaNs, and thus NaN payloads MUST be supported.

     As with all floating point numbers, NaNs with payloads MUST be
     reduced to the shortest of double, single or half precision that
     preserves the NaN payload.
     The reduction is performed by removing the rightmost N bits of the
     payload, where N is the difference in the number of bits in the
     significand (mantissa) between the original format and the
     reduced format.
     The reduction is performed only (preserves the value only) if all the
     rightmost bits removed are zero.

1. If big numbers (tags 2 and 3) are supported, the following apply:

   * Positive values from 0 to 2^63 - 1 MUST be encoded as a type 0 integer.

   * Negative values from -1 to -(2^64) MUST be encoded as a type 1 integer.

   * Leading zeros MUST not be present in the byte string content of tag 2 and 3.

1. If big floats or decimal fractions with a big number mantissa are supported, the
   big number serialization must conform to the above requirements for big numbers.


## Decoder Requirements {#PreferredDecoding}

1. Decoders MUST accept shortest-form encoded arguments.

1. If arrays or maps are supported, definite-length arrays or maps MUST be accepted.

1. If text or byte strings are supported, definite-length text or byte
   strings MUST be accepted.

1. If floating-point numbers are supported, the following apply:

   * Half-precision values MUST be accepted.
   * Double- and single-precision values SHOULD be accepted; leaving these out
     is only foreseen for decoders that need to work in exceptionally
     constrained environments.
   * If double-precision values are accepted, single-precision values
     MUST be accepted.

   * NaNs, and thus NaN payloads, MUST be accepted.

1. If big numbers (tags 2 and 3) are supported, type 0 and type 1 integers MUST
   be accepted in place of a byte string big number. Leading zeros in a big number
   byte string must be ignored.

1. If big floats or decimal fractions with a big number mantissa are supported,
   type 0 and type 1 integers must be accepted for the big number mantissa.


## When to use Preferred Serialization

It is recommended that preferred serialization be used unless an application has special needs.

It is usually implementations in constrained environments that have special needs.
For example, indefinite-length encoding is useful to send a lot of data from a device that has insufficient memory to store the data to be sent.

# CBOR Deterministic Encoding Requirements {#CDER}

The requirements in the next two sections replace the definition of CDER from {{-cbor}}:

There are no differences between these requirements and those of {{-cbor}}.
This restatement is only for the sake of clarity.
({{-cbor}} allowed indefinite-length encoding for preferred serialization but not for CDER; that is why there is a change to preferred serialization in this document but not to CDER).


## Encoder Requirements

1. Preferred Serialization defined in {{PreferredEncoding}} MUST be used.

1. If a map is emitted, the keys in it MUST be sorted in the bytewise lexicographic order of their deterministic encodings.

## Decoder Requirements

1. Decoders MUST meet the decoder requirements for {{PreferredDecoding}}.
That is, deterministic encoding imposes no requirements over and above the requirements for decoding Preferred Serialization.

## When to use Deterministic Serialization

# Deterministic Encoding for Popular Tags {#Tags}

## Date Strings, Tag 0

TODO

## Epoch Date, Tag 1

### Encoder Requirements

The integer form MUST be used unless one of the following applies: (1) the date is too far in the past or future to fit in a 64-bit integer of type 0 or 1, or (2) the date requires sub-second precision.
In these cases, the floating-point form MUST be used instead.

### Decoder Requirements
The decoder MUST decode both the integer and floating-point form.


## Big Floats and Decimal Fractions, Tags 4 and 5

### Encoder Requirements
The mantissa MUST be encoded in the preferred serialization form specified in Section 3.4.3 of RFC 8949.

The mantissa MUST NOT contain trailing zeros. For example, the decimal fraction with value 10 must be encoded with a mantissa of 1 and an exponent of 1. For big floats, the mantissa must not include any trailing zero bits if encoded as a type 0 or 1 integer, and no trailing zero bytes if encoded as a big number

### Decoder Requirements
Both the integer and big number forms of the mantissa MUST be decoded.


# General Protocol Considerations for Determinism

In addition to {{CDER}} and {{Tags}}, there are considerations in the design of any deterministic protocol.

For a protocol to be deterministic, both the encoding and data model (application) layer must be deterministic.
While CDER ensures determinism at the encoding layer, requirements at the application layer may also be necessary.

Here’s an example application layer specification:

At the sender’s convenience, the birth date MAY be sent either as an integer epoch date or string date. The receiver MUST decode both formats.

While this specification is interoperable, it lacks determinism.
There is variability in the data model layer akin to variability in the CBOR encoding layer when CDER is not required.

To make this example application layer specification deterministic, specify one date format and prohibit the other.

A more interesting source of application layer variability comes from CBOR’s variety of number types. For instance, the number 2 can be represented as an integer, float, big number, decimal fraction and other.
Most protocols designs will just specify one number type to use, and that will give determinism, but here’s an example specification that doesn’t:

At the sender’s convenience, the fluid level measurement MAY be encoded as an integer or a floating-point number. This allows for minimal encoding size while supporting a large range. The receiver MUST be able to accept both integers and floating-point numbers for the measurement.

Again, this ensures interoperability but not determinism—identical fluid level measurements can be represented in more than one way.
Determinism can be achieved by allowing only floating-point, though that doesn’t minimize encoding size.

A better solution requires the fluid level always be encoded using the smallest representation for every particular value.
For example, a fluid level of 2 is always encoding as an integer, never as a floating-point number.
2.000001 is always be encoded as a floating-point number so as to not lose precision.
See the numeric reduction defined by dCBOR.

Although this is not strictly a CBOR issue, deterministic CBOR protocol designers should be mindful of variability in Unicode text, as some characters can be encoded in multiple ways.

While this is not an exhaustive list of application-layer considerations for deterministic CBOR protocols, it highlights the nature of variability in the data model layer and some sources of variability in the CBOR data model (i.e., in the application layer).

(This is the section on ALDR)

# CDDL Support

TODO

# Security Considerations

The security considerations in {{Section 10 of -cbor}} apply.

# IANA Considerations


--- back

# Examples and Test Vectors

