# Armbian Tanix TX6

Build image Armbian untuk STB **Tanix TX6** secara otomatis lewat GitHub Actions — tidak perlu build manual di lokal.

Workflow-nya mengikuti panduan resmi Armbian:
- [Build Preparation](https://docs.armbian.com/Developer-Guide_Build-Preparation/)
- [Build Commands](https://docs.armbian.com/Developer-Guide_Build-Commands/)
- [Build Switches](https://docs.armbian.com/Developer-Guide_Build-Switches/)

## Cara build

1. Buka tab **Actions** di repo ini.
2. Pilih workflow **"Build Armbian Image (Tanix TX6)"**.
3. Klik **Run workflow**, isi input sesuai kebutuhan (default sudah pas untuk server, tanpa desktop):

   | Input              | Default      | Keterangan                                   |
   |--------------------|--------------|-----------------------------------------------|
   | `board`            | `tanix-tx6`  | Target board                                   |
   | `branch`           | `current`    | Kernel branch (`current` / `edge` / `legacy`)  |
   | `release`          | `resolute`   | Codename userspace (Ubuntu/Debian/Armbian)     |
   | `build_minimal`    | `no`         | `yes` = rootfs minimal                         |
   | `build_desktop`    | `no`         | `yes` = build image desktop                    |
   | `kernel_configure` | `no`         | `yes` = buka menuconfig kernel secara interaktif |

4. Tunggu sampai job selesai (build kernel + rootfs bisa memakan waktu 1–3+ jam).
5. Image hasil build otomatis di-upload sebagai **GitHub Release** baru (tag `build-<nomor_run>`) di tab **Releases**.

## Cara kerja workflow

Workflow (`.github/workflows/build-armbian.yml`) berjalan di runner `ubuntu-22.04` dan melakukan:

1. Membersihkan ruang disk bawaan runner (Android SDK, .NET, Haskell, dll) supaya ada cukup ruang untuk build.
2. Clone framework resmi [`armbian/build`](https://github.com/armbian/build).
3. Menjalankan build non-interaktif:
   ```bash
   sudo ./compile.sh \
     BOARD=tanix-tx6 \
     BRANCH=current \
     RELEASE=resolute \
     BUILD_MINIMAL=no \
     BUILD_DESKTOP=no \
     KERNEL_CONFIGURE=no
   ```
4. Mencari image hasil build di `output/images/` dan mem-publish-nya sebagai GitHub Release.

## Catatan / batasan

- Runner GitHub Actions publik dibatasi **maksimum 6 jam per job** dan disk kosong terbatas (~14GB, sudah dibantu step pembersihan). Build board dengan kernel besar berisiko kehabisan waktu/ruang.
- `BRANCH` ditambahkan sebagai parameter wajib (tidak ada di command awal) supaya `compile.sh` benar-benar berjalan tanpa menu interaktif.
- Tidak ada secret tambahan yang diperlukan — release memakai `GITHUB_TOKEN` bawaan Actions.
