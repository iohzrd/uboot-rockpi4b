# U-Boot: Rock Pi 4B based on PKGBUILD for RockPro64
# Maintainer: Dan Johansen <strit@manjaro.org>
# Contributor: Kevin Mihelich 
# Contributor: Adam <adam900710@gmail.com>
# Contributor: Dragan Simic <dsimic@buserror.io>

pkgname=uboot-rockpi4b
pkgver=2022.07
pkgrel=1
_tfaver=2.7.0
pkgdesc="U-Boot for Radxa Rock Pi 4B"
arch=('aarch64')
url='http://www.denx.de/wiki/U-Boot/WebHome'
license=('GPL')
makedepends=('git' 'dtc' 'bc')
provides=('uboot')
conflicts=('uboot')
replaces=('uboot-rockpi4')
install=${pkgname}.install
source=("ftp://ftp.denx.de/pub/u-boot/u-boot-${pkgver/rc/-rc}.tar.bz2"
        "https://git.trustedfirmware.org/TF-A/trusted-firmware-a.git/snapshot/trusted-firmware-a-${_tfaver}.tar.gz"
        "0001-mmc-sdhci-allow-disabling-sdma-in-spl.patch")    # From list: https://patchwork.ozlabs.org/project/uboot/patch/20220222013131.3114990-3-pgwipeout@gmail.com/
sha256sums=('92b08eb49c24da14c1adbf70a71ae8f37cc53eeb4230e859ad8b6733d13dcf5e'
            '553eeca87d4296cdf37361079d1a6446d4b36da16bc25feadd7e465537e7bd4d'
            '7014c3f1ada93536787a4ce30b484dfe651c339391bd46869c61933825a0edcc')

prepare() {
  cd u-boot-${pkgver/rc/-rc}

  patch -N -p1 -i "${srcdir}/0001-mmc-sdhci-allow-disabling-sdma-in-spl.patch"    # RK3399 suspend/resume
}

build() {
  echo "build"
  # Avoid build warnings by editing a .config option in place instead of
  # appending an option to .config, if an option is already present
  update_config() {
    if ! grep -q "^$1=$2$" .config; then
      if grep -q "^# $1 is not set$" .config; then
        sed -i -e "s/^# $1 is not set$/$1=$2/g" .config
      elif grep -q "^$1=" .config; then
        sed -i -e "s/^$1=.*/$1=$2/g" .config
      else
        echo "$1=$2" >> .config
      fi
    fi
  }

  unset CFLAGS CXXFLAGS CPPFLAGS LDFLAGS

  cd trusted-firmware-a-${_tfaver}

  echo -e "\nBuilding TF-A for Radxa Rock Pi 4B...\n"
  make PLAT=rk3399
  # make CROSS_COMPILE=aarch64-linux-gnu- PLAT=rk3399
  cp build/rk3399/release/bl31/bl31.elf ../u-boot-${pkgver/rc/-rc}

  cd ../u-boot-${pkgver/rc/-rc}

  echo -e "\nBuilding U-Boot for Radxa Rock Pi 4B...\n"
  make rock-pi-4-rk3399_defconfig

  update_config 'CONFIG_IDENT_STRING' '" Manjaro Linux ARM"'
  update_config 'CONFIG_OF_LIBFDT_OVERLAY' 'y'
  update_config 'CONFIG_SPL_MMC_SDHCI_SDMA' 'n'
  update_config 'CONFIG_MMC_SDHCI_SDMA' 'y'
  update_config 'CONFIG_MMC_SPEED_MODE_SET' 'y'
  update_config 'CONFIG_MMC_IO_VOLTAGE' 'y'
  update_config 'CONFIG_MMC_UHS_SUPPORT' 'y'
  update_config 'CONFIG_MMC_HS400_ES_SUPPORT' 'y'
  update_config 'CONFIG_MMC_HS400_SUPPORT' 'y'

  make EXTRAVERSION=-${pkgrel}
}

package() {
  echo "package"
  cd u-boot-${pkgver/rc/-rc}

  mkdir -p "${pkgdir}/boot/extlinux"
  install -D -m 0644 idbloader.img u-boot.itb -t "${pkgdir}/boot"
}
