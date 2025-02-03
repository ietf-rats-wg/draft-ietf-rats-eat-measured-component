SHELL := /bin/bash

%.cbor: %.diag ; diag2cbor.rb $< > $@
%.cddl: %.cddlc ; cddlc -2tcddl $< > $@

CDDL := measured-component.cddl
DIAG_EXAMPLES := $(wildcard ex*.diag)
CBOR_EXAMPLES := $(DIAG_EXAMPLES:.diag=.cbor)

all: check-schema check-examples mceat-check-examples

check-schema: $(CDDL) ; cddl $< g 10
.PHONY: check-schema

CLEANFILES += $(CDDL)
CLEANFILES += $(CBOR_EXAMPLES)
CLEANFILES += $(wildcard *.pretty)
CLEANFILES += $(MCEAT_CDDL)
CLEANFILES += $(MCEAT_CBOR_EXAMPLES)

clean:: ; -rm -f $(CLEANFILES)
.PHONY: clean

check-examples: $(CBOR_EXAMPLES) $(CDDL)
	@for f in $(CBOR_EXAMPLES); do \
		echo ">> validating $$f against $(CDDL)" ; \
		cddl $(CDDL) validate $$f &>/dev/null || exit 1 ; \
		echo ">> saving prettified CBOR to $${f%.cbor}.pretty" ; \
		cbor2pretty.rb $$f > $${f%.cbor}.pretty ; \
	done
.PHONY: check-examples

MCEAT_CDDL := mc+eat.cddl
MCEAT_DIAG_EXAMPLES := eat-ex1.diag
MCEAT_CBOR_EXAMPLES := $(MCEAT_DIAG_EXAMPLES:.diag=.cbor)

$(MCEAT_CDDL): minimal-eat.cddlc eat-plug.cddl $(CDDL)
	cddlc -2 -sClaims-Set -tcddl -Imeasured-component $< > $@

mceat-check-examples: $(MCEAT_CBOR_EXAMPLES) $(MCEAT_CDDL) 
	@for f in $(MCEAT_CBOR_EXAMPLES); do \
		echo ">> validating $$f against $(MCEAT_CDDL)" ; \
		cddl $(MCEAT_CDDL) validate $$f &>/dev/null || exit 1 ; \
		echo ">> saving prettified CBOR to $${f%.cbor}.pretty" ; \
		cbor2pretty.rb $$f > $${f%.cbor}.pretty ; \
	done
.PHONY: mceat-check-examples
