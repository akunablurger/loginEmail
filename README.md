# Gmail login + AI Studio keepalive

Automation Playwright untuk:

- Login ke Google.
- Membuka beberapa URL AI Studio.
- Menekan tombol `Continue to the app`.
- Menekan tombol `Skip` jika tersedia.
- Menjaga halaman tetap terbuka selama jangka waktu yang dikonfigurasi.
- Reload halaman secara berkala (default 15 menit).
- Menjalankan beberapa URL secara paralel.

## Cara kerja

Workflow biasanya dijalankan berulang (mis. cron) dan menjalankan beberapa job secara paralel: setiap job menggunakan 1 akun dan membuka beberapa page/tab dalam satu browser context akun.

Contoh konfigurasi lama yang dipakai di CI di-define melalui environment variables.

## Run outputs (perubahan penting)

- Setiap kali skrip dijalankan, skrip membuat folder baru di `runs/<timestamp>/` (contoh: `runs/2026-10-17T08-30-00-000`).
- Semua screenshot yang dihasilkan (mis. `opened-...png`, `minute-3-...png`, `reload-...png`) disimpan ke folder run tersebut.
- Setelah sesi selesai, skrip menulis file `links-<timestamp>.txt` di dalam folder run. Setiap baris berisi:
  - `<link>  <filename>`
  - Jika `PUBLIC_BASE_URL` diset, link berupa `PUBLIC_BASE_URL/<encoded-filename>`.
  - Jika `PUBLIC_BASE_URL` tidak diset, link berupa `file://<absolute-path>`.

Catatan: skrip hanya menulis file; jika Anda ingin link HTTP yang dapat diakses dari mesin lain, jalankan server static atau atur `PUBLIC_BASE_URL` yang menunjuk ke lokasi di mana folder `runs/` dilayani (mis. upload atau static server).

## Secret / Environment variables yang diperlukan

- AI_STUDIO_URL
  - Daftar URL target (bisa dipisah koma atau baris baru). Contoh:
    ```text
    https://url1,https://url2,https://url3
    ```

- GMAIL_ACCOUNT_LIST
  - Satu atau lebih akun dalam format `email:password`, satu per baris. Contoh (gunakan placeholder, jangan commit kredensial):
    ```text
    your-email@example.com:yourpassword
    another@example.com:anotherpassword
    ```
  - Jika memungkinkan gunakan Playwright storage state (`GMAIL_STORAGE_STATE`) untuk menghindari login lewat username/password.

- OPTIONAL: PUBLIC_BASE_URL
  - Base URL publik yang akan dipakai untuk membentuk link ke file screenshot di run folder. Jika tidak diset, links akan menggunakan `file://` path lokal.

- OPTIONAL: RELOAD_INTERVAL_MINUTES
  - Interval reload halaman dalam menit. Default: `15`.

- OPTIONAL: KEEP_OPEN_MINUTES
  - Total menit setiap browser context akan tetap terbuka. Default: `350`.

- OPTIONAL: EMAIL_INDEX
  - Pilih akun (1-based index) bila Anda menjalankan skrip untuk satu akun saja locally.

## Pengaturan durasi (dokumentasi)

Default yang direkomendasikan di workflow / env:

```text
KEEP_OPEN_MINUTES=350
RELOAD_INTERVAL_MINUTES=15
```

Artinya:
- Browser hidup selama `KEEP_OPEN_MINUTES` menit (default 350).
- Halaman reload setiap `RELOAD_INTERVAL_MINUTES` menit (default 15).

## Menjalankan secara lokal

1) Install dependencies dan Playwright:

```bash
npm ci
npx playwright install chromium
```

2) Contoh menjalankan lokal (gunakan placeholder akun, jangan commit password):

```bash
RELOAD_INTERVAL_MINUTES=3 KEEP_OPEN_MINUTES=6 EMAIL_INDEX=1 \
GMAIL_ACCOUNT_LIST='your-email@example.com:yourpassword' \
AI_STUDIO_URL='https://aistudio.google.com/apps/your-app' \
node scripts/login-and-click.js
```

3) Setelah run selesai, cek folder `runs/<timestamp>/` yang dibuat pada awal run. Buka file `links-<timestamp>.txt` di folder tersebut untuk daftar link ke semua screenshot yang dihasilkan selama run.

Jika Anda ingin agar link bisa dibuka via HTTP dari mesin lain, jalankan static server pada root repo (atau upload `runs/<timestamp>/` ke storage publik) dan set `PUBLIC_BASE_URL` ke root tersebut.

## Catatan penting

- Jangan menyimpan kredensial (email/password) di repo. Gunakan secrets / environment variables di CI.
- Skrip menulis file debug (HTML, URL) bila terjadi error agar analisis lebih mudah.
- Jika butuh fitur upload otomatis (S3/GCS) atau commit hasil run ke repo, saya bisa bantu menambahkan — tetapi ini membutuhkan kredensial/hak akses tambahan.
