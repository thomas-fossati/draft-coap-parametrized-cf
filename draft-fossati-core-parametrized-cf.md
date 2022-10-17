---
v: 3

title: "Parametrized Content-Format for CoAP"
category: std

docname: draft-fossati-core-parametrized-cf-latest
submissiontype: IETF
consensus: true
area: "Applications and Real-Time"
workgroup: "Constrained RESTful Environments"
keyword:
venue:
  group: "Constrained RESTful Environments"
  type: "Working Group"
  mail: "core@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/core/"
  github: "thomas-fossati/draft-coap-parametrized-cf"
  latest: "https://thomas-fossati.github.io/draft-coap-parametrized-cf/draft-fossati-core-parametrized-cf.html"

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
  RFC7252: coap
  RFC8610: cddl
  RFC9165: cddlplus
  STD94:
    -: cbor
    =: RFC8949
  IANA.core-parameters: iana-core

informative:
  STD96:
    -: cose
    =: RFC9052
  I-D.ietf-core-coral: coral
  I-D.lundblade-rats-eat-media-type: eat-mt

entity:
  SELF: "RFCthis"

--- abstract

This document specifies a "parametrized" CoAP Content-Format data item that
allows supplementing a Content-Format with additional media type parameters.

This document also defines two new CoAP Options, Parmetrized-Content-Format and
Parametrized-Multi-Valued-Accept, that build upon the "parametrized"
Content-Format data item to work around some of the limitations of the existing
Accept and Content-Format Options.

--- middle

# Introduction

CoAP squashes the combination of a media type, media type parameters and
content coding into a single Content-Format number.  (For an example, see Table
2 in {{Section 2 of -cose}}.)  This number is carried in the Content-Format and
Accept Options.

Such compression strategy is ideal in cases where the set of possible parameters
combinations is known upfront and has small cardinality.  However, it lacks the
flexibility to deal smoothly with situations where the number of combinations
can grow unbounded.

An example is {{-eat-mt}}, in which the "profile" media type parameter can
carry a number of different values that are constantly minted through a loosely
regulated process.  Another example is content negotiation of CoRAL {{-coral}}
profiles.

To avoid the combinatorial explosion that derives from such premises, this
document defines the "parametrized" Content-Format data item ({{sec-pcf}}) as a
mechanism to enrich a given Content-Format with additional media type
parameters.

Two new CoAP Options that build upon such data item are also defined:

* Parametrized-Content-Format ({{sec-pcf-option}})
* Parametrized-Multi-Valued-Accept ({{sec-pma-option}})

The latter also works around the limited content negotiation capabilities of
the CoAP Accept Option by allowing to accept more than one Content-Format per
request.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

In this document, the structure of data is specified in CDDL {{-cddl}}
{{-cddlplus}}.

The examples in {{sec-examples}} use CBOR diagnostic notation defined in
{{Section 8 of -cbor}} and {{Appendix G of -cddl}}.

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

[^TBD1]: TODO describe use of numeric identifiers for parameter name aliasing
         (requires a new registry).

## Requirements

The list that follows details the semantic requirements that a Parametrized
Content-Format data item must satisfy:

* The intersection between the media parameters already encoded in the
  Content-Format identifier and the set of parameters carried in the name-value
  pairs of the Parametrized Content-Format MUST be empty.

* Each name-value pair MUST be a registered parameter for the media type.

If any of the conditions listed above is not met, the entire data item is
considered invalid and MUST NOT be processed further.

## Examples {#sec-examples}

~~~cbor-diag
[
  65000,
  [ "p1", "a-string-value" ],
  [ "p2", 128 ]
]
~~~
{: #ex-pcf-1 title="Content-Format with two paramters" }

# Parametrized Content-Format Option {#sec-pcf-option}

| Number | C | U | N | R | Name | Format | Length | Default |
| ------ | - | - | - | - | ---- | ------ | ------ | ------- |
| TBD24  |   |   |   |   | Parametrized Content-Format Option | See {{cddl-pcf-opt-fmt}} | | none |
{: #tbl-pcf-opt title="Parametrized Content-Format Option"}

The Parametrized Content-Format Option carries a CBOR-encoded Parametrized
Content-Format data item.

~~~cddl
pcf-option-fmt = bytes .cbor parametrized-content-format
~~~
{: #cddl-pcf-opt-fmt title="Parametrized Content-Format Option Format"}

The semantic is identical to the Content-Format Option described in {{Section
5.10.3 of -coap}}.

# Parametrized Multi-Valued Accept Option {#sec-pma-option}

| Number | C | U | N | R | Name | Format | Length | Default |
| ------ | - | - | - | - | ---- | ------ | ------ | ------- |
| TBD13  | x |   |   |   | Parametrized Multi-Valued Accept Option | See {{cddl-pmva-opt-fmt}} | | none |
{: #tbl-pmva-opt title="Parametrized Multi-Valued Accept Option"}

The Parametrized Multi-Valued Accept Option carries either a single
CBOR-encoded `pa-content-format` data item or two or more `pa-content-format`
items wrapped in a CBOR array.  In turn, each `pa-content-format` can be either
a plain Content-Format or a Parametrized Content-Format as described in
{{cddl-pmva-opt-fmt}}.

~~~cddl
pa-content-format = content-format / parametrized-content-format

one-or-more<T> = T / [ 2* T ]

pmva-option-fmt = bytes .cbor one-or-more<pa-content-format>
~~~
{: #cddl-pmva-opt-fmt title="Parametrized Multi-Valued Accept Option Format"}

The semantic is identical to the Accept Option described in {{Section 5.10.4 of
-coap}}, except for the ability to list more than one acceptable (parametrized)
Content-Format, which is key to enable finer-grained content negotiation.

The Content-Formats are listed in order of preference.  If more than one match
is found, the entry with the lowest index in the array MUST be selected.

# Security Considerations

The security considerations in {{Section 11.1 of -coap}} related to the parsing
of protocol elements apply.

The security considerations in {{Section 11.3 of -coap}} related to
amplification risks apply.

TODO expand

# IANA Considerations

[^to-be-removed]

[^to-be-removed]: RFC Editor: please replace {{&SELF}} with this RFC number and
                  remove this note.

## CoAP Option Numbers Registry

IANA is requested to add the entries from {{tbl-iana-req}} to the CoAP Option
Numbers sub-registry of the Constrained RESTful Environments (CoRE) Parameters
{{-iana-core}} registry:

| Number | Name | Reference |
| ------ | ---- | --------- |
| TBD13  | Parametrized Multi-Valued Accept Option | {{sec-pma-option}} of {{&SELF}} |
| TBD24  | Parametrized Content-Format Option | {{sec-pcf-option}} of {{&SELF}} |
{: #tbl-iana-req title="New Options"}

This document suggests 13 (TBD13) and 24 (TBD24) as values to be assigned for
the new option numbers.

--- back

# Acknowledgments
{:numbered="false"}

Thank you
Carsten Bormann,
{{{Christian Amsüss}}},
and
Marco Tiloca
for the useful comments and suggestions.

