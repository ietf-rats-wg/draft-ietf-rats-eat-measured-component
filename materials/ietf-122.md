autoscale: true
footer: IETF 122 Bangkok, RATS WG
slidenumbers: true

# Measured Components
## [draft-ietf-rats-eat-measured-component](https://datatracker.ietf.org/doc/draft-ietf-rats-eat-measured-component)
### IETF 122 Bangkok, RATS WG

---

# Quick Recap

TODO

---

# Changes Since Adoption

Reviews and comments from Giri, Laurence, Carsten, Dionna, MCR, Carl (:pray:)

12 PRs merged, published `ietf-01` and `ietf-02`

^ changes can be grouped in 3 categories:

* (Limited) extensibility additions
* Technical
* Editorial

---

# (Limited) extensibility

* Add a profile-dependent `flags` field [#16](https://github.com/ietf-rats-wg/draft-ietf-rats-eat-measured-component/pull/16)

> _"The format SHOULD also allow a limited amount of extensibility to accommodate profile-specific semantics."_

```
flags-type = eat.JC<bytes8-b64u, bytes8>
```

This field contains at most 64-bit of _profile-defined_ semantics.

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

^ There was a brief discussion in the context of EAT (see EAT#111), but it was ultimately not pursued.
One reason it may have been dropped is that its "natural" scope relates more to the measured component (i.e., a portion of the overall TCB) rather than a global level (the entire TCB). This makes it less suitable as a separate claim.

[^1]: An anti-rollback counter utilized in certain attestation schemes, such as Intel TDX and AMD SEV-SNP.

---

# SVN Options

* As an alternative to `digest`:

```
(digest / svn)
```

* Both `digest` and `svn` are possible, even one alongside the other:

```
non-empty<(?digest, ?svn)>
```

* `digest` is mandatory and possibly complemented by `svn`:

```
(digest, ?svn)
```

---

# [fit] `svn` :interrobang: :rat:

---

# Next Steps

Sort out the SVN topic

WGLC

