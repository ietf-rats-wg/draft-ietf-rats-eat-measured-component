---
v: 3

title: EAT Measured Component
abbrev: "EAT Measured Component"
docname: draft-ietf-rats-eat-measured-component-latest
category: std
consensus: true
submissionType: IETF

ipr: trust200902
area: "Security"
workgroup: "Remote ATtestation ProcedureS"
keyword: [ EAT, measurements, claim, measured, component ]

stand_alone: yes
smart_quotes: no
pi: [toc, sortrefs, symrefs]

author:
 - name: Simon Frost
   organization: Arm
   email: Simon.Frost@arm.com
 - name: Thomas Fossati
   organization: Linaro
   email: Thomas.Fossati@linaro.org
 - name: Hannes Tschofenig
   org: University of Applied Sciences Bonn-Rhein-Sieg
   abbrev: H-BRS
   email: Hannes.Tschofenig@gmx.net
 - name: Henk Birkholz
   org: Fraunhofer SIT
   email: henk.birkholz@ietf.contact

normative:
  RFC7252: coap
  RFC8610: cddl
  RFC9165: cddlplus
  RFC9741: cddlctls
  RFC9711: rats-eat

informative:
  RFC3444: models
  RFC9783: psa-token
  RFC9393: coswid
  UEFI2:
    title: "Unified Extensible Firmware Interface (UEFI) Specification"
    author:
      org: "UEFI Forum, Inc."
    date: Aug 29, 2022
    version: Release 2.10
    format:
      PDF: https://uefi.org/sites/default/files/resources/UEFI_Spec_2_10_Aug29.pdf
  TBBR-CLIENT:
    title: "Trusted Board Boot Requirements Client (TBBR-CLIENT) Armv8-A"
    author:
      org: "Arm Ltd"
    date: Sep 20, 2018
    target: https://developer.arm.com/documentation/den0006
    seriesinfo:
      ARM: DEN0006D

entity:
  SELF: "RFCthis"

--- abstract

The term "measured component" refers to an object within the attester's target environment whose state can be inspected and, typically, digested.
A digest is computed through a cryptographic hash function.
Examples of measured components include firmware stored in flash memory, software loaded into memory at start time, data stored in a file system, or values in a CPU register.
This document provides the information model for the "measured component" and two associated data models.
This separation is intentional: the JSON and CBOR serializations, coupled with the media types and associated CoAP Content-Formats, enable the immediate use of the semantics within the EAT framework.
Meanwhile, the information model can be reused in future specifications to provide additional serializations, for example using ASN.1.

--- middle

# Introduction

{{Section 4.2.16 of -rats-eat}} defines a `Measurements` claim that:

> "[c]ontains descriptions, lists, evidence or measurements of the software that exists on the entity or any other measurable subsystem of the entity."

This claim allows for different measurement formats, each identified by a different CoAP Content-Format ({{Section 12.3 of -coap}}).
Currently, the only specified format is CoSWID of type "evidence", as per {{Section 2.9.4 of -coswid}}.
However, CoSWID is not suitable for measurements that cannot be anchored to a file system, such as those in early boot environments.
To address this gap, this document introduces a "measured component" format that can be used with the EAT `Measurements` claim alongside or instead of CoSWID.

The term "measured component" refers to any measurable object on a target environment, that is, an object whose state can be sampled and, possibly, digested.
This includes, for example: the invariant part of a firmware component that is loaded in memory at startup time, a run-time integrity check (RTIC), a file system object, or a CPU register.

This document provides the information model for the "measured component" and two associated data models {{-models}}.
This separation is intentional: the JSON and CBOR serializations, coupled with the media types and associated CoAP Content-Formats, enable the immediate use of the semantics within the EAT framework.
Meanwhile, the information model can be reused in future specifications to provide additional serializations, for example using ASN.1.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

In this document, CDDL {{-cddl}} {{-cddlplus}} {{-cddlctls}} is used to describe the data formats.

# Information Model {#measured-component}

This section presents the information model of a "measured component".

A "measured component" information element includes the component's sampled state (in digested or raw form) along with metadata that helps in identifying the component.
Optionally, any entities responsible for signing the installed component can also be specified.

The information model of a "measured component" is described in {{tab-mc-info-elems}}.

