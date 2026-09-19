# Loop Forge

Sequencer dan workstation musik kecil yang berjalan langsung di browser.

Loop Forge sengaja dibuat sederhana: satu file HTML, tanpa framework, tanpa proses build, dan tanpa backend wajib. Buka `index.html` di browser modern lalu mulai membuat loop.

## Yang sudah ada

- 16 track × 16 step
- pattern A/B/C/D dan song chain
- velocity, probability, swing, dan microtiming
- drum synth dan synth dua oscillator
- editor nada/piano roll inline untuk track bernada; drum/percussion tetap memakai step grid
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
- mulai dari loop kosong; tombol **New idea** tetap tersedia bila ingin dibuatkan pola awal

## Menjalankan

Tidak perlu instalasi.

1. Unduh atau clone repository.
2. Buka `index.html` di browser.
3. Klik kotak pada sequencer lalu tekan **Play**.

Sebagian browser membatasi Web MIDI pada konteks tertentu. Fitur musik utama tetap berjalan dengan Web Audio API tanpa MIDI.

## Prinsip

Fokus proyek ini adalah workflow yang mudah dipahami, ukuran kecil, dan fitur yang benar-benar berguna. WebGPU tidak diperlukan untuk fungsi inti dan tidak menjadi dependency aplikasi.

## Status

Versi saat ini: **4.3.1**.
