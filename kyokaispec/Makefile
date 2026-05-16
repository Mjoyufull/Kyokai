SOURCES := spec.md \
           src/language/00-introduction.md \
           src/language/01-goals-and-non-goals.md \
           src/language/02-lexical-syntax.md \
           src/language/03-grammar.md \
           src/language/04-modules-and-visibility.md \
           src/language/05-declarations.md \
           src/language/06-type-system.md \
           src/language/07-generics-and-typeclasses.md \
           src/language/08-patterns.md \
           src/language/09-expressions-and-evaluation.md \
           src/language/10-statements-and-control-flow.md \
           src/language/11-linearity-borrowing-and-regions.md \
           src/language/12-implicit-completions-and-elaboration.md \
           src/language/13-contracts-and-runtime-failure.md \
           src/language/14-capabilities-and-authority.md \
           src/language/15-concurrency.md \
           src/language/16-unsafe-ffi-and-abi.md \
           src/language/17-memory-layout-and-backend-contract.md \
           src/language/18-built-ins.md \
           src/language/19-examples.md \
           src/toolchain/00-toolchain-overview.md \
           src/toolchain/01-manifest-package-workspace.md \
           src/toolchain/02-module-resolution-and-koi.md \
           src/toolchain/03-cli.md \
           src/toolchain/04-build-profiles-targets-linking.md \
           src/toolchain/05-diagnostics.md \
           src/toolchain/06-formatter.md \
           src/toolchain/07-testing-coverage-bench.md \
           src/toolchain/08-docs-lsp-audit.md \
           src/toolchain/09-reproducibility-incremental-builds.md \
           src/toolchain/10-package-index-semver-releases-ci.md \
           src/toolchain/11-build-generation-and-playground.md \
           src/stdlib/00-stdlib-overview.md \
           src/stdlib/01-admission-contracts.md \
           src/stdlib/02-core-result-optional-display-error.md \
           src/stdlib/03-allocators-and-memory-containers.md \
           src/stdlib/04-text-bytes-paths-and-strings.md \
           src/stdlib/05-collections.md \
           src/stdlib/06-iterators-and-generators.md \
           src/stdlib/07-math-and-numerics.md \
           src/stdlib/08-io-files-env-process-time-random.md \
           src/stdlib/09-concurrency-primitives.md \
           src/stdlib/10-crypto-policy.md \
           src/stdlib/11-transitional-ffi-tracking.md \
           src/rationale/00-rationale-index.md \
           src/rationale/01-language-design.md \
           src/rationale/02-syntax.md \
           src/rationale/03-error-handling-and-tpoe.md \
           src/rationale/04-linear-resources.md \
           src/rationale/05-capabilities.md \
           src/rationale/06-concurrency.md \
           src/rationale/07-stdlib-philosophy.md \
           src/rationale/08-toolchain-philosophy.md \
           src/appendices/a-license.md \
           src/appendices/b-decision-traceability.md \
           src/appendices/c-austral-differences.md \
           src/appendices/d-formalization-roadmap.md \
           src/appendix-a.md

PANDOC_FLAGS := --standalone --embed-resources --section-divs --table-of-contents --toc-depth=3

OUTPUT_PDF := spec.pdf
OUTPUT_HTML := spec.html
PDF_ENGINE ?= pdflatex

.PHONY: default all pdf html check-sources check-pdf-engine clean

default: html

all: html pdf

pdf: check-pdf-engine $(OUTPUT_PDF)

html: $(OUTPUT_HTML)

check-sources:
	@missing=0; \
	for source in $(SOURCES); do \
		if [ ! -f "$$source" ]; then \
			echo "missing source: $$source"; \
			missing=1; \
		fi; \
	done; \
	exit $$missing

$(OUTPUT_HTML): $(SOURCES)
	pandoc $(SOURCES) $(PANDOC_FLAGS) -t html -o $(OUTPUT_HTML)

check-pdf-engine:
	@command -v $(PDF_ENGINE) >/dev/null 2>&1 || { \
		echo "PDF engine '$(PDF_ENGINE)' not found."; \
		echo "Run 'make html' for the default HTML build, install $(PDF_ENGINE), or pass PDF_ENGINE=<engine> to make pdf."; \
		exit 1; \
	}

$(OUTPUT_PDF): $(SOURCES)
	pandoc $(SOURCES) $(PANDOC_FLAGS) -t latex --pdf-engine=$(PDF_ENGINE) -o $(OUTPUT_PDF)

clean:
	rm -f $(OUTPUT_PDF) $(OUTPUT_HTML)
