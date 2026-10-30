# Build the preliminary notes PDF.
#
# Requires: pdflatex, bibtex, and the packages geometry, booktabs, longtable,
# natbib, hyperref, microtype, fancyhdr, xcolor, url, lmodern.
# On Debian/Ubuntu:
#   apt-get install -y --no-install-recommends \
#       texlive-latex-base texlive-latex-recommended \
#       texlive-bibtex-extra texlive-fonts-recommended lmodern

DOC = fl-biocountermeasures-preliminary

.PHONY: all clean distclean check

all: $(DOC).pdf

# Four passes: the first resolves \cite keys into the .aux, bibtex builds the
# .bbl, and the last two settle references, the table of contents and page
# numbering.
$(DOC).pdf: $(DOC).tex references.bib
	pdflatex -interaction=nonstopmode -halt-on-error $(DOC).tex
	bibtex $(DOC)
	pdflatex -interaction=nonstopmode -halt-on-error $(DOC).tex
	pdflatex -interaction=nonstopmode -halt-on-error $(DOC).tex

# Fail loudly on the problems that silently degrade a citation-heavy document.
# Rebuilds unconditionally, since the checks read the log from a fresh run.
check: distclean $(DOC).pdf
	@echo "--- undefined citations/references ---"
	@if grep -iE 'undefined (citation|reference)' $(DOC).log; then \
	    echo "FAIL: unresolved citations or references"; exit 1; \
	else echo "none"; fi
	@echo "--- overfull boxes ---"
	@grep -c 'Overfull' $(DOC).log || echo 0

clean:
	rm -f $(DOC).aux $(DOC).bbl $(DOC).blg $(DOC).log $(DOC).out $(DOC).toc \
	      *.log

distclean: clean
	rm -f $(DOC).pdf
