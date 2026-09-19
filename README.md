# Loop Forge

Sequencer dan workstation musik kecil yang berjalan langsung di browser.

Loop Forge sengaja dibuat sederhana: satu file HTML, tanpa framework, tanpa proses build, dan tanpa backend wajib. Buka `index.html` di browser modern lalu mulai membuat loop.

## Yang sudah ada

- 16 track × 16 step
- pattern A/B/C/D dan song chain
- velocity, probability, swing, dan microtiming
- drum synth dan synth dua oscillator
- piano roll inline polifonik untuk track bernada: chord, panjang nada, dan beberapa nada pada step yang sama; drum/percussion tetap memakai step grid
- sampler dengan 16 slice
- mixer volume, pan, mute, dan solo
- Euclidean rhythm dan humanize
- MIDI input dan musical typing
- generator pola/variasi lokal
- track lock agar bagian yang sudah jadi tidak ikut berubah
- undo/redo
- simpan project di browser dan import/export JSON
- export WAV dan stem
- mode mudah untuk penggunaan langsung, serta kontrol lanjutan melalui tombol **More**
- mulai dari loop kosong; **New part** membuat pola hanya untuk part aktif
- **Generate A–D** membuat A sebagai dasar lalu B/C/D sebagai variasi yang tetap dapat diedit
- **Play all** memainkan A → B → C → D berurutan lalu mengulang
- kompatibilitas iOS lama: Web Audio dibuka lewat gesture pengguna dan media channel fallback; Web MIDI tetap bergantung dukungan browser
- optimisasi playback untuk perangkat lama: noise buffer dicache, playhead diperbarui secara incremental, dan render antar-part ditunda dari jalur audio
- transport UI menampilkan indikator posisi part/step (`A · 01/16`), playhead pada grid/header, serta state Play/Pause yang sinkron
- Mute/Solo bekerja realtime melalui track bus yang sama di board dan mixer; preview sample bisa dihentikan dengan **Stop sound**
- sample yang baru di-load default ke **Whole sound**; preview sample memakai jalur audition terpisah agar tidak ikut tersangkut state track bus
- pada piano roll: klik untuk tambah/hapus nada, drag horizontal untuk mengatur panjang nada

## Menjalankan

Tidak perlu instalasi.

1. Unduh atau clone repository.
2. Buka `index.html` di browser.
3. Klik kotak pada sequencer lalu tekan **Play**.

Sebagian browser membatasi Web MIDI pada konteks tertentu. Fitur musik utama tetap berjalan dengan Web Audio API tanpa MIDI.

## Prinsip

Fokus proyek ini adalah workflow yang mudah dipahami, ukuran kecil, dan fitur yang benar-benar berguna. WebGPU tidak diperlukan untuk fungsi inti dan tidak menjadi dependency aplikasi.

## Status

Versi saat ini: **4.5.9**.
