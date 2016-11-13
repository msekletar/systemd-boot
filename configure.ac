#  This file is part of systemd-boot
#
#  Copyright (C) 2013-2016 Karel Zak <kzak@redhat.com>
#  Copyright (C) 2016 Michal Sekeltar <msekleta@redhat.com>
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

AC_PREREQ([2.64])
AC_INIT([systemd-boot],
        [232],
        [http://github.com/systemd/systemd-boot/issues],
        [systemd-boot],
        [http://github.com/systemd/systemd-boot])

AC_CONFIG_AUX_DIR([build-aux])
AC_CONFIG_MACRO_DIR([m4])
AC_CONFIG_SRCDIR([src/sd-boot/boot.c])
AC_CONFIG_HEADERS([config.h])

AM_INIT_AUTOMAKE([foreign 1.11 -Wall -Wno-portability silent-rules tar-pax no-dist-gzip dist-xz subdir-objects])
AC_CANONICAL_HOST

# Figure out target architecture and EFI machine type
ARCH="$host_cpu"

AS_IF([test x"$host_cpu" = x"x86_64"], [MACHINE_TYPE_NAME=x64])
AS_CASE([$host_cpu], [i*86], [MACHINE_TYPE_NAME=ia32])

AM_CONDITIONAL(ARCH_X86_64, [test x"$MACHINE_TYPE_NAME" = "x64"])
AM_CONDITIONAL(ARCH_IA32, [test x"$MACHINE_TYPE_NAME" = "ia32"])

AC_SUBST([ARCH])
AC_SUBST([MACHINE_TYPE_NAME])
AC_DEFINE_UNQUOTED([MACHINE_TYPE_NAME], [$MACHINE_TYPE_NAME], [EFI machine type])

AC_SYS_LARGEFILE
AC_SYS_LONG_FILE_NAMES

AC_PROG_CC
AC_PROG_MKDIR_P
AC_PATH_PROG([QEMU], [qemu-system-$ARCH])

AC_PATH_PROG([XSLTPROC], [xsltproc])

PKG_PROG_PKG_CONFIG

# ------------------------------------------------------------------------------
AC_ARG_WITH(ovmf,
        AS_HELP_STRING([--with-ovmf=PATH], [Path to OVMF firmware binary]),
        [OVMF_PATH="$withval"],
        [AC_MSG_WARN([*** No OVMF path specified, using defaults])])

# Search for OVMF firmware in default locations in case
AS_IF([ test -z "$OVMF_PATH" ], [
        AC_PATH_PROG([QEMU], [qemu-system-x86_64])
        # Fedora
        AC_CHECK_FILE([/usr/share/edk2/ovmf/OVMF_CODE.fd], [QEMU_BIOS=/usr/share/edk2/ovmf/OVMF_CODE.fd])
        # Debian
        AC_CHECK_FILE([/usr/share/qemu/OVMF.fd], [QEMU_BIOS=/usr/share/qemu/OVMF.fd])
        # Arch Linux
        AC_CHECK_FILE([/usr/share/ovmf/${MACHINE_TYPE_NAME}/ovmf_${MACHINE_TYPE_NAME}.bin], [QEMU_BIOS=/usr/share/ovmf/${MACHINE_TYPE_NAME}/ovmf_${MACHINE_TYPE_NAME}.bin])
        # SUSE
        AC_CHECK_FILE([/usr/share/qemu/ovmf-${ARCH}.bin], [QEMU_BIOS=/usr/share/qemu/ovmf-${ARCH}.bin])
        AC_SUBST([QEMU_BIOS])
])

# ------------------------------------------------------------------------------
CPPFLAGS="$CPPFLAGS -I/usr/include/efi/$ARCH"
AC_CHECK_HEADER([efi/efi.h], [], [AC_MSG_ERROR([*** gnu-efi headers not found])])

efiroot=$(echo $(cd /usr/lib/$(gcc -print-multi-os-directory); pwd))
EFI_LIB_DIR="$efiroot"
AC_ARG_WITH(efi-libdir,
        AS_HELP_STRING([--with-efi-libdir=PATH], [Path to efi lib directory]),
        [EFI_LIB_DIR="$withval"], [EFI_LIB_DIR="$efiroot"]
)
AC_SUBST([EFI_LIB_DIR])

# ------------------------------------------------------------------------------
AC_ARG_WITH(efi-ldsdir,
        AS_HELP_STRING([--with-efi-ldsdir=PATH], [Path to efi lds directory]),
        [EFI_LDS_DIR="$withval"],
        [
                for EFI_LDS_DIR in "${efiroot}/gnuefi" "${efiroot}"; do
                    for lds in ${EFI_LDS_DIR}/elf_${ARCH}_efi.lds; do
                            test -f ${lds} && break 2
                    done
                done
        ]
)
AC_SUBST([EFI_LDS_DIR])

# ------------------------------------------------------------------------------
AC_ARG_WITH(efi-includedir,
        AS_HELP_STRING([--with-efi-includedir=PATH], [Path to efi include directory]),
        [EFI_INC_DIR="$withval"], [EFI_INC_DIR="/usr/include"]
)
AC_SUBST([EFI_INC_DIR])

# ------------------------------------------------------------------------------
have_tpm=no
AC_ARG_ENABLE([tpm], AS_HELP_STRING([--enable-tpm], [Enable optional TPM support]),
                [case "${enableval}" in
                        yes) have_tpm=yes ;;
                        no) have_tpm=no ;;
                        *) AC_MSG_ERROR(bad value ${enableval} for --enable-tpm) ;;
                esac],
                [have_tpm=no])

