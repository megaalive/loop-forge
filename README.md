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
- audit fungsional v4.6.0: SAMPLE tanpa buffer tetap senyap, playhead/chain memakai timer terkelola, lock konsisten, note events dibersihkan saat konversi ke sample, dan piano dock lama dihapus
- pada piano roll: klik untuk tambah/hapus nada, drag horizontal untuk mengatur panjang nada

## Baru di 4.7.0

- **Autosave & restore**: project otomatis tersimpan di browser dan dipulihkan saat dibuka lagi; mode More/easy juga diingat
- **Undo berlabel**: toast undo/redo menyebut aksi yang dibatalkan (mis. "Undo: Mute")
- **Editor step popover**: klik-kanan (atau long-press) pada step membuka velocity, probability, microtiming, note, slice, copy/paste, dan audition tanpa pindah tab
- **Paint gesture**: drag melintasi grid untuk menyalakan/mematikan banyak step; **Alt+drag** mengatur velocity secara vertikal; Shift+klik tetap memutar strength
- **Master oscilloscope** di status bar dan **VU meter per track** di mixer
- **Share link**: seluruh pola dikodekan ke URL (`#p=...`) dan bisa disalin lewat **Copy share link**
- **Waveform sampler**: bentuk gelombang dengan 16 pembatas slice; klik area untuk menetapkan slice ke step aktif
- **Export MIDI**: file `.mid` standar dari chain A–D, termasuk swing dan microtiming
- **Master echo** (send delay) dan **sidechain Duck** (kick menekan bass) untuk kedalaman mix
- **Forge flash**: step yang dihasilkan berkedip singkat sebagai umpan balik visual; chip prompt cepat di panel Idea
- aksen warna per-role pada baris track dan indikator velocity di dalam step

## Perbaikan 4.7.1

- **Share link tidak lagi menyandera reload**: **Copy share link** tidak mengubah address bar, dan setelah link dibuka hash dibersihkan (dengan penanda sesi sebagai cadangan) sehingga edit setelah membuka link tetap menang saat reload.
- **Autosave menyeluruh**: perubahan BPM, swing, echo, duck, mixer, slider Sound, microtiming, dan chain/key/scale kini memakai `markDirty()` terpusat, bukan hanya bergantung pada render.
- **Parity export**: WAV/stem offline sekarang menyertakan master echo dan sidechain duck, jadi hasil render sama dengan yang terdengar saat Play.
- **Gesture mobile**: sentuhan satu jari menggulir/mengetuk; long-press membuka editor; drag-paint dibatasi untuk mouse/pen.
- **Lock konsisten**: row terkunci tidak bisa di-paint atau diedit dari popover/Step; popover menawarkan **Unlock track**.
- **MIDI**: export memakai format 1 (satu track per baris + conductor) dengan drum di channel 10 General MIDI; track melodik menghindari channel 10.
- **Performa**: oscilloscope dan VU meter hanya berjalan saat playing atau preview, dan berhenti saat tab tidak aktif.
- Link share terkompresi memakai `deflate-raw`; browser tanpa `DecompressionStream` diberi pesan jelas alih-alih diam-diam membuka autosave.

## Perbaikan 4.7.2

- **Lock menutup celah keyboard**: aktivasi step via Space/Enter (`e.detail===0`) kini menghormati `tr.locked`, bukan hanya klik pointer.
- **Parity level export**: `renderOffline` memakai gain master yang sama dengan realtime (0.82), jadi WAV/stem tidak lagi lebih keras dari yang terdengar saat Play.
- **Stem bass mengikuti sidechain**: saat mengekspor stem bass (atau track lain) dengan Duck aktif, kick dari part lain memicu duck yang terdengar di bus, jadi stem mencerminkan mix terproses, bukan terisolasi-dry.
- **Meter benar-benar start/stop**: loop `requestAnimationFrame` untuk oscilloscope/VU hanya hidup saat playing atau preview dan dihentikan saat Stop, tab disembunyikan, atau preview selesai.
- Catatan desain: membuka shared link lalu tidak mengedit apa pun tidak menimpa project lokal (hash dibersihkan setelah dibaca, `restoring` mencegah autosave) — disengaja agar link orang lain aman dibuka.

## Perbaikan 4.7.3

- **Mixer per-part di export benar**: `renderOffline()` dulu menulis `gain`/`pan` langsung (nilai part terakhir menang untuk seluruh lagu). Sekarang dijadwalkan di tiap batas bar dengan `setValueAtTime(value, barStart)`, jadi volume/pan/mute/solo part A/B/C/D berubah tepat saat part berganti, sama seperti realtime.
- **Kick yang di-mute atau ter-exclude solo tidak lagi memicu duck**: blok sidechain untuk bass stem kini memeriksa `!tr.mute && (!anySolo || tr.solo)` sebelum menjadikan kick sebagai trigger, konsisten dengan full mix.

## Perbaikan 4.7.4

- **Visualizer pada synth audition**: **Hear this sound** pada track synth kini menyalakan oscilloscope/VU seperti sample preview, lalu menghentikannya setelah nada selesai (berdasarkan attack+decay+release), kecuali sedang playing atau ada preview yang lebih baru.

## Perbaikan 4.7.5

- **VU per-track ikut audition**: preview sample maupun synth dulu menembus track bus (agar bisa melewati Solo), jadi VU track tidak bergerak. Sekarang audition memakai analyser preview tersendiri dan hasilnya ditampilkan pada meter track yang sedang diaudition, tanpa mengembalikan routing preview ke track bus. Playback normal tetap membaca analyser track bus seperti sebelumnya.

## Perbaikan 4.7.6

