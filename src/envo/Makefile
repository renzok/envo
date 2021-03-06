# This Makefile is for use by the ENVO Release Manager (currently cjm)
# Also used by Jenkins: http://build.berkeleybop.org/job/build-envo
# 
# requirements: Oort (http://code.google.com/p/owltools/wiki/OortIntro)
#
# To make a release, type 'make release' in this directory

OBO=http://purl.obolibrary.org/obo
USECAT= --catalog-xml catalog-v001.xml

all: build

build: envo.owl subsets/envo-basic.obo

release: build release-diffs
#release: envo-basic.obo release-diffs

# all release files are derived from the editors version.
# we use the default Oort setup (hermit as reasoner)
envo-simple.obo: envo.owl
envo.owl: envo-edit.owl
	ontology-release-runner --catalog-xml catalog-v001.xml  --skip-format owx --run-obo-basic-dag-check --error-report envo-errors.txt $< --allow-overwrite --asserted --simple
.PRECIOUS: envo.obo

envo.obo: envo.owl
	owltools $(USECAT) $< --merge-imports-closure --add-obo-shorthand-to-properties -o -f obo $@.tmp && grep -v ^owl-axioms $@.tmp > $@

# TODO - fix IRI
subsets/envo-basic.obo: envo-simple.obo
	owltools envo-simple.obo imports/ro_import.owl --merge-imports-closure  --add-obo-shorthand-to-properties --set-ontology-id $(OBO)/$@ -o -f obo $@.tmp && grep -v ^owl-axioms $@.tmp > $@

#envo-mireot.obo: envo-edit.owl chebi-lite.obo pato-lite.obo
#	ontology-release-runner --no-subsets --skip-format owx --run-obo-basic-dag-check --no-reasoner --allow-overwrite --asserted --outdir build $< chebi-lite.obo pato-lite.obo && cp build/envo-merged.obo $@

#chebi-lite.obo: 
#	owltools $(OBO)/chebi.obo --make-subset-by-properties -o -f obo $@
#pato-lite.obo: 
#	owltools $(OBO)/pato.obo --make-subset-by-properties -o -f obo $@

release-diffs:
	cd diffs && make


# ----------------------------------------
# Regenerate imports
# ----------------------------------------
# Uses OWLAPI Module Extraction code

# Type 'make imports/X_import.owl' whenever you wish to refresh the import for an ontology X. This is when:
#
#  1. X has changed and we want to include these changes
#  2. We have added onr or more new IRI from X into envo-edit.owl
#  3. We have removed references to one or more IRIs in X from envo-edit.owl
#
# You should NOT edit these files directly, changes will be overwritten.
#
# If you want to add something to these, edit envo-edit.owl and add an axiom with a IRI from X. You don't need to add any information about X.

# Base URI for local subset imports
ENVO_IMPORTS_BASE_URI = $(OBO)/envo

# Ontology dependencies
# We don't include clo, as this is currently not working
IMPORTS = pato uberon chebi ro

# Make this target to regenerate ALL
all_imports: $(patsubst %, imports/%_import.owl,$(IMPORTS)) $(patsubst %, imports/%_import.obo,$(IMPORTS))

KEEPRELS = BFO:0000050 BFO:0000051 RO:0002202 immediate_transformation_of

# Create an import module using the OWLAPI module extraction code via OWLTools.
# We use the standard catalog, but rewrite the import to X to be a local mirror of ALL of X.
# After extraction, we further reduce the ontology by creating a "mingraph" (removes all annotations except label) and by 
imports/%_import.owl: envo-edit.owl mirror/%.owl imports/%_seed.owl
	owltools  $(USECAT) --map-ontology-iri $(ENVO_IMPORTS_BASE_URI)/imports/$*_import.owl mirror/$*.owl $< imports/$*_seed.owl --merge-support-ontologies  --extract-module -s $(OBO)/$*.owl -c --remove-axiom-annotations --make-subset-by-properties $(KEEPRELS) --set-ontology-id $(ENVO_IMPORTS_BASE_URI)/$@ -o $@

imports/%_import.obo: imports/%_import.owl
	owltools $(USECAT) $< -o -f obo $@

# clone remote ontology locally, perfoming some excision of relations and annotations
mirror/%.owl: envo-edit.owl
	owltools $(OBO)/$*.owl --remove-annotation-assertions -l --remove-dangling-annotations --make-subset-by-properties -f $(KEEPRELS)  -o $@
.PRECIOUS: mirror/%.owl
mirror/ro.owl: envo-edit.owl
	owltools $(OBO)/ro.owl --merge-imports-closure -o $@
.PRECIOUS: mirror/%.owl
mirror/uberon.owl: envo-edit.owl
	owltools $(OBO)/uberon/basic.owl --remove-annotation-assertions -l -s -d --remove-axiom-annotations --remove-dangling-annotations --make-subset-by-properties -f $(KEEPRELS) --set-ontology-id $(OBO)/uberon.owl -o $@
.PRECIOUS: mirror/%.owl
mirror/po.owl: envo-edit.owl
	owltools $(OBO)/po.owl --remove-annotation-assertions -l -s -d --remove-axiom-annotations --remove-dangling-annotations --make-subset-by-properties -f $(KEEPRELS) --set-ontology-id $(OBO)/po.owl -o $@
.PRECIOUS: mirror/%.owl
ncbitaxon.obo:
	wget -N $(OBO)/ncbitaxon.obo
.PRECIOUS: ncbitaxon.obo
mirror/ncbitaxon.owl: ncbitaxon.obo
	OWLTOOLS_MEMORY=12G owltools $< --remove-annotation-assertions -l -s -d --remove-axiom-annotations --remove-dangling-annotations  --set-ontology-id $(OBO)/ncbitaxon.owl -o $@
.PRECIOUS: mirror/ncbitaxon.owl

mirror/pco.owl: imports/pco_basic.obo
	OWLTOOLS_MEMORY=12G owltools $< --set-ontology-id $(OBO)/pco.owl -o $@



# ----------------------------------------
# SLIMS
# ----------------------------------------
# These all depend on envo-basic, which is the whole ontology (ie all classes), minus non-basic axioms (e.g. complex owl axioms, some relations)
subsets/EnvO-Lite-GSC.owl: subsets/envo-basic.obo
	owltools $< --extract-ontology-subset --subset EnvO-Lite-GSC --iri $(OBO)/envo/subsets/$@ -o $@
subsets/EnvO-Lite-GSC.obo: subsets/EnvO-Lite-GSC.owl
	obolib-owl2obo $< -o $@

# ----------------------------------------
# Reports
# ----------------------------------------
reports/envo-%.csv: envo.owl sparql/%.sparql
	arq --data $< --query sparql/$*.sparql --results csv > $@.tmp && mv $@.tmp $@


# ----------------------------------------
# Temp
# ----------------------------------------
mappings/gold-mapping.txt: envo-simple.obo
	blip-findall -u metadata_nlp_parent_dist2_hook -r obol_av -i sources/gold.obo -i $< -u metadata_nlp -goal index_entity_pair_label_match "entity_pair_label_reciprocal_best_intermatch(X,Y,S)" -use_tabs -label -no_pred > $@.tmp && cut -f1-4 $@.tmp | sort -u > $@
