.PHONY: all binary build build-binary build-gccgo cross default docs gccgo test test-integration-cli test-unit validate help win tgz

PACKAGE := github.com/containers/storage
GIT_BRANCH := $(shell git rev-parse --abbrev-ref HEAD 2>/dev/null)
GIT_BRANCH_CLEAN := $(shell echo $(GIT_BRANCH) | sed -e "s/[^[:alnum:]]/-/g")
EPOCH_TEST_COMMIT := 0418ebf59f9e1f564831c0ba9378b7f8e40a1c73
NATIVETAGS := exclude_graphdriver_devicemapper exclude_graphdriver_btrfs
AUTOTAGS := $(shell ./hack/btrfs_tag.sh) $(shell ./hack/libdm_tag.sh)
BUILDFLAGS := -tags "$(AUTOTAGS) $(TAGS)" $(FLAGS)
GO := go

RUNINVM := vagrant/runinvm.sh

default all: local-binary docs local-validate local-cross local-gccgo test-unit test-integration ## validate all checks, build and cross-build\nbinaries and docs, run tests in a VM

clean: ## remove all built files
	$(RM) -f containers-storage containers-storage.* docs/*.1

binary local-binary: ## build using gc on the host
	$(GO) build -compiler gc $(BUILDFLAGS) ./cmd/containers-storage

local-gccgo: ## build using gccgo on the host
	GCCGO=$(PWD)/hack/gccgo-wrapper.sh $(GO) build -compiler gccgo $(BUILDFLAGS) -o containers-storage.gccgo ./cmd/containers-storage

local-cross: ## cross build the binaries for arm, darwin, and\nfreebsd
	@for target in linux/amd64 linux/386 linux/arm darwin/amd64 ; do \
		os=`echo $${target} | cut -f1 -d/` ; \
		arch=`echo $${target} | cut -f2 -d/` ; \
		suffix=$${os}.$${arch} ; \
		$(MAKE) GOOS=$${os} GOARCH=$${arch} FLAGS="-o containers-storage.$${suffix}" AUTOTAGS="$(NATIVETAGS)" local-binary; \
	done

cross: ## cross build the binaries for arm, darwin, and\nfreebsd using VMs
	$(RUNINVM) make local-$@

docs: ## build the docs on the host
	$(MAKE) -C docs docs

gccgo: ## build using gccgo using VMs
	$(RUNINVM) make local-$@

test: local-binary ## build the binaries and run the tests using VMs
	$(RUNINVM) make local-binary local-cross local-test-unit local-test-integration

local-test-unit: local-binary ## run the unit tests on the host (requires\nsuperuser privileges)
	@$(GO) test $(BUILDFLAGS) $(shell $(GO) list ./... | grep -v ^$(PACKAGE)/vendor)

test-unit: local-binary ## run the unit tests using VMs
	$(RUNINVM) make local-$@

local-test-integration: local-binary ## run the integration tests on the host (requires\nsuperuser privileges)
	@cd tests; ./test_runner.sh

test-integration: local-binary ## run the integration tests using VMs
	$(RUNINVM) make local-$@

local-validate: ## validate DCO and gofmt on the host
	@./hack/git-validation.sh
	@./hack/gofmt.sh

validate: ## validate DCO, gofmt, ./pkg/ isolation, golint,\ngo vet and vendor using VMs
	$(RUNINVM) make local-$@

.PHONY: install.tools
install.tools:
	go get -u $(BUILDFLAGS) github.com/cpuguy83/go-md2man
	go get -u $(BUILDFLAGS) github.com/vbatts/git-validation
	go get -u $(BUILDFLAGS) gopkg.in/alecthomas/gometalinter.v1
	gometalinter.v1 -i

help: ## this help
	@awk 'BEGIN {FS = ":.*?## "} /^[a-z A-Z_-]+:.*?## / {gsub(" ",",",$$1);gsub("\\\\n",sprintf("\n%22c"," "), $$2);printf "\033[36m%-21s\033[0m %s\n", $$1, $$2}' $(MAKEFILE_LIST)