- **Lifecycle audition bersih**: pemilihan meter preview (`auditionTrack`) kini selalu dilepas saat audition selesai — termasuk **Stop sound** manual pada sample dan saat timer synth berakhir di tengah playback. Timer juga dijaga oleh index track agar timer lama tidak menghapus audition yang lebih baru. Penghentian meter RAF tetap terpisah: hanya berhenti bila tidak ada playback atau preview lain.
- **Analyser tidak basi antar context**: `createAudioContext()` mengatur ulang `previewAnalyser`, `auditionTrack`, dan `auditionTap`, sehingga analyser dari AudioContext lama tidak pernah dipakai pada graph context baru.

## Baru di 4.8.0 — bantuan menambah nada & beat

- **Chord helper di inline piano roll**: tombol maj / min / 7 / maj7 / min7 / sus2 / sus4, plus **Add chord** pada step aktif. Chord mengikuti kunci & skala yang dipilih.
- **Snap to scale**: tombol **Snap all to scale** memperbaiki nada yang di luar skala, dan opsi **Keep notes in scale** membuat klik di piano roll otomatis membulat ke nada terdekat dalam skala.
- **Next note ideas**: saran nada berikutnya yang harmonis (berdasarkan skala dan nada terakhir), sekali klik untuk menambah.
- **Similar ↔ Wild**: kontrol variasi baru menggantikan kekuatan `Variation → next` yang dulu tetap. Geser ke kiri untuk perubahan dekat, ke kanan untuk variasi liar.
- **Humanize sadar peran**: kick/snare minim pergeseran, hat/perc lebih longgar, chord stabil, bass rapat — lewat tabel `HUMANIZE_BY_ROLE`.

## Baru di 4.9.0 — AI Mode 1 (typed decisions)

Opsional, ada di **More → AI**. Model bahasa hanya menerjemahkan niat manusia menjadi **parameter typed**; generator lokal yang tetap menulis nada. Model menilai, kode mengeksekusi.

- **Provider: OpenAI saja**, endpoint resmi dipin (`https://api.openai.com/v1/chat/completions`), lewat `fetch` langsung tanpa SDK. Tanpa Anthropic, tanpa custom endpoint pada rilis ini.
- **Key default hanya di memori** (hilang saat refresh). **Remember for this tab** opsional memakai `sessionStorage`. Tidak ada penyimpanan permanen.
- **Schema ketat**: hanya `action, target, style, density, syncopation, humanizeMs, variation, preserve`. Field tak dikenal **ditolak**, bukan diabaikan; key `__proto__`/`constructor` diblokir; `target`/`style` enum dari kosakata lokal; `density/syncopation/variation` di-clamp 0..1, `humanizeMs` 0..40.
- **Preview transparan**: panel menampilkan payload yang benar-benar dikirim ke provider dan proposal hasil normalisasi, sebelum **Apply**.
- **Satu Apply = satu `checkpoint()`** — satu Undo mengembalikan keadaan sebelum AI.
- **Lock selalu menang**; `preserve` dihormati.
- **Tanpa fallback diam-diam**: kalau AI gagal atau output ditolak, tidak ada yang diterapkan. Tombol generator lokal tetap tersedia.

### Perbaikan 4.9.1 (stabilisasi Mode 1)

- **Typed preserve benar-benar typed**: tidak ada lagi konversi ke kalimat `"keep drums bass"`. `expandPreserve()` memperluas grup secara deterministik (`drums→kick/snare/hat/perc`, `harmony→chord`, dst.) dan `isPreserved()` mencocokkan role aktual, jadi `["kick","snare","hat"]` maupun `["drums","bass"]` dihormati persis.
- **Target berbasis role, bukan slot tetap**: `semanticTargetIndexes()` memilih track dari `role` aktual, sehingga memindahkan role antar baris tetap tertangkap AI.
- **`variation` tidak lagi mati**: dipetakan ke mesin Similar↔Wild yang sudah ada lewat `mutateTrack()`/`mutateSelectedTracks()`.
- **Race request AI diperbaiki**: `aiRequestId` + `AbortController` lokal per request; request baru membatalkan yang lama, `Forget key` membatalkan yang sedang jalan, dan respons lama tidak bisa menimpa proposal baru.
- **TOCTOU ditutup**: tiap proposal menyimpan konteks (`pattern/key/scale/state`); kalau konteks berubah sebelum **Apply**, proposal ditolak dengan pesan *stale*.
- **Preview payload persis**: panel menampilkan body yang benar-benar dikirim (termasuk `temperature` dan `response_format`); `Authorization` tidak pernah ditampilkan.

Keamanan (dinyatakan apa adanya): key tidak pernah melewati server milik LOOPFORGE — browser mengirimnya langsung ke OpenAI. Ini **bukan** mekanisme penyimpanan aman: kode pada origin yang sama, extension browser, atau akses perangkat bisa membacanya. Karena itu key tidak pernah masuk ke project, share link, export JSON, history, toast, atau console. OpenAI sendiri menonaktifkan penggunaan SDK di browser secara default karena risiko ini; fitur ini disediakan sebagai alat personal/advanced, bukan jaminan keamanan.

## Menjalankan

Tidak perlu instalasi.

1. Unduh atau clone repository.
2. Buka `index.html` di browser.
3. Klik kotak pada sequencer lalu tekan **Play**.

Sebagian browser membatasi Web MIDI pada konteks tertentu. Fitur musik utama tetap berjalan dengan Web Audio API tanpa MIDI.

## Prinsip

Fokus proyek ini adalah workflow yang mudah dipahami, ukuran kecil, dan fitur yang benar-benar berguna. WebGPU tidak diperlukan untuk fungsi inti dan tidak menjadi dependency aplikasi.

## Status

Versi saat ini: **4.9.1**.
