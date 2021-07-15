###############################################################################
## Makefile for apertium-cym
###############################################################################

LANG1=cym
BASENAME=apertium-$(LANG1)

TARGETS_COMMON = $(LANG1).automorf.bin \
	$(LANG1).autogen.bin pre-$(LANG1).t1x.bin $(LANG1).autopgen.bin $(LANG1).autoorth.bin \
	$(LANG1).automorf.att.gz $(LANG1).autogen.att.gz \
	$(LANG1).rlx.bin

# This include defines goals for install-modes, .deps/.d and .mode files:
@ap_include@

###############################################################################
## Welsh transducer
###############################################################################

$(LANG1).autogen.bin: $(BASENAME).$(LANG1).dix
	lt-comp rl $< $@

$(LANG1).automorf.bin: $(BASENAME).$(LANG1).dix
	lt-comp lr $< $@ $(BASENAME).$(LANG1).acx

$(LANG1).autogen.att.gz: $(LANG1).autogen.bin
	lt-print $< | gzip -9 -c -n > $@

$(LANG1).automorf.att.gz: $(LANG1).automorf.bin
	lt-print $< | gzip -9 -c -n > $@

$(LANG1).autopgen.bin: $(BASENAME).post-$(LANG1).dix
	lt-comp lr $< $@

$(LANG1).autoorth.bin: $(BASENAME).orth-$(LANG1).dix
	lt-comp lr $< $@

###############################################################################
## Disambiguation rules 
###############################################################################

$(LANG1).rlx.bin: $(BASENAME).$(LANG1).rlx
	cg-comp $< $@

###############################################################################
## Transfer rules (used before generation)
###############################################################################

pre-$(LANG1).t1x.bin: $(BASENAME).pre-$(LANG1).t1x
	apertium-validate-transfer $(BASENAME).pre-$(LANG1).t1x
	apertium-preprocess-transfer $(BASENAME).pre-$(LANG1).t1x $@

###############################################################################
## Spell checker
###############################################################################

$(LANG1).zhfst: .deps/acceptor.default.hfst .deps/errmodel.default.hfst
	rm -f $@
	zip -Z store -j $@ .deps/acceptor.default.hfst .deps/errmodel.default.hfst speller/index.xml

.deps/editdist.default.hfst: speller/editdist.default.txt .deps/acceptor.default.hfst
	python dev/editdist.py -v -s -d 1 -e '@0@' -i $< -a .deps/acceptor.default.hfst >$@.txt
	hfst-txt2fst -i $@.txt -e '@0@' -o $@
	rm -f $@.txt

.deps/editstrings.default.hfst: .deps/strings.default.hfst .deps/editdist.default.hfst
	hfst-disjunct $^ | hfst-minimise | hfst-repeat -f 1 -t 2 -o $@

.deps/errmodel.default.hfst: .deps/words.default.hfst .deps/editstrings.default.hfst
	hfst-disjunct $^ | hfst-fst2fst -f olw -o $@

.deps/words.default.hfst: speller/words.default.txt
	grep -v -e "^#" -e "^$$" $< | hfst-strings2fst -j -o $@

.deps/strings.default.hfst: speller/strings.default.txt .deps/anystar.hfst
	grep -v -e "^#" -e "^$$" $< | hfst-strings2fst -j | hfst-concatenate .deps/anystar.hfst - | hfst-concatenate - .deps/anystar.hfst -o $@

.deps/anystar.hfst:
	echo "?*;" | hfst-regexp2fst -S -o $@

.deps/acceptor.default.hfst: .deps/$(LANG1).autogen.bin 
	lt-print .deps/$(LANG1).autogen.bin | sed 's/ε/@0@/g' | sed 's/ /@_SPACE_@/g' | hfst-txt2fst | hfst-fst2fst -t | hfst-project --project=lower | hfst-minimise |hfst-fst2fst -f olw -o $@ 

.deps/$(LANG1).autogen.bin: .deps/$(BASENAME).$(LANG1).dix
	lt-comp rl $< $@

.deps/$(BASENAME).$(LANG1).dix: $(BASENAME).$(LANG1).dix .deps/.d
	echo '<dictionary>' > $@
	cat $< | xmllint  --xpath './/alphabet|.//sdefs|.//pardefs|.//section[@id="main"]' - >> $@
	echo '</dictionary>' >> $@
	cat .deps/$(BASENAME).$(LANG1).dix | xmllint --encode utf8 - > .deps/$(BASENAME).$(LANG1).dixtmp
	mv .deps/$(BASENAME).$(LANG1).dixtmp $@


###############################################################################
## Distribution
###############################################################################
EXTRA_DIST=$(BASENAME).$(LANG1).dix \
		$(BASENAME).$(LANG1).acx \
		$(BASENAME).$(LANG1).rlx \
		$(BASENAME).post-$(LANG1).dix \
		$(BASENAME).pre-$(LANG1).t1x \
		$(LANG1).prob \
		modes.xml

###############################################################################
## Installation stuff
###############################################################################
#
#   apertium_cym_dir: This is where the compiled binaries go
#   apertium_cym_srcdir: This is where the source files go

apertium_cymdir=$(prefix)/share/apertium/$(BASENAME)/

EXTRA_TARGETS=

if HAVE_HFSTOSPELL
EXTRA_TARGETS += $(LANG1).zhfst
endif # HAVE_HFSTOSPELL

apertium_cym_DATA=$(TARGETS_COMMON) $(LANG1).prob $(BASENAME).pre-$(LANG1).t1x $(EXTRA_TARGETS) \
	$(BASENAME).$(LANG1).dix \
	$(BASENAME).$(LANG1).rlx


pkgconfigdir = $(prefix)/share/pkgconfig
pkgconfig_DATA = $(BASENAME).pc

noinst_DATA=modes/$(LANG1)-morph.mode

install-data-local: install-modes

CLEANFILES = $(TARGETS_COMMON)
clean-local:
	-rm -rf .deps modes

###############################################################################
## Test
###############################################################################

test: all
	apertium-regtest test
