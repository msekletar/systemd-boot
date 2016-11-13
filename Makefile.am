#  This file is part of systemd-boot
#
#  Copyright (C) 2013-2016 Karel Zak <kzak@redhat.com>
#  Copyright (C) 2016 Michal Sekletar <msekleta@redhat.com>
#
#  systemd-boot is free software; you can redistribute it and/or modify it
#  under the terms of the GNU Lesser General Public License as published by
#  the Free Software Foundation; either version 2.1 of the License, or
#  (at your option) any later version.
#
#  systemd-boot is distributed in the hope that it will be useful, but
#  WITHOUT ANY WARRANTY; without even the implied warranty of
#  MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the GNU
#  Lesser General Public License for more details.
#
#  You should have received a copy of the GNU Lesser General Public License
#  along with systemd-boot; If not, see <http://www.gnu.org/licenses/>.

systemd_bootlibdir = $(prefix)/lib/systemd-boot
kernelinstalldir = $(prefix)/lib/kernel/install.d
bashcompletiondir=@bashcompletiondir@
zshcompletiondir=@zshcompletiondir@

dist_bin_SCRIPTS = \
	src/kernel-install/kernel-install

dist_kernelinstall_SCRIPTS = \
	src/kernel-install/50-depmod.install \
	src/kernel-install/90-loaderentry.install

dist_bashcompletion_data = \
	shell-completion/bash/kernel-install

dist_zshcompletion_data = \
	shell-completion/zsh/_kernel-install

if ENABLE_BASH_COMPLETION
dist_bashcompletion_DATA = $(dist_bashcompletion_data)
endif
if ENABLE_ZSH_COMPLETION
dist_zshcompletion_DATA = $(dist_zshcompletion_data)
endif

EXTRA_DIST = autogen.sh README.md LICENSE
CLEANFILES =

DISTCHECK_CONFIGURE_FLAGS = \
	--with-bashcompletiondir=$$dc_install_base/$(bashcompletiondir) \
	--with-zshcompletiondir=$$dc_install_base/$(zshcompletiondir)

if ENABLE_MANPAGES
%.8: %.xml
	$(AM_V_GEN)$(XSLTPROC) -o $@ --nonet \
	  --stringparam man.output.quietly 1 \
	  --stringparam man.th.extra1.suppress 1 \
	  --stringparam man.authors.section.enabled 0 \
	  --stringparam man.copyright.section.enabled 0 \
	  http://docbook.sourceforge.net/release/xsl/current/manpages/docbook.xsl $<

dist_man_MANS = man/kernel-install.8
endif

EXTRA_DIST += man/kernel-install.xml
CLEANFILES += man/kernel-install.8

# ------------------------------------------------------------------------------
# EFI compilation -- this part of the build system uses custom make rules and
# bypasses regular automake to provide absolute control on compiler and linker
# flags.
EFI_MACHINE_TYPE_NAME = @MACHINE_TYPE_NAME@

efi_cppflags = \
	$(EFI_CPPFLAGS) \
	-I$(top_builddir) -include config.h \
	-I$(EFI_INC_DIR)/efi \
	-I$(EFI_INC_DIR)/efi/$(ARCH) \
	-DEFI_MACHINE_TYPE_NAME=\"$(EFI_MACHINE_TYPE_NAME)\"

efi_cflags = \
	$(EFI_CFLAGS) \
	-Wall \
	-Wextra \
	-std=gnu90 \
	-nostdinc \
	-ggdb -O0 \
	-fpic \
	-fshort-wchar \
	-nostdinc \
	-ffreestanding \
	-fno-strict-aliasing \
	-fno-stack-protector \
	-Wsign-compare \
	-mno-sse \
	-mno-mmx

if ARCH_X86_64
efi_cflags += \
	-mno-red-zone \
	-DEFI_FUNCTION_WRAPPER \
	-DGNU_EFI_USE_MS_ABI
endif

efi_ldflags = \
	$(EFI_LDFLAGS) \
	-T $(EFI_LDS_DIR)/elf_$(ARCH)_efi.lds \
	-shared \
	-Bsymbolic \
	-nostdlib \
	-znocombreloc \
        -L $(EFI_LIB_DIR) \
	$(EFI_LDS_DIR)/crt0-efi-$(ARCH).o

# ------------------------------------------------------------------------------
systemd_boot_headers = \
	src/sd-boot/util.h \
	src/sd-boot/console.h \
	src/sd-boot/graphics.h \
	src/sd-boot/pefile.h \
	src/sd-boot/measure.h \
	src/sd-boot/disk.h

