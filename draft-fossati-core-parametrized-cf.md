---
title: "Parametrized Content-Format for CoAP"
category: std

docname: draft-fossati-core-parametrized-cf-latest
submissiontype: IETF
number:
date:
consensus: true
v: 3
area: "Applications and Real-Time"
workgroup: "Constrained RESTful Environments"
keyword:
venue:
  group: "Constrained RESTful Environments"
  type: "Working Group"
  mail: "core@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/core/"
  github: "thomas-fossati/draft-core-mtp-option"
  latest: "https://thomas-fossati.github.io/draft-core-mtp-option/draft-fossati-core-mtp-option.html"

author:
 -
    fullname: Thomas Fossati
    organization: arm
    email: thomas.fossati@arm.com
 -
    fullname: Henk Birkholz
    organization: Fraunhofer SIT
    email: henk.birkholz@sit.fraunhofer.de

normative:

informative:
  RFC8152:

--- abstract

This document specifies a "parametrized" CoAP Content-Format data item that
allows supplementing a Content-Format with additional media type parameters.

This document also defines two new CoAP Options, Parmetrized-Content-Format and
Parametrized-Multi-Valued-Accept, that build upon said data item to work around
some of the limitations of the existing Accept and Content-Format Options.

--- middle

# Introduction

CoAP squashes the combination of a media type, media type parameters and
content coding into a single Content-Format number.  This number is carried in
the Content-Format and Accept Options.  (For an example, see {{Section 16.10 of
RFC8152}}.)

This compression strategy is ideal in cases where the set of possible
combinations is known upfront and has small cardinality.  However, it lacks the
flexibility to deal smoothly with situations where the number of combinations
can grow unbounded.

An example is {{?I-D.lundblade-rats-eat-media-type}}, in which the "profile"
media type parameter can carry a number of different values that are constantly
minted through a loosely regulated process.

To avoid the combinatorial explosion that derives from such premises, this
document defines the "parametrized" Content-Format data item ({{sec-pcf}}) as a
mechanism to enrich a given Content-Format with additional media type
parameters.

Two new CoAP Options that build upon such data item are also defined:

* Parametrized-Content-Format ({{sec-pcf-option}})
* Parametrized-Multi-Valued-Accept ({{sec-pma-option}})

The latter also works around the limited content negotiation capabilities of
the Accept Option by offering more than one Content-Format per request.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Parametrized Content-Format {#sec-pcf}

The Parametrized Content-Format is a CBOR data item defined by the CDDL in
{{cddl-pcf}}.

The first element in the tuple is the Content-Format identifier, followed by
one or more name-value pairs representing the additional media type parameters.

~~~cddl
parametrized-content-format = [
  content-format,
  + [ parameter-name, parameter-value ]
]

content-format = 0..65535

parameter-name = textual / numeric
parameter-value = any

textual = text .abnf ("parameter-name" .det RFC6838-parameter-name)
numeric = int

RFC6838-parameter-name = '
  parameter-name = restricted-name

  restricted-name = restricted-name-first *126restricted-name-chars
  restricted-name-first  = ALPHA / DIGIT
  restricted-name-chars  = ALPHA / DIGIT / "!" / "#" /
                           "$" / "&" / "-" / "^" / "_"
  restricted-name-chars =/ "." ; Characters before first dot always
                               ; specify a facet name
  restricted-name-chars =/ "+" ; Characters after last plus always
                               ; specify a structured syntax suffix

  ALPHA         =  %x41-5A / %x61-7A  ; A-Z / a-z
  DIGIT         =  %x30-39            ; 0-9
'
~~~
{: #cddl-pcf title="CDDL for the Parametrized Content-Format"}

[^TBD1]

[^TBD1]: TODO describe use of numeric identifiers as alias for parameter names.

## Requirements

The list that follows details the semantic requirements that a Parametrized
Content-Format data item must satisfy:

* The intersection between the media parameters already encoded in the
  Content-Format identifier and the set of parameters carried in the name-value
  pairs of the Parametrized Content-Format MUST be empty.

* Each name-value pair MUST be a registered parameter for the media type.

If any of the conditions listed above is not met, the entire data item is
considered invalid and MUST NOT be processed further.

## Examples

~~~cbor-diag
[
  1,
  [ "p1", "a-string-value" ],
  [ "p2", 128 ]
]
~~~
{: #ex-pcf-1 title="Example #1" }

# Parametrized Content-Format Option {#sec-pcf-option}

| Number | C | U | N | R | Name | Format | Length | Default |
| ------ | - | - | - | - | ---- | ------ | ------ | ------- |
| TBD    |   |   |   |   | Parametrized Content-Format Option | See {{cddl-pcf-opt-fmt}} | | none |
{: #tbl-pcf-opt title="Parametrized Content-Format Option"}

The Parametrized Content-Format Option carries a CBOR-encoded Parametrized
Content-Format data item.

~~~cddl
pcf-option-fmt = bytes .cbor parametrized-content-format
~~~
{: #cddl-pcf-opt-fmt title="Parametrized Content-Format Option Format"}

# Parametrized Multi-Valued Accept Option {#sec-pma-option}

| Number | C | U | N | R | Name | Format | Length | Default |
| ------ | - | - | - | - | ---- | ------ | ------ | ------- |
| TBD    |   |   |   |   | Parametrized Multi-Valued Accept Option | See {{cddl-pmva-opt-fmt}} | | none |
{: #tbl-pmva-opt title="Parametrized Multi-Valued Accept Option"}

The Parametrized Multi-Valued Accept Option carries a single CBOR-encoded
Parametrized Content-Format data item or two or more Parametrized
Content-Format data items as a CBOR array.

~~~cddl
one-or-more<T> = T / [ 2* T ]

pmva-option-fmt = bytes .cbor one-or-more<parametrized-content-format>
~~~
{: #cddl-pmva-opt-fmt title="Parametrized Multi-Valued Accept Option Format"}

# Security Considerations

TODO Security

# IANA Considerations

TODO IANA

--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
