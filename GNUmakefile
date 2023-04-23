TEST?=./...
PKGNAME?=./internal/provider
VERSION?=$(shell grep -oP '(?<=Version = ).+' version/version.go | xargs)
RELEASE?=v$(VERSION)

default: build

.PHONY: build
build: fmtcheck
	go install

.PHONY: test
test: fmtcheck
	go test $(TEST) -timeout=30s -parallel=4

.PHONY: testacc
testacc: fmtcheck
	TF_ACC=1 go test $(TEST) -v -count 1 -parallel 20 $(TESTARGS) -timeout 120m

.PHONY: testcompile
testcompile:
	go test -c $(TEST) $(TESTARGS)

.PHONY: vet
vet:
	go vet ./...

.PHONY: fmt
fmt:
	@gofmt -s -w $(CURDIR)

.PHONY: fmtcheck
fmtcheck:
	@sh -c "'$(CURDIR)/scripts/gofmtcheck.sh'"

.PHONY: depscheck
depscheck:
	@echo "==> Checking source code with go mod tidy..."
	@go mod tidy
	@git diff --exit-code -- go.mod go.sum || \
		(echo; echo "Unexpected difference in go.mod/go.sum files. Run 'go mod tidy' command or revert any go.mod/go.sum changes and commit."; exit 1)

.PHONY: docs
docs: tools
	go run github.com/hashicorp/terraform-plugin-docs/cmd/tfplugindocs

.PHONY: docscheck
docscheck: docs
	go run github.com/hashicorp/terraform-plugin-docs/cmd/tfplugindocs validate

.PHONY: tools
tools:
	@go generate -tags tools tools.go

.PHONY: release
release: CI_JOB_NUMBER=$(shell read -p "» Last successful CI job number: " n && [[ -z "$$n" ]] && n="unknown"; echo $$n)
release:
	@git commit -a -m "chore(release): $(RELEASE)" -m "CI_JOB_NUMBER: $(CI_JOB_NUMBER)"
	@git tag -f -m    "chore(release): $(RELEASE)" $(RELEASE)
	@git push --follow-tags

# uncomment for local development

#OS ?= darwin
#ARCH ?= arm64
#OS_ARCH := $(OS)_$(ARCH)
#
#
## Builds the go binary
#.PHONY: binary
#binary:
#	go fmt ./...
#	echo "Building Go binary"
#	go build -o terraform-provider-cm_v1.2.0
#
## Sets up your local workstation to "accept" this local provider binary
#.PHONY: init
#init: binary
#	echo "Initializing..."
#	echo "Setting up for local provider..."
#	rm -f ~/.terraform.d/plugins/example.com/control-monkey/cm/1.2.0/$(OS_ARCH)/terraform-provider-cm_v1.2.0
#	mkdir -p ~/.terraform.d/plugins/example.com/control-monkey/cm/1.2.0/$(OS_ARCH)
#	ln -s $(CURDIR)/terraform-provider-cm_v1.2.0 ~/.terraform.d/plugins/example.com/control-monkey/cm/1.2.0/$(OS_ARCH)/terraform-provider-cm_v1.2.0
#
## Builds the go binary, and cleans up Terraform lock file just in case
#.PHONY: build_local
#build: binary
#	if [ -f "sandbox/.terraform.lock.hcl" ]; then \
#	  rm sandbox/.terraform.lock.hcl; \
#	fi
