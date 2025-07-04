# SPDX-License-Identifier: GPL-2.0
include ../../scripts/Makefile.include
include ../../scripts/utilities.mak		# QUIET_CLEAN

ifeq ($(srctree),)
srctree := $(patsubst %/,%,$(dir $(CURDIR)))
srctree := $(patsubst %/,%,$(dir $(srctree)))
srctree := $(patsubst %/,%,$(dir $(srctree)))
#$(info Determined 'srctree' to be $(srctree))
endif

CC ?= $(CROSS_COMPILE)gcc
AR ?= $(CROSS_COMPILE)ar
LD ?= $(CROSS_COMPILE)ld

MAKEFLAGS += --no-print-directory

INSTALL = install


# Use DESTDIR for installing into a different root directory.
# This is useful for building a package. The program will be
# installed in this directory as if it was the root directory.
# Then the build tool can move it later.
DESTDIR ?=
DESTDIR_SQ = '$(subst ','\'',$(DESTDIR))'

LIBFILE = $(OUTPUT)libapi.a

CFLAGS := $(EXTRA_WARNINGS) $(EXTRA_CFLAGS)
CFLAGS += -ggdb3 -Wall -Wextra -std=gnu99 -U_FORTIFY_SOURCE -fPIC

ifeq ($(DEBUG),0)
  CFLAGS += -O3
endif

ifeq ($(DEBUG),0)
  CFLAGS += -D_FORTIFY_SOURCE=2
endif

# Treat warnings as errors unless directed not to
ifneq ($(WERROR),0)
  CFLAGS += -Werror
endif

CFLAGS += -D_LARGEFILE64_SOURCE -D_FILE_OFFSET_BITS=64
CFLAGS += -I$(srctree)/tools/lib/api
CFLAGS += -I$(srctree)/tools/include

RM = rm -f

API_IN := $(OUTPUT)libapi-in.o

ifeq ($(LP64), 1)
  libdir_relative = lib64
else
  libdir_relative = lib
endif

prefix ?=
libdir = $(prefix)/$(libdir_relative)

# Shell quotes
libdir_SQ = $(subst ','\'',$(libdir))

all:

export srctree OUTPUT CC LD CFLAGS V
include $(srctree)/tools/build/Makefile.include
include $(srctree)/tools/scripts/Makefile.include

all: fixdep $(LIBFILE)

$(API_IN): FORCE
	@$(MAKE) $(build)=libapi

$(LIBFILE): $(API_IN)
	$(QUIET_AR)$(RM) $@ && $(AR) rcs $@ $(API_IN)

define do_install_mkdir
	if [ ! -d '$(DESTDIR_SQ)$1' ]; then             \
		$(INSTALL) -d -m 755 '$(DESTDIR_SQ)$1'; \
	fi
endef

define do_install
	if [ ! -d '$2' ]; then             \
		$(INSTALL) -d -m 755 '$2'; \
	fi;                                \
	$(INSTALL) $1 $(if $3,-m $3,) '$2'
endef

install_lib: $(LIBFILE)
	$(call QUIET_INSTALL, $(LIBFILE)) \
		$(call do_install_mkdir,$(libdir_SQ)); \
		cp -fpR $(LIBFILE) $(DESTDIR)$(libdir_SQ)

HDRS := cpu.h debug.h io.h
FD_HDRS := fd/array.h
FS_HDRS := fs/fs.h fs/tracing_path.h
INSTALL_HDRS_PFX := $(DESTDIR)$(prefix)/include/api
INSTALL_HDRS := $(addprefix $(INSTALL_HDRS_PFX)/, $(HDRS))
INSTALL_FD_HDRS := $(addprefix $(INSTALL_HDRS_PFX)/, $(FD_HDRS))
INSTALL_FS_HDRS := $(addprefix $(INSTALL_HDRS_PFX)/, $(FS_HDRS))

$(INSTALL_HDRS): $(INSTALL_HDRS_PFX)/%.h: %.h
	$(call QUIET_INSTALL, $@) \
		$(call do_install,$<,$(INSTALL_HDRS_PFX)/,644)

$(INSTALL_FD_HDRS): $(INSTALL_HDRS_PFX)/fd/%.h: fd/%.h
	$(call QUIET_INSTALL, $@) \
		$(call do_install,$<,$(INSTALL_HDRS_PFX)/fd/,644)

$(INSTALL_FS_HDRS): $(INSTALL_HDRS_PFX)/fs/%.h: fs/%.h
	$(call QUIET_INSTALL, $@) \
		$(call do_install,$<,$(INSTALL_HDRS_PFX)/fs/,644)

install_headers: $(INSTALL_HDRS) $(INSTALL_FD_HDRS) $(INSTALL_FS_HDRS)
	$(call QUIET_INSTALL, libapi_headers)

install: install_lib install_headers

clean:
	$(call QUIET_CLEAN, libapi) $(RM) $(LIBFILE); \
	find $(or $(OUTPUT),.) -name \*.o -or -name \*.o.cmd -or -name \*.o.d | xargs $(RM)

FORCE:

.PHONY: clean FORCE
