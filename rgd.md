# Readable Graph Description
A simplified, easy to write, easy to read, graph file format.

## Purpose
The RGD format has been created to address the complexity of storing, sharing, 
reading, and even hand-writing graph data.
RGD is meant to be simple to read, simple to write, and instantly intuitive.

### Ecosystem
Many graph data file formats already exist which provide either some or all of
the same features; however, all of the more capable formats are quite
complicated to read, write, and parse.
Most formats either prioritize machine-to-machine interchange, or require more
complicated machinery than RGD aims to need.
The goal of RGD is to provide a simple, intuitive, and flexible format for
describing structured relationships.

### Compatibility
The [Trivial Graph Format (TGF)](https://en.wikipedia.org/wiki/Trivial_Graph_Format)
is a similar simplified adjacency-style format with relatively loose
specification.
RGD is designed to be compatible with the TGF format through legacy notation,
but the relationship is only one way `TGF -> RGD`.
That means the following TGF file is valid RGD:

```rgd
1
2
3
#
1 2
3 2
```

and so is this

```rgd
A leader
B follower
C follower
#
C A at 1979-05-27T07:33Z
B A at 1979-05-30T12:05Z
```

RGD specifies a particular subset of its syntax as "Legacy".
Naturally, it is recommended that you avoid using legacy syntax when possible, 
but its inclusion is convenient for compatibility purposes.

### Conformance
There are many cases where not every feature of the RGD spec makes sense, or is
technically feasible.
An application may choose not to support particular features of the RGD format.

A conforming RGD parser must support:
- node declarations
- `# edges` and legacy `#` edge headers
- scalar descriptions
- comments and blank lines
- undirected edges
- multiple-graph files

A parser may optionally support:
- directed, and bidirectional ordinary edges
- multigraphs (multiple edges per node-pair)
- graph sections
- hyperedges
- directed hyperedges
- key-value descriptions
- legacy variable-length descriptions

An implementation must document any unsupported optional features.

### Acknowledgement
This spec is heavily influenced by [TOML](https://toml.io), and borrows
liberally from its style, grammar, documentation, and tone.

The requirements discussion from 
[The Hitchhikers Guide to Sharing Graph Data](https://doi.org/10.1109/FiCloud.2015.76)
has also been instrumental in designing RGD's feature set and requirements.
- General and flexible
- Quick and dirty for minimal exchange
- Compressible for large graphs

## Contents
- [Intro](#user-content-intro)
- [Sections](#user-content-sections)
- [Keys](#user-content-keys)
- [Values](#user-content-values)
- [Graph](#user-content-graph)
- [Nodes](#user-content-nodes)
- [Edges](#user-content-edges)
- [Hyperedges](#user-content-hyperedges)
- [File Extension](#user-content-file-extension)
- [ABNF Grammar](#user-content-abnf-grammar)

## Intro
- RGD is case-sensitive
- Whitespace is a tab (U+0009) or space (U+0020)
- A newline is either LF (U+000A) or CRLF (U+000D U+000A)
- Each line represents a complete expression (no multiline values)
- A RGD file must be a valid UTF-8 encoded Unicode document.

  Specifically this means that a file _as a whole_ must form a
  [well-formed code-unit sequence](https://unicode.org/glossary/#well_formed_code_unit_sequence).
  Otherwise, it must be rejected (preferably) or have ill-formed byte sequences
  replaced with U+FFFD, as per the Unicode specification.

## Sections
A RGD document consists of three potential sections which must appear in order:
1. Graph (optional)
2. Nodes (required)
3. Edges (optional)

An RGD file may be empty, which represents an empty graph.
There are cases where the section headers may not be necessary:
- If the graph section is not present, the `# nodes` header is optional (though recommended)
- If the graph has no edges, the edge section can be omitted entirely

Each section marked with a section header: `#`, followed by its name; e.g.
`# nodes`.
**Exception:** the edges section may omit the name in its section header, but it
is recommended that you include it:

```rgd
A
B
C
#
A -> B
B -> C
```

### Multiple Graphs
Multiple graphs can be listed in the same file, but each must begin with the 
`# graph` section header.

```rgd
# graph
# nodes
{A, B, C}
# edges
A -- B

#graph
#nodes
{A, B, C}
# edges
B -- C
```

## Keys
Keys are used to name nodes, edges, and the lvalue in a key-value pair.
Keys can be quoted or unquoted, but unquoted keys may not contain:
- whitespace
- non-alphanumeric characters except `.`, `-`, `_`
Some examples of valid keys are:

```rgd
first
"second"
3rd
4
"?"
six.th
seven-th
"with spaces"
"<item>"
"🫡"
```

### Key Set
A key set is a comma separated list of valid keys enclosed in matching `{ }`.
A key set represents an unordered grouping of unique keys.

```rgd
{A, B, C}
// ...
{"hello", "there"} -> {"general", "kenobi"}
```

By the uniqueness constraint, keys may not be repeated:

```
//INVALID
{A, A, B, C}
```

Key sets have section-dependent meaning:
- Nodes: a collection of nodes, primarily for bulk description
- Edges: denotes a hyperedge

## Values
Values are meant to be simple to understand by reading them.
A value cannot be a nested key-value list.

### String
There are two ways to express strings: basic, and literal.
All strings must contain only Unicode characters.

**Basic strings** are surrounded by quotation marks (`"`). Any Unicode character
may be used except those that must be escaped: quotation mark, backslash, and
the control characters other than tab (U+0000 to U+0008, U+000A to U+001F,
U+007F).

```rgd
str = "I'm a string. \"You can quote me\". Name\tJos\xE9\nLocation\tSF."
```

For convenience, some popular characters have a compact escape sequence.

```
\b         - backspace       (U+0008)
\t         - tab             (U+0009)
\n         - linefeed        (U+000A)
\f         - form feed       (U+000C)
\r         - carriage return (U+000D)
\e         - escape          (U+001B)
\"         - quote           (U+0022)
\\         - backslash       (U+005C)
\xHH       - unicode         (U+00HH)
\uHHHH     - unicode         (U+HHHH)
\UHHHHHHHH - unicode         (U+HHHHHHHH)
```

Any Unicode character may be escaped with the `\xHH`, `\uHHHH`, or `\UHHHHHHHH`
forms. The escape codes must be Unicode
[scalar values](https://unicode.org/glossary/#unicode_scalar_value).

Keep in mind that all RGD strings are sequences of Unicode characters, _not_
byte sequences. For binary data, avoid using these escape codes. Instead,
external binary-to-text encoding strategies, like hexadecimal sequences or
[Base64](https://www.base64decode.org/), are recommended for converting between
bytes and strings.

All other escape sequences not listed above are reserved; if they are used, RGD
should produce an error.

### Integer

Integers are whole numbers. Positive numbers may be prefixed with a plus sign.
Negative numbers are prefixed with a minus sign.

```rgd
int1 = +99
int2 = 42
int3 = 0
int4 = -17
```

For large numbers, you may use underscores between digits to enhance
readability. Each underscore must be surrounded by at least one digit on each
side.

```rgd
int5 = 1_000
int6 = 5_349_221
// Indian number system grouping
int7 = 53_49_221
// VALID but discouraged
int8 = 1_2_3_4_5
```

Leading zeros are not allowed. Integer values `-0` and `+0` are valid and
identical to an unprefixed zero.

Non-negative integer values may also be expressed in hexadecimal, octal, or
binary. In these formats, leading `+` is not allowed and leading zeros are
allowed (after the prefix). Hex values are case-insensitive. Underscores are
allowed between digits (but not between the prefix and the value).

```rgd
// hexadecimal with prefix `0x`
hex1 = 0xDEADBEEF
hex2 = 0xdeadbeef
hex3 = 0xdead_beef

// octal with prefix `0o`
oct1 = 0o01234567
// useful for Unix file permissions
oct2 = 0o755 

// binary with prefix `0b`
bin1 = 0b11010110
```

Implementations are free to support any integer size. It's recommended that at
least 64-bit signed integers (from −2^63 to 2^63−1) are accepted and handled
losslessly. If an integer cannot be represented losslessly, an error must be
thrown.

### Float

A float consists of an integer part (which follows the same rules as decimal
integer values) followed by a fractional part and/or an exponent part. If both a
fractional part and exponent part are present, the fractional part must precede
the exponent part.

```rgd
// fractional
flt1 = +1.0
flt2 = 3.1415
flt3 = -0.01

// exponent
flt4 = 5e+22
flt5 = 1e06
flt6 = -2E-2

// both
flt7 = 6.626e-34
```

A fractional part is a decimal point followed by one or more digits.

An exponent part is an E (upper or lower case) followed by an integer part
(which follows the same rules as decimal integer values but may include leading
zeros).

The decimal point, if used, must be surrounded by at least one digit on each
side.

```
// INVALID FLOATS
invalid_float_1 = .7
invalid_float_2 = 7.
invalid_float_3 = 3.e+20
```

Similar to integers, you may use underscores to enhance readability. Each
underscore must be surrounded by at least one digit.

```rgd
flt8 = 224_617.445_991_228
```

Float values `-0.0` and `+0.0` are valid and should map according to IEEE 754.

Special float values can also be expressed. They are always lowercase.

```rgd
// infinity
// positive infinity
sf1 = inf
// positive infinity
sf2 = +inf
// negative infinity
sf3 = -inf

// not a number
// actual sNaN/qNaN encoding is implementation-specific
sf4 = nan  
// same as `nan`
sf5 = +nan
// valid, actual encoding is implementation-specific
sf6 = -nan
```

Implementations are free to support any precision level. It's recommended that
at least IEEE 754 binary64 values are supported.

### Boolean

Booleans are just the tokens you're used to. Always lowercase.

```rgd
bool1 = true
bool2 = false
```

### Offset Date-Time

To unambiguously represent a specific instant in time, you may use an
[RFC 3339](https://tools.ietf.org/html/rfc3339) formatted date-time with offset.

```rgd
odt1 = 1979-05-27T07:32:00Z
odt2 = 1979-05-27T00:32:00-07:00
odt3 = 1979-05-27T00:32:00.5-07:00
odt4 = 1979-05-27T00:32:00.999999-07:00
```

For the sake of readability, you may replace the T delimiter between date and
time with a space character (as permitted by RFC 3339 section 5.6).

```rgd
odt5 = 1979-05-27 07:32:00Z
```

One exception to RFC 3339 is permitted: seconds may be omitted, in which case
`:00` will be assumed. The offset immediately follows the minutes.

```rgd
odt6 = 1979-05-27 07:32Z
odt7 = 1979-05-27 07:32-07:00
```

Implementations are required to support at least millisecond precision.
Additional digits of precision may be specified, but if they exceed the
supported precision then the extra digits must be truncated, not rounded.

### Local Date-Time

If you omit the offset from an [RFC 3339](https://tools.ietf.org/html/rfc3339)
formatted date-time, it will represent the given date-time without any relation
to an offset or timezone. It cannot be converted to an instant in time without
additional information. Conversion to an instant, if required, is
implementation-specific.

```rgd
ldt1 = 1979-05-27T07:32:00
ldt2 = 1979-05-27T07:32:00.5
ldt3 = 1979-05-27T00:32:00.999999
```

Seconds may be omitted, in which case `:00` will be assumed.

```rgd
ldt4 = 1979-05-27T07:32
```

Implementations are required to support at least millisecond precision.
Additional digits of precision may be specified, but if they exceed the
supported precision then the extra digits must be truncated, not rounded.

### Local Date

If you include only the date portion of an
[RFC 3339](https://tools.ietf.org/html/rfc3339) formatted date-time, it will
represent that entire day without any relation to an offset or timezone.

```rgd
ld1 = 1979-05-27
```

### Local Time

If you include only the time portion of an
[RFC 3339](https://tools.ietf.org/html/rfc3339) formatted date-time, it will
represent that time of day without any relation to a specific day or any offset
or timezone.

```rgd
lt1 = 07:32:00
lt2 = 00:32:00.5
lt3 = 00:32:00.999999
```

Seconds may be omitted, in which case `:00` will be assumed.

```rgd
lt4 = 07:32
```

Implementations are required to support at least millisecond precision.
Additional digits of precision may be specified, but if they exceed the
supported precision then the extra digits must be truncated, not rounded.

### Array

Arrays are ordered values surrounded by square brackets. Whitespace is ignored.
Elements are separated by commas. Arrays can contain values of the same data
types as allowed in key/value pairs. Values of different types may be mixed.

```rgd
integers = [ 1, 2, 3 ]
colors = [ "red", "yellow", "green" ]
nested_arrays_of_ints = [ [ 1, 2 ], [3, 4, 5] ]
nested_mixed_array = [ [ 1, 2 ], ["a", "b", "c"] ]
string_array = [ "all", 'strings', "are the same", 'type' ]

// Mixed-type arrays are allowed
numbers = [ 0.1, 0.2, 0.5, 1, 2, 5 ]
contributors = ["Foo Bar <foo@example.com>", 1e-10, true]
```

## Graph
The graph section is an optional first section where each line is a key-value
pair; e.g.
```
# graph
diameter = 10
class = "expander"
```
This section can be omitted, but when present, the section header must appear.

## Nodes
Each line in the nodes section describes a node, or set of nodes, present in
the graph; e.g.

```
# nodes
A
B: label = "leader"
C: label = "follower", color = "red"
{D, E, F}: value = 2.5
G: 12
H: [1,2,3,"A","B","C"]
```
Each node is represented by a key with an optional description.
Descriptions must follow a `:` character, and can contain either key-value
pairs, or a single value.

Nodes can be described in bulk using a key set: `{key1, key2, ...}`.
The description provided is distributed among each node in the set.

```rgd
// Each node in the set should receive the same description
{A, B, C}: gang = "lollipop"
```

Every node in the graph **must** appear in the nodes section at least once.
Nodes may appear multiple times, but only if their description is key-valued.
The final aggregated description will be the union of the key-value sets.

It is **INVALID** to duplicate description keys on a node:

```rgd
INVALID:
# nodes
{A, B, C}: label = "bulk"

// This is disallowed because A has already had `label` assigned!
A: label = "individual"
```

### Legacy Nodes
Nodes can also use a legacy format for convenience and compatibility.

```rgd
A label
B
C multiple labels 1.0
```

The legacy format includes some restrictions, and may not be compatible in all
cases.
For example,

```rgd
// INVALID: Legacy descriptions cannot start with `:`
Z : values
// INVALID: Legacy keys cannot use set notation
{A} label
```

## Edges
Edges are described primarily as a connection type between endpoints.
Endpoints are keys or key sets (for hyperedges).
The relationship between the endpoints uses one of the direction specifiers:

- `A -> B` is left-to-right directed
- `A <- B` is right-to-left directed
- `A -- B` is undirected
- `A <> B` is bi-directional

Each edge can be described just like nodes are, with a `:` followed by a valid
description.
Similarly, edges can be listed multiple times if their descriptions are
key-valued, and the final description will be the union of its unique values.
That also means, like with nodes, edges must only describe a description key
once!

```
{A,B,C}
# edges
A -> B: times = [1979-05-27T07:32Z, 1979-05-27T09:28Z, 1979-05-28T01:05:20Z]
C <> B: value = 12.0
C -> B: times = [1979-05-27T07:33Z]
C <- B: times = [1979-05-27T08:01Z]
```

### Edge Identity

Two directed edges are identical if they have the same source endpoint and
target endpoint after direction normalization.
Therefore, `A -> B` and `B <- A` identify the same edge.

Two undirected edges are identical if they contain the same unordered pair of
endpoints.
Therefore, `A -- B` and `B -- A` identify the same edge.

Two bidirectional edges are identical if they contain the same unordered pair of
endpoints.
Therefore, `A <> B` and `B <> A` identify the same edge.

### Legacy Edges
For convenience and compatibility reasons, you can also use legacy edge notation
with certain restrictions.
Legacy edges are assumed undirected unless specified otherwise by the parser,
and may be followed by a space separated list of a subset of value types:
`string`, `array`, `date-time`, or `unquoted-key`.

```
A
B
C
#
// Valid edge with no description
A B

// Valid edge with the description: `key`: d, `key`: 20
// Implementations may choose to coerce types into richer value types
C B d 20

// INVALID edge! Must use direction specifier with key-value descriptions
A C: property = "wrong!"

// Corrected
A -- C: property = "correct!"
```

## Hyperedges
In some contexts sets of nodes are described together as a hyperedge.
Hyperedges are noted as unordered key sets.
They can also be directed!

**Note:** The directed hyperedge `{A,B} -> {C,D}` represents the directed
relationship between the set `{A,B}` and the set `{C,D}`.
It does not represent the pairwise edges `A -> C, A -> D, B -> C, B -> D`.

**Note:** Not all libraries or applications will provide this feature, and those
that do may only provide a subset of the representable structures.
For example, directed hyperedges and mixed edge graphs are useful but uncommon.

```
# nodes
{A, B, C, D, E, F, G}
# edges
{A, B, C}: hyperproperty = 42.0
{B, C} -> {E, F, G}: directed-hyperedge = true
D -- A
```

Hyperedges are not compatible with legacy edge notation.

## File Extension
RGD files should use the `.rgd` extension.

## ABNF Grammar
You can view the spec grammar in the [ABNF file](rgd.abnf).