AS_IF([test "x${have_tpm}" != xno], AC_DEFINE(SD_BOOT_LOG_TPM, 1, [Define if TPM should be used to log events and extend the registers]))

AC_ARG_WITH(tpm-pcrindex,
        AS_HELP_STRING([--with-tpm-pcrindex=<NUM>],
                [TPM PCR register number to use]),
        [SD_TPM_PCR="$withval"],
        [SD_TPM_PCR="8"])

AC_DEFINE_UNQUOTED(SD_TPM_PCR, [$SD_TPM_PCR], [TPM PCR register number to use])

# ------------------------------------------------------------------------------
AC_ARG_WITH([bashcompletiondir],
        AS_HELP_STRING([--with-bashcompletiondir=DIR], [bash completions directory]),
        [],
        [AS_IF([$($PKG_CONFIG --exists bash-completion)], [
                with_bashcompletiondir=$($PKG_CONFIG --variable=completionsdir bash-completion)
        ] , [
                with_bashcompletiondir=${datadir}/bash-completion/completions
        ])])
AM_CONDITIONAL(ENABLE_BASH_COMPLETION, [test "$with_bashcompletiondir" != "no"])
AX_NORMALIZE_PATH([with_bashcompletiondir])
AC_SUBST([bashcompletiondir], [$with_bashcompletiondir])

# ------------------------------------------------------------------------------
AC_ARG_WITH([zshcompletiondir],
        AS_HELP_STRING([--with-zshcompletiondir=DIR], [zsh completions directory]),
        [], [with_zshcompletiondir=${datadir}/zsh/site-functions])
AM_CONDITIONAL(ENABLE_ZSH_COMPLETION, [test "$with_zshcompletiondir" != "no"])
AX_NORMALIZE_PATH([with_zshcompletiondir])
AC_SUBST([zshcompletiondir], [$with_zshcompletiondir])

# ------------------------------------------------------------------------------
have_manpages=no
AC_ARG_ENABLE(manpages, AS_HELP_STRING([--disable-manpages], [disable manpages]))
AS_IF([test "x$enable_manpages" != xno], [
        AS_IF([test "x$enable_manpages" = xyes -a "x$XSLTPROC" = x], [
                AC_MSG_ERROR([*** Manpages requested but xsltproc not found])
        ])
        AS_IF([test "x$XSLTPROC" != x], [have_manpages=yes])
])
AM_CONDITIONAL(ENABLE_MANPAGES, [test "x$have_manpages" = "xyes"])

# ------------------------------------------------------------------------------
AC_CONFIG_FILES([
        Makefile
])

AC_OUTPUT
AC_MSG_RESULT([
        $PACKAGE_NAME $VERSION

        prefix:                  ${prefix}
        datarootdir:             ${datarootdir}
        Bash completions dir:    ${with_bashcompletiondir}
        Zsh completions dir:     ${with_zshcompletiondir}

        arch:                    $ARCH
        EFI machine type:        $MACHINE_TYPE_NAME
        EFI libdir:              ${EFI_LIB_DIR}
        EFI ldsdir:              ${EFI_LDS_DIR}
        EFI includedir:          ${EFI_INC_DIR}
        TPM:                     ${have_tpm}

        QEMU:                    ${QEMU}
        QEMU OVMF:               ${QEMU_BIOS}
])
