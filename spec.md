---
The Amazon Ion Hash Specification
---

# {{ page.title }}

This specification defines a hash algorithm for the Ion Data Model, as defined
by the [Amazon Ion Specification](http://amzn.github.io/ion-docs/spec.html) version 1.0.

## Algorithm

*H(value)*, the hash of a value from the Ion data model, is defined as
follows:

<table cellpadding="5">
  <tr>
    <td align="right"><i>H(value)</i></td><td>&rarr;</td><td><i>h(s(value))</i></td>
  </tr>
  <tr>
    <td align="right"><i>h(bytes)</i></td><td>&rarr;</td><td><i>hashed bytes</i></td>
  </tr>
  <tr>
    <td align="right"><i>s(value)</i></td><td>&rarr;</td><td><i>serialized bytes</i></td>
  </tr>
  <tr>
    <td colspan="3"><i><b>scalars</b></i></td>
  </tr>
  <tr>
    <td align="right" valign="top">
      <i>s(blob)<br/>
      s(bool)<br/>
      s(clob)<br/>
      s(decimal)<br/>
      s(float)<br/>
      s(int)<br/>
      s(null)<br/>
      s(string)<br/>
      s(symbol)<br/>
      s(timestamp)</i>
    </td>
    <td valign="top">&rarr;</td>
    <td valign="top"><i>B</i> || <i>TQ</i> || <i>escape(representation)</i> || <i>E</i></td>
  </tr>
  <tr>
    <td colspan="3"><i><b>containers</b></i></td>
  </tr>
  <tr>
    <td align="right"><i>H(field)</i></td>
    <td>&rarr;</td>
    <td><i>h(s(field<sub>name</sub>)</i> || <i>s(field<sub>value</sub>))</i></td>
  </tr>
  <tr>
    <td align="right"><i>s(struct)</i></td>
    <td>&rarr;</td>
    <td><i>B</i> || <i>TQ</i> || <i>escape(concat(sort(H(field<sub>1</sub>), H(field<sub>2</sub>), ..., H(field<sub>n</sub>))))</i> || <i>E</i></td>
  </tr>
  <tr>
    <td align="right"><i>s(list)</i> or <i>s(sexp)</i></td>
    <td>&rarr;</td>
    <td><i>B</i> || <i>TQ</i> || <i>s(value<sub>1</sub>)</i> || <i>s(value<sub>2</sub>)</i> || <i>...</i> || <i>s(value<sub>n</sub>))</i> || <i>E</i></td>
  </tr>
  <tr>
    <td align="right"><i>s(annotated&nbsp;value)</i></td>
    <td>&rarr;</td>
    <td><i>B</i> || <i>TQ</i> || <i>s(annotation<sub>1</sub>)</i> || <i>s(annotation<sub>2</sub>)</i> || <i>...</i> || <i>s(annotation<sub>n</sub>)</i> || <i>s(value)</i> || <i>E</i></td>
  </tr>
  <tr>
    <td colspan="3"><i><b>where:</b></i></td>
  </tr>
  <tr>
    <td valign="top" align="right"><i>H(value)</i></td>
    <td colspan="2"><i>is a function that returns the hash of a value from the Ion data model</i></td>
  </tr>
  <tr>
    <td valign="top" align="right"><i>h(bytes)</i></td>
    <td colspan="2"><i>is the user-provided hash function</i></td>
  </tr>
  <tr>
    <td valign="top" align="right"><i>s(value)</i></td>
    <td colspan="2"><i>is a function that returns the serialized bytes of value</i></td>
  </tr>
  <tr>
    <td valign="top" align="right"><i>TQ</i></td>
    <td colspan="2"><i>is a type qualifier octet consisting of a four-bit type code T followed by a four-bit qualifier Q</i></td>
  </tr>
  <tr>
    <td valign="top" align="right"><i>B</i></td>
    <td colspan="2"><i>is the single byte begin marker,</i> <code>0x0B</code></td>
  </tr>
  <tr>
    <td valign="top" align="right"><i>E</i></td>
    <td colspan="2"><i>is the single byte end marker,</i> <code>0x0E</code></td>
  </tr>
  <tr>
    <td valign="top" align="right"><i>ESC</i></td>
    <td colspan="2"><i>is the single byte escape,</i> <code>0x0C</code></td>
  </tr>
  <tr>
    <td valign="top" align="right"><i>escape(bytes)</i></td>
    <td colspan="2"><i>is a function that replaces every occurrence of:</i>
      <ul>
        <li><i>B</i> (<code>0x0B</code>) <i>with ESC B</i> (<code>0x0C 0x0B</code>)
        <li><i>E</i> (<code>0x0E</code>) <i>with ESC E</i> (<code>0x0C 0x0E</code>)
        <li><i>ESC</i> (<code>0x0C</code>) <i>with ESC ESC</i> (<code>0x0C 0x0C</code>)
      </ul>
    </td>
  </tr>
  <tr>
    <td valign="top" align="right">|| or <i>concat()</i></td>
    <td colspan="2"><i>denotes concatenation</i></td>
  </tr>
