# openmausbot-repo

Repository pacman custom untuk Arch Linux (`x86_64`) yang berisi paket:

| Paket | Sumber upstream | Deskripsi |
| --- | --- | --- |
| `openmausbot-bin` | [milind-soni/OpenMausBot](https://github.com/milind-soni/OpenMausBot) | Local-first chat app untuk tim AI agent |
| `lastwave-bin` | [Clash-Projects/LastWave-Desktop](https://github.com/Clash-Projects/LastWave-Desktop) | Desktop YouTube Music client (WinUI, Bit-Perfect, Lossless) |

Semua paket dikemas ulang dari file `.deb` resmi upstream menjadi `.pkg.tar.zst`
tanpa kompilasi dari source dan tanpa `debtap` — cukup ekstrak `data.tar.*`
memakai `bsdtar`.

Satu workflow GitHub Actions berjalan setiap 4 jam (dan bisa dipicu manual), lalu:

1. cek rilis terbaru tiap upstream di `packages.json` lewat GitHub API;
2. untuk paket yang versinya berubah (atau belum terbit di release `repo`), perbarui `pkgver`, `source`, dan `sha256sums` di `packages/<pkgname>/PKGBUILD`;
3. build paket yang berubah di container `archlinux:base-devel` sebagai user non-root;
4. perbarui database pacman secara inkremental dengan `repo-add`;
5. ubah symlink `.db` dan `.files` jadi file biasa (GitHub Releases tidak menyimpan symlink);
6. upload paket baru + database ke **satu tag tetap bernama `repo`** (paket lain yang tidak berubah tetap dipertahankan);
7. commit perubahan `PKGBUILD` ke branch `main`.

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
sudo pacman -Syu openmausbot-bin
```

Atau untuk `lastwave-bin`:

```bash
sudo pacman -Syu lastwave-bin
```

Atau install keduanya sekaligus:

```bash
sudo pacman -Syu openmausbot-bin lastwave-bin
```

> Hindari menjalankan `pacman -Sy` diikuti `pacman -S <paket>` secara terpisah
> dalam jangka panjang; itu bisa menyebabkan *partial upgrade*. Memakai
> `pacman -Syu` sekali jalan lebih aman.

### 3. Jalankan

- OpenMausBot: `openmausbot` (atau dari menu aplikasi: `com.openmausbot.app.desktop`)
- LastWave: `lastwave` (atau dari menu aplikasi: `com.lastwave.lastwave_desktop.desktop`)

Semua paket di repo ini hanya tersedia untuk `x86_64` / `amd64`.

## Update

Workflow terjadwal otomatis mengecek rilis baru tiap 4 jam. Untuk memperbarui di sistem Arch kamu:

```bash
sudo pacman -Syu
```

Cek status workflow di tab **Actions** repository ini. Tombol **Run workflow**
di tab tersebut bisa dipakai untuk memicu pengecekan secara manual.

## Isi release `repo`

Tag `repo` berisi file-file ini:

| File | Fungsi |
| --- | --- |
| `openmausbot-repo.db` | database pacman (isi sama dengan `.db.tar.gz`) |
| `openmausbot-repo.db.tar.gz` | database pacman, dipakai pacman |
| `openmausbot-repo.files` | indeks daftar file paket |
| `openmausbot-repo.files.tar.gz` | versi terkompresi dari indeks tersebut |
| `openmausbot-bin-<versi>-1-x86_64.pkg.tar.zst` | paket OpenMausBot |
| `lastwave-bin-<versi>-1-x86_64.pkg.tar.zst` | paket LastWave |

File `.db` dan `.files` sengaja disalin dari `*.tar.gz` menjadi file biasa.
`repo-add` awalnya membuat keduanya sebagai symlink, tetapi GitHub Releases
mengabaikan symlink saat upload.

## Build manual (opsional)

Butuh Arch Linux dengan `base-devel` dan `bsdtar` (sudah termasuk di `base`):

```bash
git clone https://github.com/adttsonaisse/openmausbot-repo.git
cd openmausbot-repo/packages/lastwave-bin   # atau packages/openmausbot-bin
makepkg -si
```

`makepkg` tidak boleh dijalankan sebagai `root`. Kalau terpaksa memakai root,
buat user biasa dulu lalu jalankan `makepkg` lewat user itu.

## Catatan

- Tiap `PKGBUILD` hanya mengambil biner yang sudah jadi dari rilis upstream;
  perubahan versi dan hash ditangani otomatis oleh workflow.
- Untuk menambah paket baru berbasis `.deb`, buat `packages/<pkgname>/PKGBUILD`
  lalu daftarkan entri baru di `packages.json`.
- Ini adalah repack pihak ketiga, bukan rilis resmi OpenMausBot maupun LastWave.
