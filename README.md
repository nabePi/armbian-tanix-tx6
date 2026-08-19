# Armbian Tanix TX6

Build image Armbian untuk STB **Tanix TX6** secara otomatis lewat GitHub Actions — tidak perlu build manual di lokal.

Workflow-nya pakai reusable Action resmi [`armbian/build`](https://github.com/armbian/build), sesuai contoh minimal di dokumentasi Armbian:
- [Build Preparation — Minimal workflow example](https://docs.armbian.com/Developer-Guide_Build-Preparation/#minimal-workflow-example)
- [Build Commands](https://docs.armbian.com/Developer-Guide_Build-Commands/)
- [Build Switches](https://docs.armbian.com/Developer-Guide_Build-Switches/)

## Cara build

1. Buka tab **Actions** di repo ini.
2. Pilih workflow **"Build Armbian Image (Tanix TX6)"**.
3. Klik **Run workflow**, isi input sesuai kebutuhan (default sudah pas untuk server, tanpa desktop):

   | Input           | Default      | Keterangan                                                |
   |-----------------|--------------|-------------------------------------------------------------|
   | `board`         | `tanix-tx6`  | `armbian_board` — target board                               |
   | `kernel_branch` | `current`    | `armbian_kernel_branch` (`current` / `edge` / `legacy`)      |
   | `release`       | `resolute`   | `armbian_release` — codename userspace (Ubuntu/Debian/Armbian) |
   | `ui`            | `server`     | `armbian_ui` — `server` (BUILD_DESKTOP=no, BUILD_MINIMAL=no), `minimal` (rootfs minimal), atau nama desktop environment (mis. `gnome`) |

4. Tunggu sampai job selesai (build kernel + rootfs bisa memakan waktu 1–3+ jam).
5. Image hasil build otomatis di-upload sebagai **GitHub Release** oleh action-nya sendiri.

## Cara kerja workflow

Workflow (`.github/workflows/build-armbian.yml`) hanya memanggil action resmi `armbian/build@main`, yang di baliknya:

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
- Tidak ada secret tambahan yang diperlukan — release memakai `GITHUB_TOKEN` bawaan Actions (`armbian_token`).
- Versi/tag release otomatis dihitung oleh action (`ARMBIAN_VERSION`) kalau `armbian_release_tag` tidak diisi.