systemd_boot_sources = \
	src/sd-boot/util.c \
	src/sd-boot/console.c \
	src/sd-boot/graphics.c \
	src/sd-boot/pefile.c \
	src/sd-boot/disk.c \
	src/sd-boot/measure.c \
	src/sd-boot/boot.c

systemd_boot_objects = $(addprefix $(top_builddir)/,$(systemd_boot_sources:.c=.o))
systemd_boot_solib = $(top_builddir)/src/sd-boot/systemd_boot.so
systemd_boot = systemd-boot$(EFI_MACHINE_TYPE_NAME).efi

systemd_bootlib_DATA = $(systemd_boot)
CLEANFILES += \
	$(systemd_boot_objects) \
	$(systemd_boot_solib) \
	$(systemd_boot)

EXTRA_DIST += \
	$(systemd_boot_sources) \
	$(systemd_boot_headers)

$(top_builddir)/src/sd-boot/%.o: $(top_srcdir)/src/sd-boot/%.c $(addprefix $(top_srcdir)/,$(systemd_boot_headers))
	@$(MKDIR_P) $(top_builddir)/src/sd-boot
	$(AM_V_CC)$(CC) $(efi_cppflags) $(efi_cflags) -c $< -o $@

$(systemd_boot_solib): $(systemd_boot_objects)
	$(AM_V_CCLD)$(LD) $(efi_ldflags) $(systemd_boot_objects) \
		-o $@ -lefi -lgnuefi $(shell $(CC) -print-libgcc-file-name); \
	nm -D -u $@ | grep ' U ' && exit 1 || :
.DELETE_ON_ERROR: $(sustemd_boot_solib)

$(systemd_boot): $(systemd_boot_solib)
	$(AM_V_GEN) objcopy -j .text -j .sdata -j .data -j .dynamic \
	  -j .dynsym -j .rel -j .rela -j .reloc \
	  --target=efi-app-$(ARCH) $< $@

# ------------------------------------------------------------------------------
stub_headers = \
	src/sd-boot/util.h \
	src/sd-boot/pefile.h \
	src/sd-boot/disk.h \
	src/sd-boot/graphics.h \
	src/sd-boot/splash.h \
	src/sd-boot/measure.h \
	src/sd-boot/linux.h

stub_sources = \
	src/sd-boot/util.c \
	src/sd-boot/pefile.c \
	src/sd-boot/disk.c \
	src/sd-boot/graphics.c \
	src/sd-boot/splash.c \
	src/sd-boot/linux.c \
	src/sd-boot/measure.c \
	src/sd-boot/stub.c

stub_objects = $(addprefix $(top_builddir)/,$(stub_sources:.c=.o))
stub_solib = $(top_builddir)/src/sd-boot/stub.so
stub = linux$(EFI_MACHINE_TYPE_NAME).efi.stub

systemd_bootlib_DATA += $(stub)
CLEANFILES += \
	$(stub_objects) \
	$(stub_solib) \
	$(stub)

EXTRA_DIST += \
	$(stub_sources) \
	$(stub_headers) \
	test/splash.bmp

$(top_builddir)/src/sd-boot/%.o: $(top_srcdir)/src/sd-boot/%.c $(addprefix $(top_srcdir)/,$(stub_headers))
	@$(MKDIR_P) $(top_builddir)/src/sd-boot
	$(AM_V_CC)$(CC) $(efi_cppflags) $(efi_cflags) -c $< -o $@

$(stub_solib): $(stub_objects)
	$(AM_V_CCLD)$(LD) $(efi_ldflags) $(stub_objects) \
		-o $@ -lefi -lgnuefi $(shell $(CC) -print-libgcc-file-name); \
	nm -D -u $@ | grep ' U ' && exit 1 || :
.DELETE_ON_ERROR: $(systemd_boot_solib)

$(stub): $(stub_solib)
	$(AM_V_GEN) objcopy -j .text -j .sdata -j .data -j .dynamic \
	  -j .dynsym -j .rel -j .rela -j .reloc \
	  --target=efi-app-$(ARCH) $< $@

# ------------------------------------------------------------------------------
CLEANFILES += test-disk.img
EXTRA_DIST += test/test-efi-create-disk.sh

test-efi-disk.img: systemd_boot$(EFI_MACHINE_TYPE_NAME).efi test/test-efi-create-disk.sh
	$(AM_V_GEN)test/test-efi-create-disk.sh

qemu: test-efi-disk.img
	$(QEMU) -machine accel=kvm -m 1024 -bios $(QEMU_BIOS) -snapshot test-efi-disk.img

install-tree: all
	rm -rf $(abs_srcdir)/install-tree
	$(MAKE) install DESTDIR=$(abs_srcdir)/install-tree
	tree $(abs_srcdir)/install-tree
