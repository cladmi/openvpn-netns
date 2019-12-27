# This is currently a testing PKGBUILD file.
# Maintainer: Your Name <your.name@example.com>
pkgname=openvpn-netns-git
pkgver=20191227173910
pkgrel=1
pkgdesc=""
arch=('any')
url=""
license=('Unlicense')
groups=()
depends=('socat')
makedepends=('git') # 'bzr', 'git', 'mercurial' or 'subversion'
provides=("${pkgname%-git}")
conflicts=("${pkgname%-git}")
replaces=()
backup=()
options=()
install=
#source=('FOLDER::VCS+URL#FRAGMENT')
source=()
noextract=()
#md5sums=('SKIP')
md5sums=()

pkgver() {
	date +%Y%m%d%H%M%S
}

package() {
	cd "$startdir"

	mkdir -p "$pkgdir"/usr/lib/systemd/system/
	cp services/netns@.service "$pkgdir"/usr/lib/systemd/system/
	cp services/socat-netns@.service "$pkgdir"/usr/lib/systemd/system/

	mkdir -p "$pkgdir"/usr/share/"$pkgname"/
	cp templates/service-netns.conf "$pkgdir"/usr/share/"$pkgname"/

	mkdir -p "$pkgdir"/usr/bin
	cp openvpn-netns-ip "$pkgdir"/usr/bin
	cp openvpn-netns-nameserver "$pkgdir"/usr/bin
}