| IE | Description | Requirement Level |
|----|-------------|-------------------|
| Component Name | The name given to the measured component. It is important that this name remains consistent across different releases to allow for better tracking of the same measured item across updates. When combined with a consistent versioning scheme, it enables better signaling from the appraisal procedure to the relying parties. | REQUIRED |
| Component Version | A value representing the specific release or development version of the measured component.  Using [Semantic Versioning](https://semver.org/spec/v2.0.0.html) is RECOMMENDED. | OPTIONAL |
| Digested or Raw Value | Either the raw value or the digested value of the measured component. | REQUIRED |
| Digest Algorithm | Hash algorithm used to compute the Digest Value. | REQUIRED only if the value is in the digested form |
| Signers | One or more unique identifiers of entities signing the component that is measured. | OPTIONAL |
{: #tab-mc-info-elems title="Measured Component Information Elements"}

The format SHOULD also allow a limited amount of extensibility to accommodate profile-specific semantics.

# Data Model

This section presents a JSON and CBOR data model that implements the information model outlined in {{measured-component}}.

The data model is inspired by the "PSA software component" claim ({{Section 4.4.1 of -psa-token}}), which has been refactored to take into account the recommendations about the design of new EAT claims described in {{Appendix E of -rats-eat}}.

CDDL is used to express rules and constraints of the data model for both JSON and CBOR.
These rules must be strictly followed when creating or validating "measured component" data items.
When there is variation between CBOR and JSON, the `JC<>` CDDL generic defined in {{Appendix D of -rats-eat}} is used.

## Common Types

The following three basic types are used at various places within the measured component data model:

~~~ cddl
{::include cddl/common-types.cddl}
~~~

## The `digest` Type {#digest}

A digest represents the result of a hashing operation together with the hash algorithm used.
The type of the digest algorithm identifier can be either `int` or `text` and is interpreted according to the {{!IANA.named-information}} registry.
Specifically, `int` values are matched against "ID" entries and `text` values are matched against "Hash Name String" entries.
Whenever possible, using the `int` encoding is RECOMMENDED.

~~~ cddl
digest = [
  alg: (int / text)
  val: digest-value-type
]

digest-value-type = eat.JC<bytes-b64u, bytes>
~~~

## The `measured-component` Data Item

The `measured-component` data item is as follows:

~~~ cddl
{::include cddl/mc.cddl}

{::include cddl/labels.cddl}
~~~

The members of the `measured-component` CBOR map / JSON object are:

{:vspace}
`"id"` (index 1):
: The measured component identifier encoded according to the format described in {{component-id}}.

`"measurement"`:
: Either a digest value and algorithm (index 2), encoded using the digest format ({{digest}}), or the "raw" measurement (index 5), encoded as a byte string.

`"signers"` (index 3):
: One or more signing entities, see {{signer}}.

`"flags"` (index 4):
: a 64-bit field with profile-defined semantics, see {{profile-flags}}.

### Component Identifier {#component-id}

The `component-id` data item is as follows:

~~~ cddl
{::include cddl/component-id.cddl}
~~~
{:vspace}

`name`
: A string that provides a human readable identifier for the component in question.  Format and adopted conventions depend on the component type.

`version`
: A compound `version` data item that reuses encoding and semantics of {{-rats-eat}} `sw-version-type`.

### Signer {#signer}

A signer is an entity that digitally signed the component that is measured.
Typically, the signature is verified during installation or when the measured component is executed by the boot ROM, operating system, or application launcher.
For example, as in UEFI Secure Boot {{UEFI2}} and Arm Trusted Board Boot {{TBBR-CLIENT}}.
Another example may be the controlling entity in an app store.
It is important to note that a signer is different from the identity of the manufacturer of the component, such as would be found in a manifest like a payload CoSWID.

A signer is identified using a public key.
It could be an X.509 certificate, a raw public key, a public key thumbprint, or some other identifier that can be uniquely associated with the signing entity.
In some cases, multiple parties may need to sign a component to indicate their endorsement or approval.
This could include roles such as a firmware update system, fleet owner, or third-party auditor.
The specific purpose of each signature may depend on the deployment, and the order of signers within the array could indicate meaning.

If an EAT profile ({{Section 6 of -rats-eat}}) uses measured components, it MUST specify whether the `signers` field is used.
If it is used, the profile MUST also specify what each of the entries in the `signers` array represents, and how to interpret the corresponding `signer-id-type`.

The `signer-id-type` is defined as follows:

~~~ cddl
{::include cddl/signer.cddl}
~~~

### Profile-specific Flags {#profile-flags}

This field contains at most 64-bit of profile-defined semantics.
It can be used to carry information in fixed-size chunks, such as a bit mask or a single value within a predetermined set of codepoints.
Regardless of its internal structure, the size of this optional field is exactly 8 bytes.

The `flags-type` is defined as follows:

~~~ cddl
{::include cddl/profile-flags.cddl}
~~~

If an EAT profile ({{Section 6 of -rats-eat}}) uses measured components, it MUST specify whether the `profile-flags` field is used.
If it is used, the profile MUST also specify how to interpret the 64 bits.

## EAT `measurements-format` Extensions

The CDDL in {{fig-eat-plug}} extends the `$measurements-body-cbor` and `$measurements-body-json` EAT sockets to add support for `measured-component`s to the `Measurements` claim.

~~~ cddl
{::include cddl/eat-plug.cddl}
~~~
{: #fig-eat-plug title="EAT measurements-format Extensions"}

Each socket is extended with two new types: a "native" representation that is used when `measured-component` and the EAT have the same serialization (e.g., they are both CBOR), and a "tunnel" representation that is used when the serializations differ.

## `measurements-format` for CBOR EAT

The entries in {{tab-mf-cbor}} are the allowed `content-type` / `content-format` pairs when the `measured-component` is carried in a CBOR EAT.

Note the use of the "native" and "tunnel" formats from {{fig-eat-plug}}, and how the associated CoAP Content-Format is used to describe the original serialization.

| `content-type` (CoAP C-F equivalent) | `content-format` |
|--|--|
| `application/measured-component+cbor` | `mc-cbor` |
| `application/measured-component+json` | `mc-json` |
{: #tab-mf-cbor title="measurement-format for EAT CWT"}

## `measurements-format` for JSON EAT

{{tab-mf-json}} is the equivalent of {{tab-mf-cbor}} for JSON-serialized EAT.

| `content-type` (CoAP C-F equivalent) | `content-format` |
|--|--|
| `application/measured-component+json` | `mc-json` |
| `application/measured-component+cbor` | `tstr .b64u mc-cbor` |
{: #tab-mf-json title="measurement-format for EAT JWT"}

# EAT Profiles and Measured Components

The semantics of the signers and profile flags fields are defined by the applicable EAT profile, i.e., the profile of the wrapping EAT.

If the profile of the EAT is not known to the consumer and one or more Measured Components within that EAT include signers and/or profile flags, the consumer MUST reject the EAT.

# Examples

The example in {{ex-1}} is a digested measured component with all the fields populated.

~~~ cbor-edn
{::include cddl/ex1.diag}
~~~
{: #ex-1 title="Complete Measured Component"}

The example in {{ex-eat-1}} is the same measured component as above but used as the format of a `measurements` claim in a EAT claims-set.

The example uses TBD1 as the `content-type` value of the `measurements-format` entry.
(This will change to the value assigned by IANA to the `mc+cbor` Content-Format.)

Note that the array contains only one measured component, but additional entries could be added if the measured TCB is made of multiple, individually measured components.

~~~ cbor-edn
{::include cddl/eat-ex1.diag.in}
~~~
{: #ex-eat-1 title="EAT Measurements Claim using a Measured Component (CBOR)"}

The example in {{ex-eat-2}} illustrates the inclusion of a JSON measured component inside a JSON EAT.

The example uses TBD2 as the `content-type` value of the `measurements-format` entry.
(This will change to the value assigned by IANA to the `mc+json` Content-Format.)

~~~ cbor-edn
{::include-fold cddl/eat-ex1-json.diag.in}
~~~
{: #ex-eat-2 title="EAT Measurements Claim using a Measured Component (JSON)"}

The example in {{ex-2}} is a measured component representing a boot loader identified by its path name:

~~~ cbor-edn
{::include cddl/ex2.diag}
~~~
{: #ex-2 title="Digested Measured Component using File Path as Identifier"}

The example in {{ex-3}} is a raw measured component.

~~~ cbor-edn
{::include cddl/ex3.diag}
~~~
{: #ex-3 title="Raw Measured Component"}

# Security and Privacy Considerations {#seccons}

Please review {{Sections 9.1 (Claim Trustworthiness), 9.4 (Multiple EAT Consumers) and 9.5 (Detached EAT Bundle Digest Security Considerations) of -rats-eat}}; these considerations apply to this document as well.
Note that similar security and privacy considerations may apply when the Measured Component information model is serialized using different data models than the ones specified in this document.

The Component Name and Component Version can give an attacker detailed information about the software running on a device and its configuration settings.
This information could offer an attacker valuable insights.
Additionally, the stability requirement of the Component Name could potentially allow for tracking.

If the component measurement is digested, the digest must be computed using a strong cryptographic hash function.

# IANA Considerations

[^rfced] replace "{{&SELF}}" with the RFC number assigned to this document.

## Media Types Registrations

IANA is requested to add the following media types to the "Media Types" registry {{!IANA.media-types}}.

| Name | Template | Reference |
|-----------------|-------------------------|-----------|
| `mc+cbor` | `application/measured-component+cbor` | {{&SELF}} |
| `mc+json` | `application/measured-component+json` | {{&SELF}} |
{: #tab-mc-regs title="Measured Component Media Types"}

### `application/measured-component+cbor`

{:compact}
Type name:
: application

Subtype name:
: measured-component+cbor

Required parameters:
: n/a

Optional parameters:
: n/a

Encoding considerations:
: binary (CBOR)

Security considerations:
: {{seccons}} of {{&SELF}}

Interoperability considerations:
: n/a

Published specification:
: {{&SELF}}

Applications that use this media type:
: Attesters, Verifiers and Relying Parties

Fragment identifier considerations:
: The syntax and semantics of fragment identifiers are as specified for "application/cbor". (No fragment identification syntax is currently defined for "application/cbor".)

Person & email address to contact for further information:
: RATS WG mailing list (rats@ietf.org)

Intended usage:
: COMMON

Restrictions on usage:
: none

Author/Change controller:
: IETF

Provisional registration:
: no

### `application/measured-component+json`

{:compact}
Type name:
: application

Subtype name:
: measured-component+json

Required parameters:
: n/a

Optional parameters:
: n/a

Encoding considerations:
: binary (JSON is UTF-8-encoded text)

Security considerations:
: {{seccons}} of {{&SELF}}

Interoperability considerations:
: n/a

Published specification:
: {{&SELF}}

Applications that use this media type:
: Attesters, Verifiers and Relying Parties

Fragment identifier considerations:
: The syntax and semantics of fragment identifiers are as specified for "application/json". (No fragment identification syntax is currently defined for "application/json".)

Person & email address to contact for further information:
: RATS WG mailing list (rats@ietf.org)

Intended usage:
: COMMON

Restrictions on usage:
: none

Author/Change controller:
: IETF

Provisional registration:
: no

## Measured Component Content-Format Registrations

IANA is requested to register two Content-Format numbers in the "CoAP Content-Formats" sub-registry, within the "Constrained RESTful Environments (CoRE) Parameters" Registry {{!IANA.core-parameters}}, as follows:

| Content-Type | Content Coding | ID | Reference |
| application/measured-component+cbor | - | TBD1 | {{&SELF}} |
| application/measured-component+json | - | TBD2 | {{&SELF}} |

If possible, TBD1 and TBD2 should be assigned in the 256..9999 range.

--- back

# Open Issues

The list of currently open issues for this documents can be found at [](https://github.com/ietf-rats-wg/draft-ietf-rats-eat-measured-component/issues).

<cref>Note to RFC Editor: please remove before publication.</cref>

# Acknowledgments
{:numbered="false"}

The authors would like to thank
Carl Wallace,
Carsten Bormann,
Dionna Glaze,
Giridhar Mandyam,
{{{Ionuț Mihalcea}}},
Laurence Lundblade,
Michael Richardson
and
Usama Sardar
for providing comments, reviews and suggestions that greatly improved this document.

[^rfced]: RFC Editor:
