# openmausbot-repo

Repository pacman untuk Arch Linux yang berisi satu paket, `openmausbot-bin`.

Paket ini dibuat dengan mengemas ulang file `.deb` resmi dari
[milind-soni/OpenMausBot](https://github.com/milind-soni/OpenMausBot) menjadi
`.pkg.tar.zst`. Tidak ada kompilasi dari source dan tidak memakai `debtap` —
cukup ekstrak `data.tar.*` memakai `bsdtar`.

Satu workflow GitHub Actions berjalan tiap 4 jam (dan bisa dipicu manual), lalu:

1. cek rilis terbaru upstream lewat GitHub API;
2. kalau versinya berubah, perbarui `pkgver`, `source`, dan `sha256sums` di `PKGBUILD`;
3. build di container `archlinux:base-devel` sebagai user non-root;
4. buat database pacman dengan `repo-add`;
5. ubah symlink `.db` dan `.files` jadi file biasa (GitHub Releases tidak menyimpan symlink);
6. upload paket + database ke **satu tag tetap bernama `repo`**;
7. commit `PKGBUILD` ke branch `main`.

Karena tag-nya tetap, URL di `pacman.conf` tidak pernah berubah.

## Install

### 1. Tambahkan repo ke `/etc/pacman.conf`

Taruh di bawah bagian repository lain:

```ini
[openmausbot-repo]
SigLevel = Optional TrustAll
Server = https://github.com/adttsonaisse/openmausbot-repo/releases/download/repo
```

`SigLevel = Optional TrustAll` diperlukan karena database dan paket di repo ini
tidak ditandatangani dengan kunci PGP. Artinya pacman akan menerima paket apa pun
yang ada di URL tersebut — hanya pakai ini kalau kamu percaya pada repo ini.

### 2. Sinkronkan database dan install

```bash
sudo pacman -Sy
sudo pacman -S openmausbot-bin
```

Atau, kalau sekalian mau memperbarui seluruh sistem:

```bash
sudo pacman -Syu openmausbot-bin
```

> Hindari menjalankan `pacman -Sy` diikuti `pacman -S <paket>` secara terpisah
> dalam jangka panjang; itu bisa menyebabkan *partial upgrade*. Memakai
> `pacman -Syu` sekali jalan lebih aman.

### 3. Jalankan

```bash
openmausbot
```

Atau lewat menu aplikasi (file `.desktop` ikut terpasang, nama desktop-nya
`com.openmausbot.app.desktop`).

Paket ini hanya tersedia untuk `x86_64` / `amd64`.

## Update

Workflow terjadwal otomatis mengecek rilis baru tiap 4 jam. Untuk memperbarui:

```bash
sudo pacman -Syu
```

Cek status workflow di tab **Actions** repository ini. Tombol **Run workflow**
di tab tersebut bisa dipakai untuk memicu pengecekan secara manual.

## Isi release `repo`

Tag `repo` berisi file-file ini, dan selalu ditimpa setiap build:

| File | Fungsi |
| --- | --- |
| `openmausbot-repo.db` | database pacman (isi sama dengan `.db.tar.gz`) |
| `openmausbot-repo.db.tar.gz` | database pacman, dipakai pacman |
| `openmausbot-repo.files` | indeks daftar file paket |
| `openmausbot-repo.files.tar.gz` | versi terkompresi dari indeks tersebut |
| `openmausbot-bin-<versi>-1-x86_64.pkg.tar.zst` | paketnya sendiri |

File `.db` dan `.files` sengaja disalin dari `*.tar.gz` menjadi file biasa.
`repo-add` awalnya membuat keduanya sebagai symlink, tetapi GitHub Releases
mengabaikan symlink saat upload.

## Build manual (opsional)

Butuh Arch Linux dengan `base-devel` dan `bsdtar` (sudah termasuk di `base`):

```bash
git clone https://github.com/adttsonaisse/openmausbot-repo.git
cd openmausbot-repo
makepkg -si
```

`makepkg` tidak boleh dijalankan sebagai `root`. Kalau terpaksa memakai root,
buat user biasa dulu lalu jalankan `makepkg` lewat user itu.

## Catatan

- `PKGBUILD` hanya mengambil biner yang sudah jadi dari rilis upstream;
  perubahan versi dan hash ditangani otomatis oleh workflow.
- `depends` memakai paket Electron/Chromium yang umum di Arch: `gtk3`,
  `libnotify`, `nss`, `libxss`, `libxtst`, `xdg-utils`, `at-spi2-core`, dan
  `libsecret`. Kalau aplikasi menbutuhkan library tambahan, tambahkan ke array
  `depends` di `PKGBUILD`.
- Ini adalah repack pihak ketiga, bukan rilis resmi OpenMausBot.