</table>

For a given value in the Ion data model and consistent hash function
*h(bytes)*, this algorithm guarantees hashing the value will always
produce the same hash.

Details of the hash function *h(bytes)* are intentionally not defined,
as there is no single function that is suitable for all scenarios, and
individual hash functions tend to gain and subsequently lose popularity
over time. Instead, implementations of this specification shall allow
callers to define the *h(bytes)* function.

## Basic Field Formats

Any UInt, Int, VarUInt, or VarInt fields are encoded in the manner
defined in the [Amazon Ion Binary Encoding](http://amzn.github.io/ion-docs/binary.html)
specification with one difference: such fields shall encoded in minimal
fashion -- no padding is allowed.

For example, `0x00 0x00 0x00 0x81` is a valid VarUInt in Ion binary, but
shall be encoded as `0x81` for hashing purposes. Similarly,
`0x00 0x00 0x00 0x7F` is a valid UInt in Ion binary, but shall be
encoded as `0x7F` for hashing purposes.

## Typed Values

A *value* consists of a one-octet type descriptor and optional
representation bytes:

```
      +-----+-----+================+
value |  T  |  Q  | representation :
      +-----+-----+================+
       7   4 3   0
```

The type descriptor octet has two subfields: a four-bit type code *T*
and a four-bit qualifier *Q*.

Each value of *T* identifies the Ion type, which in turn specifies the
format of the *representation* field. The semantics of the *Q* field may
be specified by each Ion type; if not specified, *Q* shall be `0`.

### 0: null

The value `null` (*aka* `null.null`) has no *representation* field, and
is encoded by the single byte `0x0F` (T=0, L=15).

| value  | TQ byte |
| ----:  | ------- |
| `null` | `0F`    |

### 1: bool

Values of type `bool` do not have a *representation* field. *T* is one,
and *Q* is used to indicate `false` (Q=0), `true` (Q=1), and `null.bool`
(Q=15) values as follows:

| value       | TQ byte |
| ----------: | ------- |
| `false`     | `10`    |
| `true`      | `11`    |
| `null.bool` | `1F`    |

### 2 and 3: int

Values of type int use two type codes: 2 for positive values and 3 for
negative values. The *representation* for both codes is a UInt subfield
that encodes the *magnitude*.

```
                   +--------------------+
Int representation |  magnitude [UInt]  |
                   +--------------------+
```

Zero is always encoded as positive; negative zero is illegal.

If the value is zero then *T* must be 2 and there is no *magnitude* subfield.
Similarly, if the value is `null.int` then *T* must be 2, *Q* is 15, and there
is no *magnitude* subfield. When *T* is 3, the *magnitude* subfield must be non-zero.

### 4: float

Floats are encoded as big-endian octets of their IEEE-754 bit patterns.

```
                     +-------------------------+
Float representation |  data [IEEE-754 bytes]  |
                     +-------------------------+
```

All floats shall be encoded in 64-bit representation (8 octets).

These special values have the following *representation* subfield value:

| value  | representation            |
| -----: | ------------------------- |
| `-inf` | `ff f0 00 00 00 00 00 00` |
| `-0e0` | `80 00 00 00 00 00 00 00` |
| `0e0`  | (no *representation*)     |
| `+inf` | `7f f0 00 00 00 00 00 00` |
| `nan`  | `7f f8 00 00 00 00 00 00` |

The representation shall be constructed such that each float has just
one encoding per the approach described in IEEE 754-2008 section 3.4
(Binary interchange format encodings).

### 5: decimal

Decimal representations have two components: *exponent* and
*coefficient*. The decimal's value is *coefficient* \* 10 \^ *exponent*.

```
                       +---------------------+
Decimal representation |  exponent [VarInt]  |
                       +---------------------+
                       |  coefficient [Int]  |
                       +---------------------+
```

The length of the *coefficient* subfield is the total length of the
representation minus the length of *exponent* subfield. The
*coefficient* subfield shall not be present (that is, it has zero
length) when the coefficient’s value is (positive) zero.

If the value is `0.` (*aka* `0d0`) there is no *representation*
subfield.

An exponent with the value `-0` is not distinct from `0`, and shall be
encoded as `0` (0x80).

Decimal values of the form 0dN (for any value of N except -0) are
distinct values and thereby have different encodings; similarly, decimal
values of the form -0dN are distinct values and have different
encodings.

### 6: timestamp

Timestamp representations have 7 components, where 5 of these components
are optional depending on the precision of the timestamp. The 2
non-optional components are *offset* and *year*. The 5 optional
components are (from least precise to most precise): *month*, *day*,
*hour and minute*, *second*, and *fraction\_exponent and
fraction\_coefficient*. All 7 of these components are in Universal
Coordinated Time (UTC).

```
                         +----------------------------+
Timestamp representation |      offset [VarInt]       |
                         +----------------------------+
                         |        year [VarUInt]      |
                         +----------------------------+
                         :       month [VarUInt]      :
                         +============================+
                         :         day [VarUInt]      :
                         +============================+
                         :        hour [VarUInt]      :
                         +====                    ====+
                         :      minute [VarUInt]      :
                         +============================+
                         :      second [VarUInt]      :
                         +============================+
                         : fraction_exponent [VarInt] :
                         +============================+
                         : fraction_coefficient [Int] :
                         +============================+
```

The *offset* denotes the local-offset portion of the timestamp, in
minutes difference from UTC.

The *hour and minute* is considered as a single component, that is, it
is illegal to have *hour* but not *minute* (and vice versa).

The *fraction\_exponent* and *fraction\_coefficient* denote the
fractional seconds of the timestamp as a decimal value. The fractional
seconds value is *coefficient* \* 10 \^ *exponent*. It must be greater
than or equal to zero and less than 1. A *coefficient* of zero shall not
be encoded, as zero is the default. Fractions whose coefficient is zero
and exponent is greater than -1 are illegal and shall not be encoded.
The following hex sequences are the *representation* field of valid Ion
binary timestamps, and are all equivalent:

```
80 0F D0 81 81 80 80 80       // 2000-01-01T00:00:00Z with no fractional seconds
80 0F D0 81 81 80 80 80 80    // The same instant with 0d0 fractional seconds and implicit zero coefficient
80 0F D0 81 81 80 80 80 80 00 // The same instant with 0d0 fractional seconds and explicit zero coefficient
80 0F D0 81 81 80 80 80 C0    // The same instant with 0d-0 fractional seconds
80 0F D0 81 81 80 80 80 81    // The same instant with 0d1 fractional seconds
```

For hashing purposes, minimal/compact form is required, so the
representation of all of the above timestamps shall be
`80 0F D0 81 81 80 80 80`.

Conversely, the following representations are all valid and distinct for
hashing purposes:

```
80 0F D0 81 81 80 80 80       // 2000-01-01T00:00:00Z with no fractional seconds
80 0F D0 81 81 80 80 80 C1    // 2000-01-01T00:00:00.0Z
80 0F D0 81 81 80 80 80 C2    // 2000-01-01T00:00:00.00Z
```

If a timestamp representation has a component of a certain precision,
each of the less precise components must also be present or else the
representation is illegal. For example, a timestamp representation that
has a *fraction\_exponent* and *fraction\_coefficient* component but not
the *month* component, is illegal.

### 7: symbol

Symbol IDs shall be resolved to the appropriate symbol text and encoded
as UTF-8.

```
                      +----------------------+
Symbol representation |  symbol text [UTF8]  |
                      +----------------------+
```

Symbols with unknown text shall result in an error as this indicates
that the context of the data is missing, with the exception of symbol ID
zero.

For symbol ID zero, *Q* shall be `1`.

### 8: string

The representation of a string is always a sequence of Unicode
characters and encoded as UTF-8.

```
                      +----------------------+
String representation |  string text [UTF8]  |
                      +----------------------+
```

### 9: clob

Values of type clob are represented as a sequence of octets.

```
                    +----------------+
Clob representation |  data [Bytes]  |
                    +----------------+
```

Zero-length clobs are legal, so *representation* may be empty.

### 10: blob

Values of type blob are represented as a sequence of octets.

```
                    +----------------+
Blob representation |  data [Bytes]  |
                    +----------------+
```

Zero-length blobs are legal, so *representation* may be empty.

### 11: list

The representation of a `list` is the ordered concatenation of the serialized bytes of
the contained elements.

<pre><code>                    +-------------------------------------------+
List representation | s(value<sub>1</sub>) || s(value<sub>2</sub>) || ... || s(value<sub>n</sub>) |
                    +-------------------------------------------+
</code></pre>

### 12: sexp

Values of type `sexp` are represented exactly as are `list` values,
except with a different type code.

### 13: struct

Values of type `struct` are a group of unordered, named fields; ordering
must be imposed for hashing purposes.

The hash of a struct *field* is calculated by hashing the result of concatenating the begin marker B,
the serialized bytes of the field name (as a symbol), the serialized bytes
of the field value, and the end marker E:

> *H(field) → h(s(field<sub>name</sub>) || s(field<sub>value</sub>))*

The *representation* for a struct value is found by sorting the hashes
of all the struct fields, and concatenating them:

<pre><code>                      +-----------------------------------------------------------+
Struct representation | escape(concat(sort(H(field<sub>1</sub>), H(field<sub>2</sub>), ..., H(field<sub>n</sub>)))) |
                      +-----------------------------------------------------------+
</code></pre>

where `sort` is based on a lexicographical ordering of the octets as
unsigned integers.

Note that for struct fields whose value has one or more annotations, it
is the field value that is annotated, not the field name/value combination.

### 14: annotated value

Values of type `annotation` are wrappers around another value.
Calculating the *representation* simply involves concatenating the
serialized bytes of each annotation (as a symbol) and the value as follows:

<pre><code>                               +----------------------------------------------------------------------+
Annotated value representation | s(annotation<sub>1</sub>) || s(annotation<sub>2</sub>) || ... || s(annotation<sub>n</sub>) || s(value) |
                               +----------------------------------------------------------------------+
</code></pre>

There must be at least one annotation.

## Implementation Considerations

-   The encoding aspects in this specification are intentionally similar
    to those of Ion binary in order to allow for reuse of existing Ion
    binary encoding logic, as well as allow a minimal transform when
    hashing Ion binary data. For those implementing this specification,
    it is important to note that hashing must not include any of the
    following aspects of Ion binary encoding:
    -   version marker
    -   symbol tables
    -   SIDs
    -   NOP Padding
    -   NOP Padding in struct fields
-   If N represents the number of fields in a `struct`, time complexity
    of the hashing algorithm is governed by the sorting of the hashes of
    struct fields, O(N log N). Memory requirements are similarly
    governed by the sort operation, and are on the order of either O(N +
    D), where D represents the depth of struct nesting.
