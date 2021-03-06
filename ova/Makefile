OVA_DIR := $(shell dirname $(MAKEFILE_LIST))

PKGTOOLS_DIR := $(OVA_DIR)/../../ngfw_pkgtools

## overridables
# repository
REPOSITORY ?= stretch
# distribution to draw packages from
DISTRIBUTION ?= current
# architecture
ARCH ?= amd64
# FLAVOR
FLAVOR ?= untangle
# size in GB
SIZE ?= 320

## constants
TS := $(shell date --iso-8601=seconds)
HOST := $(shell hostname -s)
NETBOOT_HOST := netboot-server
IMAGES_DIR := /data/untangle-images-$(REPOSITORY)
MOUNT_SCRIPT := /data/image-manager/mounts.py
VERSION := $(shell cat $(PKGTOOLS_DIR)/resources/VERSION)
VMDK_NAME := $(FLAVOR).vmdk
OVF_FILE := untangle.ovf
OVF_FILE_TPL := $(OVF_FILE).tpl
OVA_NAME := $(FLAVOR).ova
FULL_NAME := $(FLAVOR)-$(VERSION)_$(REPOSITORY)_$(ARCH)_$(DISTRIBUTION)_$(TS)_$(HOST).ova

## targets
.PHONY: all clean vmdk ova push

all: vmdk image

clean:
	rm -fr *-stamp $(VMDK_NAME) *.ova $(OVF_FILE)

vmdk: $(VMDK_NAME)
$(VMDK_NAME):
	$(OVA_DIR)/vmdk.sh $(REPOSITORY) $(DISTRIBUTION) $(ARCH) $@ $(SIZE) $(FLAVOR)

$(OVF_FILE): $(OVF_FILE_TPL)
	perl -pe 's/\+SIZE\+/'$(shell echo "$(SIZE)*1024^3" | bc)'/' $^ > $@

image: $(OVA_NAME)
$(OVA_NAME): $(OVF_FILE) $(VMDK_NAME)
	tar cvf $@ $^
	chmod 644 $@

push:
	scp $(shell ls --sort=time *ova | head -1) $(NETBOOT_HOST):$(IMAGES_DIR)/$(VERSION)/$(FULL_NAME)
	ssh $(NETBOOT_HOST) "sudo python $(MOUNT_SCRIPT) all $(VERSION) foo $(ARCH) $(REPOSITORY)"

