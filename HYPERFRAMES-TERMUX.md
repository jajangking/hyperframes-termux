# HyperFrames di HP Android (Termux) — Catatan Instalasi

Hasil eksperimen 2026-10-14. Perangkat: Android aarch64, Termux native + proot Alpine 3.24.

## Kesimpulan

- Render HyperFrames **bisa jalan di HP** via **Termux native** (bukan proot Alpine).
- Chromium Alpine di proot **tidak bisa** dipakai (crash `Trace/breakpoint trap`,
  syscall `sched_getscheduler` diblokir proot, `/dev/shm` tidak ada).

## Prasyarat (Termux native)

| Komponen | Status |
|---|---|
| Node.js | bawaan Termux (`v26.4.0` saat dicoba) |
| Chromium | `pkg install tur-repo x11-repo chromium` → `155.0.8059.39`, headless OK |
| FFmpeg | opsional (`pkg install ffmpeg`; atau Alpine `ffmpeg 8.1.2` untuk pasca-proses) |
| Docker | tidak ada / tidak perlu untuk render lokal |
| HyperFrames Studio AppImage | tidak bisa (hanya x86_64) |

## Instalasi (sesi Termux native; matikan VPN yang menyela DNS dulu)

```sh
pkg install tur-repo x11-repo chromium
npm install -g hyperframes
```

Verifikasi browser:

```sh
chromium-browser --headless --no-sandbox --disable-dev-shm-usage \
  --disable-gpu --dump-dom about:blank
# harus keluar: <html><head></head><body></body></html>
# (error DBus hanya noise, abaikan)
```

## Render

HyperFrames tidak mengenal platform `android arm64`
(`hyperframes browser ensure` → `Unsupported platform`),
jadi arahkan ke binary sistem secara manual:

```sh
export HYPERFRAMES_BROWSER_PATH=$PREFIX/bin/chromium-browser
hyperframes init demo -e blank
cd demo
hyperframes render
```

Bukti sukses: `renders/demo_2026-10-14_19-58-09.mp4`
(`h264 1920x1080 30fps, 10.0s`, 300 frame, ~65 detik,
mode `screenshot capture`, software GPU) — terverifikasi via `ffprobe`.

## Error yang diketahui (non-fatal)

- `EACCES: permission denied, link ...` saat scaffold dan pasca-render:
  keterbatasan symlink. Render tetap sukses, MP4 valid.
- Error DBus/NetworkManager dari Chromium: noise, abaikan.

## Yang tidak jalan

- Chromium Alpine (`/usr/bin/chromium`, v152) di proot: crash di semua kombinasi
  flag. Render dengan `HYPERFRAMES_BROWSER_PATH=/usr/bin/chromium`
  → `Chrome not found`.
- `hyperframe` (Python, library HTTP/2) adalah hal lain — bukan aplikasi video ini.
- VPS `muse` (Tailnet `100.82.83.36` / `fd7a:115c:a1e0::ed3b:5325`):
  ping OK tapi SSH (22/2222) `Connection refused` — `sshd` belum listen di
  interface Tailscale. Perbaikan di sisi VPS.
