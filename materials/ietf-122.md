autoscale: true
footer: IETF 122 Bangkok, RATS WG
slidenumbers: true
theme: Plain Thomas

# Measured Components
## [draft-ietf-rats-eat-measured-component](https://datatracker.ietf.org/doc/draft-ietf-rats-eat-measured-component)
### IETF 122 Bangkok, RATS WG

---

# Quick Recap

Extends the [EAT Measurements](https://www.ietf.org/archive/id/draft-ietf-rats-eat-31.html#section-4.2.16) claim

It can be used in addition to or as an alternative to [CoSWID](https://www.rfc-editor.org/rfc/rfc9393.html)

_"Measured component"_ $$\equiv$$ any measurable object on a target environment:

* Firmware component loaded in memory at boot time (typical)
* Run-time integrity check (RTIC)
* File
* CPU register

^ RTIC(RunTime Integrity Check) feature is to protect Linux kernel at runtime.
This relocates some of the security sensitive kernel structures (e.g., `selinux_state`) to a separate
RTIC specific page.  This is to enable monitoring and protection of these
kernel assets from a higher exception level(EL) against any unauthorized
changes.

---

# Changes Since Adoption

Reviews and comments from Giri, Laurence, Carsten, Dionna, MCR, and Carl (:pray:)

12 PRs merged, published `ietf-01` and `ietf-02`

^ changes can be grouped in 3 categories:

* Add (limited) extensibility
* Technical
* Editorial

---

# (Limited) extensibility

_"The format SHOULD also allow a limited amount of extensibility to accommodate profile-specific semantics."_

* Add a profile-dependent `flags` field [#16](https://github.com/ietf-rats-wg/draft-ietf-rats-eat-measured-component/pull/16)

```
  flags-type = eat.JC<bytes8-b64u, bytes8>
```

^This field contains at most 64-bit of _profile-defined_ semantics

* Bit mask
* Single value within a predetermined set of codepoints

^Regardless of its internal structure, the size is exactly 8 bytes

---

# (Unlimited) extensibility

A slightly more radical approach:

```
any-defined-by-profile-type = eat.JC<bytes-b64u, bytes>
```

*Any* _profile-defined_ semantics, e.g.:
```
  flags-type = bytes .cbor my-extension-map
  flags-type = text .json my-extension-map
```

^this way you can have the bitmask or the virtually unlimited codepoints space
but you can also have structured information that is still opaque to the
generic EAT processor and is interpreted by the profile specific processor

RATS WG, is it worth it?

---

# Technical

* Remove extra `b64u`-wrapping for JSON in CBOR (Carsten) [#17](https://github.com/ietf-rats-wg/draft-ietf-rats-eat-measured-component/pull/17)
* Add JSON examples [#20](https://github.com/ietf-rats-wg/draft-ietf-rats-eat-measured-component/pull/20)
* Fix wrong `flags` size (Dionna) [#26](https://github.com/ietf-rats-wg/draft-ietf-rats-eat-measured-component/pull/26)
* Describe behaviour when the profile is not known (Carl) [#29](https://github.com/ietf-rats-wg/draft-ietf-rats-eat-measured-component/pull/29)

---

# Editorial

* Clarification around `signers` by Laurence [#13](https://github.com/ietf-rats-wg/draft-ietf-rats-eat-measured-component/pull/13)
* Editorial suggestions from Carl [#27](https://github.com/ietf-rats-wg/draft-ietf-rats-eat-measured-component/pull/27)
* Text changes in the abstract [#23](https://github.com/ietf-rats-wg/draft-ietf-rats-eat-measured-component/pull/23)
* Slightly rephrase the security considerations [#24](https://github.com/ietf-rats-wg/draft-ietf-rats-eat-measured-component/pull/24)
* Add example using file system object as `component-id` [#25](https://github.com/ietf-rats-wg/draft-ietf-rats-eat-measured-component/pull/25)

---

# Open Issues

Two remaining issues:

* [#21](https://github.com/ietf-rats-wg/draft-ietf-rats-eat-measured-component/issues/21) Purely editorial (CDDL import)
* [#10](https://github.com/ietf-rats-wg/draft-ietf-rats-eat-measured-component/issues/10) Add SVN?

---

# SVN

There are cases where the measurement includes a Security Version Number (SVN)[^1].

Discussed in the context of EAT but not pursued.

^ There was a brief discussion in the context of EAT (see EAT#111), but it was
ultimately not pursued.  One reason it may have been dropped is that its
"natural" scope relates more to the measured component (i.e., a portion of the
overall TCB) rather than a global level (the entire TCB). This makes it less
suitable as a separate claim.

[^1]: An anti-rollback counter utilized in certain attestation schemes, such as Intel TDX and AMD SEV-SNP.

---

# SVN Options

^we have three options


A) As an alternative to `digest`:

```
    (digest / svn)
```

B) Both `digest` and `svn` are possible, even one alongside the other:

```
    non-empty<(?digest, ?svn)>
```

C) `digest` is mandatory and possibly complemented by `svn`:

```
    (digest, ?svn)
```

^if people want it we can do it, otherwise we do nothing and move on

---

# Next Steps

Sort out the SVN topic

WGLC

