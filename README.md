# Armbian Tanix TX6

## Tentang repo ini

Repo ini **tidak berisi source code Armbian** — isinya cuma satu GitHub Actions workflow (`.github/workflows/build-armbian.yml`) yang men-trigger build image Armbian untuk STB **Tanix TX6** secara otomatis di server GitHub, tanpa perlu install toolchain atau build manual di komputer lokal.

Saat workflow dijalankan, GitHub Actions akan:
1. Menyiapkan runner Linux.
2. Mengambil source Armbian build framework dari [`armbian/build`](https://github.com/armbian/build).
3. Meng-compile image Armbian khusus board `tanix-tx6` (server, tanpa desktop).
4. Mem-publish image hasil build ke tab **Releases** repo ini, siap di-download dan di-flash.

Konfigurasinya mengikuti dokumentasi resmi Armbian:
- [Build Preparation — Minimal workflow example](https://docs.armbian.com/Developer-Guide_Build-Preparation/#minimal-workflow-example)
- [Build Commands](https://docs.armbian.com/Developer-Guide_Build-Commands/)
- [Build Switches](https://docs.armbian.com/Developer-Guide_Build-Switches/)

## Cara build image

1. Buka repo ini di GitHub, klik tab **Actions** (di bagian atas, sebelah "Pull requests").
2. Di sidebar kiri, pilih workflow **"Build Armbian Image (Tanix TX6)"**.
3. Klik tombol **Run workflow** (kanan atas daftar run) → pilih branch `main` → akan muncul form input.
4. Isi form sesuai kebutuhan, atau biarkan default (sudah diset untuk build **server, tanpa desktop**):

   | Input           | Default      | Keterangan                                                |
   |-----------------|--------------|-------------------------------------------------------------|
   | `board`         | `tanix-tx6`  | `armbian_board` — target board                               |
   | `kernel_branch` | `current`    | `armbian_kernel_branch` (`current` / `edge` / `legacy`)      |
   | `release`       | `resolute`   | `armbian_release` — codename userspace (Ubuntu/Debian/Armbian) |
   | `ui`            | `server`     | `armbian_ui` — `server` (tanpa desktop, tanpa rootfs minimal), `minimal` (rootfs minimal), atau nama desktop environment (mis. `gnome`) |

5. Klik tombol hijau **Run workflow** untuk mulai.
6. Klik run yang baru muncul untuk memantau progres build secara real-time (biasanya **1–3+ jam**, tergantung board dan beban runner).
7. Setelah run selesai (centang hijau ✅), buka tab **Releases** di repo ini — image hasil build (`.img.xz` dkk) sudah otomatis ter-upload di situ, tinggal di-download.
8. Flash file image tersebut ke SD card / eMMC Tanix TX6 pakai tool seperti [Balena Etcher](https://etcher.balena.io/) atau `dd`.

## Cara kerja workflow

Workflow hanya memanggil action resmi `armbian/build@main`, yang di baliknya:

1. Membersihkan ruang disk runner (`armbian_runner_clean: "yes"`, direkomendasikan untuk GitHub-hosted runner).
2. Clone framework `armbian/build` dan menjalankan `./compile.sh build` dengan variabel yang dipetakan dari input di atas — setara dengan:
   ```bash
   ./compile.sh \
     BOARD=tanix-tx6 \
     BRANCH=current \
     RELEASE=resolute \
     BUILD_MINIMAL=no \
     BUILD_DESKTOP=no
   ```
3. Membuat GitHub Release dan meng-upload image dari `output/images/` sebagai asset-nya — semua ditangani action, tidak perlu step tambahan di workflow kita.

## Catatan / batasan

- Runner GitHub Actions publik dibatasi **maksimum 6 jam per job**. Build board dengan kernel besar berisiko kehabisan waktu/ruang meski sudah dibantu `armbian_runner_clean`.
- Tidak ada secret tambahan yang perlu disiapkan — release memakai `GITHUB_TOKEN` bawaan Actions (`armbian_token`).
- Versi/tag release otomatis dihitung oleh action (`ARMBIAN_VERSION`) kalau `armbian_release_tag` tidak diisi.
