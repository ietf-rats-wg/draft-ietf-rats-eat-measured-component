autoscale: true
footer: IETF 123 Madrid, RATS WG
slidenumbers: true
theme: Plain Thomas

# Measured Components
## [draft-ietf-rats-eat-measured-component](https://datatracker.ietf.org/doc/draft-ietf-rats-eat-measured-component)
### IETF 123 Madrid, RATS WG

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

# Changes Since Bangkok

* Support "raw" values alongside "digested" values

```
 	   {	
 	     / id / 1: [	
 	       / name / "hardware-config"	
 	     ],	
 	     / measurement / 5: h'4f6d616861'	
 	   }	
```

* Decided not to go ahead with SVN 

* Feature complete

---

# Next Steps

WGLC
