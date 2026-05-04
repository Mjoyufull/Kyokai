SOURCES := spec.md \
           src/0.introduction.md \
           src/1.goals.md \
           src/rationale/0.rationale.md \
           src/rationale/1.syntax.md \
           src/rationale/3.resource-types.md \
           src/rationale/2.error-handling.md \
           src/rationale/4.capabilities.md \
           src/2.syntax.md \
	   src/3.modules.md \
	   src/4.types.md \
	   src/4a.type-classes.md \
	   src/5.declarations.md \
	   src/6.statements.md \
	   src/7.expressions.md \
           src/7a.linearity.md \
           src/7b.builtins.md \
           src/7c.austral-memory.md \
           src/7d.austral-pervasive.md \
           src/7e.ffi.md \
           src/7f.c-ffi.md \
	   src/8.examples.md \
	   src/9.style.md \
	   src/appendix-a.md

PANDOC_FLAGS := --standalone --embed-resources --section-divs --table-of-contents --toc-depth=3

OUTPUT_PDF := spec.pdf
OUTPUT_HTML := spec.html
AUSTRAL ?= austral

.PHONY: default pdf html check-austral-fences clean

default: pdf html

pdf: $(OUTPUT_PDF)

html: $(OUTPUT_HTML)

check-austral-fences:
	python3 scripts/check_austral_fences.py --austral $(AUSTRAL)

$(OUTPUT_HTML): $(SOURCES)
	pandoc $(SOURCES) $(PANDOC_FLAGS) -t html -o $(OUTPUT_HTML)

$(OUTPUT_PDF): $(SOURCES)
	pandoc $(SOURCES) $(PANDOC_FLAGS) -t latex -o $(OUTPUT_PDF)

clean:
	rm spec.pdf; rm spec.html
