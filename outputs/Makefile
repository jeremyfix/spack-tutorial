.PHONY: update-outputs run interactive clean

# Tools
DOCKER := docker

# Name of the container we'll generate the tutorial outputs with
container := ghcr.io/spack/tutorial:latest

# Sections to build
sections := basics cache dev environments modules packaging scripting stacks

# Allow users to override variables (e.g. DOCKER := sudo docker)
-include Make.user

run_targets := $(addprefix run-,$(sections))

# Entrypoint
update-outputs: run
	echo "Filtering raw outputs though col"
	for raw in raw/*/*.out; do \
		out=$$(echo $$raw | sed 's.raw/..'); \
		cat $$raw | perl -pe 's/\x1b\[\d+F\x1b\[J//g' | perl -pe 's/\033\[([01];)?\d+m//g' | col -bp | sed '/^==> Waiting for/d' > $$out; \
	done

run: $(run_targets)

$(addprefix local-run-,$(sections)):
	$(CURDIR)/$(@:local-run-%=%).sh

$(run_targets): run-%: %.sh init_spack.sh defs.sh
	$(DOCKER) run --rm -t \
		--mount type=bind,source=$(CURDIR),target=/project \
		${container} \
		/project/$(@:run-%=%).sh && touch $@

interactive:
	$(DOCKER) run --rm -it \
		--mount type=bind,source=$(CURDIR),target=/project \
		${container}

$(addprefix clean-,$(sections)):
	rm -f $(@:clean-%=run-%) $(wildcard raw/$(@:clean-%=%)/*.out)

clean:
	rm -f $(run_targets) $(wildcard raw/*/*.out)
