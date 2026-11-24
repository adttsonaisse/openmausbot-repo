# Maintainer: adttsonaisse <adttsonaisse@users.noreply.github.com>
# Repack dari .deb upstream. Tidak memakai debtap dan tidak build dari source.

pkgname=openmausbot-bin
pkgver=0.1.180
pkgrel=1
pkgdesc='A local-first chat app for running a team of AI agents (repack dari .deb resmi)'
arch=('x86_64')
url='https://github.com/milind-soni/OpenMausBot'
license=('Apache-2.0')
depends=(
  'gtk3'
  'libnotify'
  'nss'
  'libxss'
  'libxtst'
  'xdg-utils'
  'at-spi2-core'
  'libsecret'
)
options=('!strip' '!debug')
noextract=("OpenMausBot-${pkgver}-amd64.deb")
source=("OpenMausBot-0.1.180-amd64.deb::https://github.com/milind-soni/OpenMausBot/releases/download/v0.1.180/OpenMausBot-0.1.180-amd64.deb")
sha256sums=('a6bd2ab584de3bab8329ecbe1fcf91d1e5868344805f0e30364392af9f5c8acc')

_deb="OpenMausBot-${pkgver}-amd64.deb"

prepare() {
  cd "$srcdir"

  # File .deb adalah arsip ar. Keluarkan control.tar.* dan data.tar.*
  bsdtar -xf "$_deb"
}

package() {
  cd "$srcdir"

  if ! compgen -G 'data.tar.*' > /dev/null; then
    echo 'data.tar.* tidak ditemukan setelah ekstraksi .deb' >&2
    return 1
  fi

  # Isi data.tar.* langsung dipindahkan ke pkgdir
  bsdtar -xf data.tar.* -C "$pkgdir"

  # Semua direktori bisa dibaca/dilalui
  find "$pkgdir" -type d -exec chmod 755 {} +

  # File yang di .deb memang ber-execute bit: pertahankan execute bit-nya
  find "$pkgdir" -type f -perm /111 -exec chmod a+x {} +

  # Shared library ikut perlu +x supaya dlopen bekerja
  find "$pkgdir" -type f \( -name '*.so' -o -name '*.so.*' \) -exec chmod a+x {} +

  # .desktop dan aset data tidak boleh executable
  find "$pkgdir" -type f \( -name '*.desktop' -o -name '*.png' -o -name '*.svg' -o -name '*.json' \) -exec chmod 644 {} +

  # Entry point di /usr/bin wajib executable. Symlink tidak terkena -type f,
  # jadi targetnya yang sudah diurus pada langkah sebelumnya.
  if [[ -d "$pkgdir/usr/bin" ]]; then
    find "$pkgdir/usr/bin" -type f -exec chmod 755 {} +
  fi

  # .deb upstream tidak membuat launcher di /usr/bin. Ambil path binary dari
  # file .desktop supaya tetap benar kalau upstream pindah direktori install.
  desktop=$(find "$pkgdir/usr/share/applications" -maxdepth 1 -name '*.desktop' -print -quit 2>/dev/null || true)
  if [[ -n "$desktop" ]]; then
    exec_path=$(sed -n 's/^Exec=\([^ ]*\).*/\1/p' "$desktop" | head -n1)
    if [[ -n "$exec_path" && -e "$pkgdir$exec_path" ]]; then
      mkdir -p "$pkgdir/usr/bin"
      ln -sfn "$exec_path" "$pkgdir/usr/bin/openmausbot"
    fi
  fi
}
