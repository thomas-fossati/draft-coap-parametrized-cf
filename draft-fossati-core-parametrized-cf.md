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
Parametrized-Multivalued-Accept, that build upon such data item to work around
some of the limitations of the existing Accept and Content-Format Options.

--- middle

# Introduction

CoAP squashes the combination of a media type, media type parameters and
content coding into a single Content-Format number.  This number is carried in
the Content-Format and Accept Options.  For example, see {{Section 16.10 of
RFC8152}}.

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
* Parametrized-Multivalued-Accept ({{sec-pma-option}})

The latter also works around a well known limitation of the Accept Option
allowing more than one Content-Format per request.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Parametrized Content-Format {#sec-pcf}

The Parametrized Content-Format is a CBOR data item defined in {{cddl}}.

~~~cddl
{::include pcf.cddl}}
~~~
{: #cddl title="CDDL for the Media Type Parameters Option"}

## Requirements

> The term "additional" has not been used casually in the preceding paragraph: if
> the Content-Format already encodes one or more parameters, the values in the
> parameter can only extend the set.
> Furthermore, the latter is restricted to contain only parameters that are not
> already encoded in the sibling Content-Format.

* MTP must have a sibling C-F (there is no default if C-F is absent)
  * if not, TBD-ACTION
* the intersection between parameters in MTP and C-F must be empty
  * if not, TBD-ACTION

# Parametrized Content-Format Option {#sec-pcf-option}

| Number | C | U | N | R | Name | Format | Length | Default |
| ------ | - | - | - | - | ---- | ------ | ------ | ------- |
| TBD    |   |   |   |   | Parametrized Content-Format Option | opaque (see TBD) | | none |


# Parametrized Multivalued Accept Option {#sec-pma-option}

| Number | C | U | N | R | Name | Format | Length | Default |
| ------ | - | - | - | - | ---- | ------ | ------ | ------- |
| TBD    |   |   |   |   | Parametrized Content-Format Option | opaque (see TBD) | | none |


# Security Considerations

TODO Security

# IANA Considerations

TODO IANA


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
