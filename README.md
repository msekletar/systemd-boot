# systemd-boot

systemd-boot is a simple UEFI Boot Manager. Its main job is to lunch
selected boot menu entry. systemd-boot leverages APIs provided by UEFI firmware
and offloads all the heavy lifting to a firmware (e.g. loading files from disk,
executing binaries). This allows for very minimal implementation, but feature
complete to support common usecases (desktop, laptop).

systemd-boot fully supports The Freedesktop Boot Loader Specification [[1]](https://www.freedesktop.org/wiki/Specifications/BootLoaderSpec/).

## Installation

```
./autogen.sh
./configure --prefix=/usr
make
sudo make install
```

Note that installation of sytemd-boot to the EFI System Partition must be
handled separately. It is usually done by ```bootctl``` command line utility
from systemd package.

### Dependencies

* gnu-efi
* xsltproc (optional)
* pkg-config (optional)
* qemu (optional)
* OVMF (optional)

## Authors

* Kay Sievers
* Harald Hoyer
* ... and many others (for complete list see git history at [[2]](https://www.github.com/systemd/systemd))

