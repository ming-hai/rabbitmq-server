# Platform detection.

ifeq ($(PLATFORM),)
UNAME_S := $(shell uname -s)

ifeq ($(UNAME_S),Linux)
PLATFORM = linux
else ifeq ($(UNAME_S),Darwin)
PLATFORM = darwin
else ifeq ($(UNAME_S),SunOS)
PLATFORM = solaris
else ifeq ($(UNAME_S),GNU)
PLATFORM = gnu
else ifeq ($(UNAME_S),FreeBSD)
PLATFORM = freebsd
else ifeq ($(UNAME_S),NetBSD)
PLATFORM = netbsd
else ifeq ($(UNAME_S),OpenBSD)
PLATFORM = openbsd
else ifeq ($(UNAME_S),DragonFly)
PLATFORM = dragonfly
else ifeq ($(shell uname -o),Msys)
PLATFORM = msys2
else
$(error Unable to detect platform. Please open a ticket with the output of uname -a.)
endif
endif

all: packages
	@:

# --------------------------------------------------------------------
# Packaging.
# --------------------------------------------------------------------

.PHONY: packages package-deb \
	package-rpm package-rpm-fedora package-rpm-suse \
	package-windows package-standalone-macosx \
	package-generic-unix

PACKAGES_DIR ?= ../PACKAGES
SOURCE_DIST_FILE ?= $(wildcard $(PACKAGES_DIR)/rabbitmq-server-*.tar.xz)

ifneq ($(filter-out clean,$(MAKECMDGOALS)),)
ifeq ($(SOURCE_DIST_FILE),)
$(error Cannot find source archive; please specify SOURCE_DIST_FILE)
endif
ifneq ($(words $(SOURCE_DIST_FILE)),1)
$(error Multile source archives found; please specify SOURCE_DIST_FILE)
endif
ifeq ($(filter %.tar.xz %.txz,$(SOURCE_DIST_FILE)),)
$(error The source archive must a tar.xz archive)
endif
ifeq ($(wildcard $(SOURCE_DIST_FILE)),)
$(error The source archive must exist)
endif
endif

ifndef NO_CLEAN
DO_CLEAN := clean
endif

VARS = SOURCE_DIST_FILE="$(abspath $(SOURCE_DIST_FILE))" \
       PACKAGES_DIR="$(abspath $(PACKAGES_DIR))" \
       SIGNING_KEY="$(SIGNING_KEY)"

packages: package-deb package-rpm package-windows package-generic-unix
	@:

package-deb: $(SOURCE_DIST_FILE)
	$(gen_verbose) $(MAKE) -C debs/Debian $(VARS) all $(DO_CLEAN)

package-rpm: package-rpm-fedora package-rpm-suse
	@:

package-rpm-fedora: $(SOURCE_DIST_FILE)
	$(gen_verbose) $(MAKE) -C RPMS/Fedora $(VARS) all $(DO_CLEAN)

package-rpm-suse: $(SOURCE_DIST_FILE)
	$(gen_verbose) $(MAKE) -C RPMS/Fedora $(VARS) RPM_OS=suse all $(DO_CLEAN)

package-windows: $(SOURCE_DIST_FILE)
	$(gen_verbose) $(MAKE) -C windows $(VARS) all $(DO_CLEAN)
	$(verbose) $(MAKE) -C windows-exe $(VARS) all $(DO_CLEAN)

package-generic-unix: $(SOURCE_DIST_FILE)
	$(gen_verbose) $(MAKE) -C generic-unix $(VARS) all $(DO_CLEAN)

ifeq ($(PLATFORM),darwin)
packages: package-standalone-macosx

package-standalone-macosx: $(SOURCE_DIST_FILE)
	$(gen_verbose) $(MAKE) -C standalone $(VARS) OS=mac all $(DO_CLEAN)
endif

.PHONY: clean

clean:
	for subdir in debs/Debian RPMS/Fedora windows windows-exe generic-unix standalone; do \
		$(MAKE) -C "$$subdir" clean; \
	done
