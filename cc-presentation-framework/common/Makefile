SRC     := $(wildcard index.adoc)
HTML    := $(patsubst %.adoc,%.html,$(SRC))
PDF     := $(patsubst %.adoc,%.pdf,$(SRC))
ALL_SRC := *.adoc **/*.adoc

FORMATS := html pdf

_OUT_FILES := $(foreach FORMAT,$(FORMATS),$(shell echo $(FORMAT) | tr '[:lower:]' '[:upper:]'))
OUT_FILES  := $(foreach F,$(_OUT_FILES),$($F))

SHELL := /bin/bash

all: $(OUT_FILES)

%.html: %.adoc revealjs-css reveal.js images bundle ../cc-presentation-framework/revealjs-css/calconnect.css
	bundle exec asciidoctor-revealjs $< --trace

%.pdf: %.html
	decktape reveal $^ $@

define FORMAT_TASKS
OUT_FILES-$(FORMAT) := $($(shell echo $(FORMAT) | tr '[:lower:]' '[:upper:]'))

open-$(FORMAT):
	open $$(OUT_FILES-$(FORMAT))

clean-$(FORMAT):
	rm -f $$(OUT_FILES-$(FORMAT))

$(FORMAT): clean-$(FORMAT) $$(OUT_FILES-$(FORMAT))

.PHONY: clean-$(FORMAT) open-$(FORMAT)

endef

$(foreach FORMAT,$(FORMATS),$(eval $(FORMAT_TASKS)))

open: $(foreach FORMAT,$(FORMATS),open-$(FORMAT))

clean:
	rm -rf $(OUT_FILES) Gemfile Gemfile.lock package.json reveal.js revealjs-css

bundle: Gemfile Gemfile.lock
	bundle

Gemfile Gemfile.lock package.json:
	ln -s ../common/$@ .

.PHONY: bundle all open clean

#
# Reveal.js-related jobs
#

CSS_SRC := ../cc-presentation-framework/revealjs-css/calconnect.scss

$(CSS_SRC):
	git submodule update --init
	pushd ../cc-presentation-framework ; \
		git submodule update --init

../cc-presentation-framework/revealjs-css/calconnect.css: $(CSS_SRC) ../cc-presentation-framework/reveal.js/node_modules/grunt
	cp $< ../cc-presentation-framework/reveal.js/css/theme/source/
	pushd ../cc-presentation-framework/reveal.js ; \
		grunt css-themes
	cp ../cc-presentation-framework/reveal.js/css/theme/calconnect.css ../cc-presentation-framework/revealjs-css/
	rm -f ../cc-presentation-framework/reveal.js/css/theme/calconnect.scss
	rm -f ../cc-presentation-framework/reveal.js/css/theme/calconnect.css

../cc-presentation-framework/reveal.js/node_modules/grunt:
	pushd ../cc-presentation-framework/reveal.js ; \
		npm install ; \
		popd

reveal-images: ../cc-presentation-framework/reveal-images
	ln -s $< .

revealjs-css: ../cc-presentation-framework/revealjs-css
	ln -s $< .

reveal.js: ../cc-presentation-framework/reveal.js
	ln -s $< .

#
# Watch-related jobs
#

.PHONY: watch watch-html watch-pdf serve watch-serve

NODE_BINS          := onchange live-serve run-p
NODE_BIN_DIR       := node_modules/.bin
NODE_PACKAGE_PATHS := $(foreach PACKAGE_NAME,$(NODE_BINS),$(NODE_BIN_DIR)/$(PACKAGE_NAME))

$(NODE_PACKAGE_PATHS): package.json
	npm i

watch: $(NODE_BIN_DIR)/onchange
	make all
	$< $(ALL_SRC) -- make all

define WATCH_TASKS
watch-$(FORMAT): $(NODE_BIN_DIR)/onchange
	make $(FORMAT)
	$$< $(ALL_SRC) -- make $(FORMAT)

.PHONY: watch-$(FORMAT)
endef

$(foreach FORMAT,$(FORMATS),$(eval $(WATCH_TASKS)))

serve: $(NODE_BIN_DIR)/live-server revealjs-css reveal.js images
	export PORT=$${PORT:-8123} ; \
	port=$${PORT} ; \
	for html in $(HTML); do \
		$< --entry-file=$$html --port=$${port} --ignore="*.html,*.xml,Makefile,Gemfile.*,package.*.json" --wait=1000 & \
		port=$$(( port++ )) ;\
	done

watch-serve: $(NODE_BIN_DIR)/run-p
	$< watch serve
	# make all
