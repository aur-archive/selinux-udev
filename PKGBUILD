# $Id: PKGBUILD 159405 2012-05-23 20:09:25Z tomegun $
# Maintainer: Tom Gundersen <teg@jklm.no>
# Contributor: Aaron Griffin <aaron@archlinux.org>
# Contributor: Tobias Powalowski <tpowa@archlinux.org>
# Contributor: Thomas Bächler <thomas@archlinux.org>
# Maintainer: Nicky726 <nicky726@gmail.com>

pkgname=selinux-udev
_origname=udev
pkgver=182
pkgrel=4
pkgdesc="The userspace dev tools (udev) with SELinux support"
depends=('selinux-usr-libselinux' 'selinux-util-linux' 'glib2' 'kmod' 'hwids' 'bash' 'acl')
conflicts=("${_origname}")
provides=("${_origname}=${pkgver}-${pkgrel}")
install=udev.install
arch=(i686 x86_64)
license=('GPL')
makedepends=('gobject-introspection' 'gperf' 'libxslt')
source=(ftp://ftp.kernel.org/pub/linux/utils/kernel/hotplug/$_origname-$pkgver.tar.xz
	0001-split-usr-always-read-config-files-from-lib-udev.patch
	0002-reinstate-TIMEOUT-handling.patch
	initcpio-hooks-udev
	initcpio-install-udev)
url="http://git.kernel.org/?p=linux/hotplug/udev.git;a=summary"
backup=(etc/udev/udev.conf)
groups=('selinux' 'selinux-system-utilites')
options=(!makeflags !libtool)
md5sums=('023877e6cc0d907994b8c648beab542b'
         '0fa3eac115ad0140af1582d941b15f2c'
         '94b816896bf23275c0598fc8e07270c3'
         'e433c11d38cf4f877b41d06e2753ebe0'
         'e6faf4c3fe456f10d8efd2487d5e3cb7')

build() {
  cd $srcdir/$_origname-$pkgver

  patch -p1 -i ../0001-split-usr-always-read-config-files-from-lib-udev.patch
  patch -p1 -i ../0002-reinstate-TIMEOUT-handling.patch

  ./configure --prefix=/usr \
              --sysconfdir=/etc \
              --libexecdir=/usr/lib \
              --with-systemdsystemunitdir=/usr/lib/systemd/system \
              --with-firmware-path=/usr/lib/firmware/updates:/lib/firmware/updates:/usr/lib/firmware:/lib/firmware \
              --with-usb-ids-path=/usr/share/hwdata/usb.ids \
              --with-pci-ids-path=/usr/share/hwdata/pci.ids \
              --with-selinux

  make
}

check() {
  make -C "$_origname-$pkgver" check
}
 
package() {
  cd $srcdir/$_origname-$pkgver
  make DESTDIR=${pkgdir} install

  # install the mkinitpcio hook
  install -D -m644 ../initcpio-hooks-udev ${pkgdir}/usr/lib/initcpio/hooks/udev
  install -D -m644 ../initcpio-install-udev ${pkgdir}/usr/lib/initcpio/install/udev

  # udevd moved, symlink to make life easy for restarting udevd manually
  ln -s ../lib/udev/udevd ${pkgdir}/usr/bin/udevd

  # the path to udevadm is hardcoded in some places
  install -d ${pkgdir}/sbin
  ln -s ../usr/bin/udevadm ${pkgdir}/sbin/udevadm

  # fix wrong path to /bin/sh
  sed -i -e 's#/usr/bin/sh#/bin/sh#g' $pkgdir/usr/lib/udev/keyboard-force-release.sh

  # Replace dialout/tape/cdrom group in rules with uucp/storage/optical group
  for i in $pkgdir/usr/lib/udev/rules.d/*.rules; do
    sed -i -e 's#GROUP="dialout"#GROUP="uucp"#g;
               s#GROUP="tape"#GROUP="storage"#g;
               s#GROUP="cdrom"#GROUP="optical"#g' $i
  done
}
