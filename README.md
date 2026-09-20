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

## Riwayat perubahan (terbaru dulu)

### Perbaikan 4.11.1 (UX sanity generator)

Pass ini **bukan penambahan kemampuan musik**. Prinsipnya: kontrol yang terlihat harus benar-benar berpengaruh, dan kontrol yang tampak seperti pilihan tidak boleh diam-diam mengubah project.

- **Sidebar tidak lagi kosong saat pertama dibuka.** Dulu tab `Sound` tampak aktif tetapi panel `#sound` masih `display:none`, jadi editor kanan kosong sampai tab diklik. Sekarang satu fungsi `selectTab()` menggerakkan panel yang tampil dan tab yang aktif bersamaan, dipanggil saat init (dan sesudah restore project/share), jadi keduanya tidak bisa lagi berbeda.
- **Prompt tidak lagi terlihat wajib di Native.** Native bisa generate tanpa mengetik apa pun (mode `New idea` default). Kemampuan teks lokal tetap ada, tapi dipindah ke **More → Local text idea** — bukan lagi kontrol pertama. Alur default kini terasa **pilih → Generate**.
- **`New idea` / `Variation` jadi mode selection, bukan tombol mutation.** Dulu keduanya terlihat seperti radio tapi klik langsung meng-checkpoint/mutate/mengganti part. Sekarang klik mode hanya **memilih**; mutasi hanya terjadi saat CTA footer ditekan. Mode memakai segmented control dengan selected state dan `aria-pressed`.
- **`Fill gaps` bukan mode generation.** Dipindah ke **More & tools** karena implementasinya memang tool untuk selected track (Euclid/Humanize/Fill gaps sekelompok).
- **Variation jujur terhadap `variationToNext()`.** UI menampilkan **From A → B** dan `Similar ↔ Wild` (satu-satunya kontrol yang dibaca handler); Scope dan Change **disembunyikan** di mode Variation karena handler tidak membacanya. CTA menjadi **Create variation → B**.
- **`Change` (target) tidak muncul di New idea.** `generateIdea()` tidak membaca `genTarget`, sehingga kontrol itu tidak lagi dipajang di alur default — capability teks lokal + target tetap ada di More, jujur sebagai "local text idea".
- **Label arah density dibetulkan**: `Busy ↔ simple` → **`Simple ↔ Busy`** (kanan = lebih ramai).
- **UI LLM berbeda dari Native.** Saat Engine=LLM, kontrol yang tidak masuk `aiBuildPayload()` (Style, Density, Similar↔Wild, Change, Seed, rhythm tools) **disembunyikan**. LLM hanya menampilkan: Describe the change, Part, konteks Key/Scale, connection ringkas, dan proposal.
- **Prompt WAJIB untuk LLM, dan jujur.** Saat prompt kosong, CTA **disabled** dan helper inline **"Describe what you want to change."** muncul di dekat field (bukan toast global). Provider tidak dipanggil bila prompt kosong.
- **CTA punya state yang terlihat**: `Ask LLM` → `Asking…` (disabled) → `Apply proposal` → kembali usable saat error. Duplikat klik saat `asking` tidak mengirim request kedua (guard request-id/abort existing tetap utuh).
- **Connection tidak lagi mendominasi.** Setelah profile/key ada, main flow hanya menampilkan **summary ringkas** (`nama · model · state`) + tombol **Configure**. Config (profile/endpoint/model/key/remember/Test/Forget/warning) tetap ada di collapsible yang **auto-open bila belum ada key**, default collapsed bila sudah configured. Status dibedakan jujur: `No key` / `Key set` / `Testing connection…` / `Connection OK` — **key ada tidak pernah diklaim sudah tested**. Security semantics tidak berubah.
- **Proposal user-facing.** Kartu **Proposal** human-readable (Change / Style / Density / Syncopation / Humanize / Variation / Keep) dengan catatan **"Nothing has changed yet."**. Request/JSON mentah pindah ke **Technical details** (collapsed). Schema AI tidak diubah.
- **Feedback dekat CTA.** Baris status di footer menampilkan waiting/error/proposal/applied sehingga user tidak perlu scroll ke blok connection.
- **Toast di atas modal.** `.toast z-index` dinaikkan dari 99 menjadi 200 (> `130` modal) agar notifikasi global tidak tersembunyi di belakang modal; validasi form utama tetap **inline**.
- **Responsif** 375/768/1280: tidak ada horizontal overflow, CTA footer terjangkau, segmented mode tidak terpotong, modal scroll wajar.
- Invariant tetap: Native default & full offline; LLM hanya menghasilkan typed decision; semua mutasi tetap ditulis LocalExecutor/code; transport BPM, audio, WAV/MIDI, dan schema project/share/export tidak berubah.

#### Koreksi lanjutan 4.11.1 (cross-state)

Fresh audit menemukan lima bug lintas-state yang belum tertangkap suite; pass ini hanya memperbaikinya (bukan redesign, bukan fitur musik baru).

- **Local text idea bukan lagi "hidden mode ketiga".** Teks lokal hanya authoritative pada **Native + New idea + Current**. Di **A–D** dan **Variation** blok Local text idea + `Change` **disembunyikan**, dan teks lama yang tersembunyi **tidak lagi memengaruhi routing maupun seed** Variation. Saat teks lokal aktif, baris **Style / Simple↔Busy disembunyikan** (karena `runGen()` mengabaikannya) dan muncul notice **"Local text idea is active…"** — jadi kontrol yang terlihat selalu mencerminkan handler yang benar-benar berjalan.
- **`New connection` memakai lifecycle switch yang sama** dengan ganti profil: membatalkan request in-flight (`AbortController` + request-id), menghapus proposal lama, mengosongkan preview mentah, reset `aiTestedOk`, lalu mengikat state key profil baru. Response lama yang datang terlambat tidak bisa mengisi UI profil baru, dan key/proposal tidak bocor antar-profil.
- **Edit config langsung mencabut "Connection OK".** Setelah Test sukses, mengubah name/endpoint/model langsung me-refresh badge (→ `Key set` / not-tested) dan summary (`name`/`model`/`endpoint`) tanpa menunggu refresh lain.
- **Technical details tidak auto-open saat Ask.** `aiAsk()` mengisi `aiSent`/`aiProposal` mentah tetapi **tidak** memaksa disclosure terbuka; pilihan buka/tutup manual user dihormati. Kartu Proposal human-readable tetap primary.
- **Version consistency:** `APP_VERSION` di-bump ke **4.11.1** (judul, komentar CSS, dan nama file export ikut konsisten otomatis). Project schema version tetap **5** (tidak diubah).

#### Koreksi lanjutan 4.11.1 (control honesty / hidden dependency)

Truth table diperluas dari *direct reads* ke **transitive reads** (call graph). Beberapa kontrol ternyata memengaruhi output lewat rantai fungsi, bukan dibaca di body handler teratas.

- **Variation jujur soal Key/Scale.** `variationToNext()` → `mutatePattern()` → `mutateTrack()` → `scaleNotes($('#key'),$('#scale'))`, jadi pitched variation memang mengikuti Key/Scale. UI Variation kini menampilkan **Musical context (Key + Scale)** di samping From→To dan Similar↔Wild. Style/Density tetap tidak ditampilkan karena Variation tidak membacanya.
- **Seed dipisahkan dari blok Local text** menjadi baris **Generation seed** sendiri di More & tools. Seed dibaca oleh `runGen()` (local text), `generateAllParts()` (A–D), `variationToNext()` (Variation) dan `fillGaps()`. Karena Fill gaps (tool yang selalu ada di Native) membacanya, field seed tetap tersedia di Native, dengan **hint yang eksplisit** menyebut operasi mana yang memakainya — sehingga field yang terlihat tidak pernah diam-diam diabaikan oleh CTA Generate.
- **Local text kosong tidak lagi memajang kontrol yang diabaikan.** Saat textarea Local text **kosong**, `Change` **disembunyikan** (hanya `runGen()` yang membacanya) dan muncul hint *"Add text to use target-specific local generation."*. Saat teks **non-kosong**, `Change` muncul, Style + Simple↔Busy disembunyikan, dan muncul notice aktif. Mengosongkan kembali mengembalikan Style/Density dan routing `generateIdea()`. Mengetik saja tidak mengubah project.
- **A–D menampilkan seluruh dependency transitifnya**: Style, Density (dibaca `populateCurrentIdea()`), Key, Scale, dan Seed — sementara blok Local text + Change tetap tersembunyi dan teks lama tidak relevan.
- **Fill gaps tidak lagi bergantung pada hidden stale local text.** Karena `nativePrompt()` bergantung-pada-state (kosong di luar Native+New+Current), teks lokal yang tersembunyi tidak memengaruhi seed Fill gaps; output identik dengan/tanpa stale text.
- **Hint Seed state-aware dan jujur per operasi.** Klaim reproducibility generik *"same seed + same settings reproduces the same result"* dihapus karena **salah untuk A–D**: di `generateAllParts()` part A dibuat fresh lewat `populateCurrentIdea()` (memakai `Math.random`), dan seed hanya menggerakkan mutasi B–D. Helper kecil `seedHintForState()` memilih copy yang benar: New idea Current tanpa teks → *"Not used by this Generate…"*; local text aktif → reproducible dengan *text/settings* yang sama; Variation → reproducible dari *source part* yang sama; A–D → *"Seed affects the B–D variations. Part A is generated fresh, so A–D is not fully reproducible."* Algoritma/RNG tidak diubah.
- Invariant tetap: algoritma musik tidak berubah; AI lifecycle, provider protocol, key storage, LocalExecutor, AI schema, transport BPM, audio, WAV/MIDI, dan schema project/share/export tidak disentuh. `APP_VERSION` tetap **4.11.1**.

### Baru di 4.11.0 (Generator dialog)

- **Satu entry point**: tombol **Generate** di topbar (menggantikan "New part"). Tombol **Blank loop** tetap.
- **Generator jadi satu dialog/modal** (responsif, mobile-friendly) dengan **dua engine**: **Native** (default, jalan tanpa network/key) dan **LLM**. Keduanya adalah engine dari konsep yang sama, bukan subsystem terpisah.
- **Tab "Idea" di sidebar dihapus.** Sidebar kini hanya Sound / Step(advanced) / Mix / Save.
- **Kontrol dipusatkan di dialog**: What do you want? (satu textarea bersama), Target/scope (current part atau A–D + target role), Feel (Style, Density, Key, Scale, Similar/Wild), Action (New idea / Variation / Fill gaps), serta **satu CTA workflow** di footer. Rhythm tools (Euclid/Humanize) pindah ke collapsible **More controls**.
- **Satu CTA generation saja.** Connection config **tidak** lagi punya tombol Ask/Apply — LLM connection hanya berisi profile, add/remove, name, endpoint, model, API key, remember, Test connection, Forget key, dan warning/security. Tombol **Generate A–D** yang terpisah dihapus: **scope selector yang otoritatif**. Footer menampilkan tepat satu label sesuai state:
  - Native + Current → **[ Generate ]**
  - Native + A–D → **[ Generate A–D ]**
  - LLM + Current (belum ada proposal) → **[ Ask LLM ]**
  - LLM + A–D → **[ LLM A–D not available yet ]** (disabled, jujur — bukan fallback diam-diam ke Native)
  - LLM + proposal siap → **[ Apply proposal ]**, lalu kembali ke **Ask LLM** setelah Apply.
- **Preview di luar connection settings.** "Preview what is sent & what is proposed" pindah ke collapsible dekat footer/main flow. Saat **Ask LLM** diklik, `aiSent` **langsung terisi sebelum fetch**, preview **auto-open**, "Proposed locally" menampilkan **Waiting for provider...**, lalu jadi proposal (sukses) atau pesan error (rejected/gagal). Preview tidak pernah terlihat "tidak terjadi apa-apa".
- **Status LLM eksplisit (state machine)**: `no key` → **"No key for <profile>"**, `has key` → **"Ready · <profile> / <model>"**, plus state testing / asking / proposal ready / error / applied. Teks "LLM off" tidak lagi muncul saat SecretStore sudah punya key.
- **Action hanya untuk Native.** Di Engine=LLM tombol New idea / Variation / Fill gaps **disembunyikan** — karena keduanya hanya memanggil `aiAsk()` yang sama (tidak punya semantik berbeda) dan Fill gaps hanya toast Native. Mode 1 LLM tetap typed reshape.
- **Konsistensi visual tanpa design language baru.** Prompt field dibungkus `label.control` sehingga memakai styling form LOOPFORGE yang sudah ada (`--surface-2`/`--text`/`--border`/`--radius-sm`, focus `--accent-2`, `resize:vertical`). Modal body dan `.aipre` memakai scrollbar **tipis & bertema** (`scrollbar-width:thin` + `scrollbar-color` untuk Firefox, dan `::-webkit-scrollbar` 8px untuk WebKit) — bukan scrollbar browser-default yang terang.
- **LLM config bersih**: saat Engine=Native seluruh noise API/key disembunyikan; saat Engine=LLM muncul **LLM connection** (profile, name, endpoint, model, key, remember, test, forget) sebagai collapsible inline — bukan modal bersarang.
- **Arsitektur**: Native (`controls`/`parseIntent`) dan LLM (`prompt → provider → extractJson → AIValidator → AISemanticValidate`) sama-sama menghasilkan **GeneratorDecision** lalu lewat **LocalExecutor** yang sama. Tidak ada executor duplikat, dan **tidak ada silent engine switch**: memilih LLM + A–D tidak pernah menjalankan `generateAllParts()` Native.
- **Backward compatible**: perilaku generator lokal, AI Mode 1, profile/security, schema project/share/export, dan transport/audio tidak berubah. Membuka dialog tidak mengubah project; satu Generate = satu checkpoint/Undo. Dialog bisa ditutup via Escape, tombol ×, backdrop, dan tombol **Close**.

### Perbaikan 4.10.1 (semantik preserve Mode 1)

- **Token preserve `all` dihapus.** Karena `target` di Mode 1 selalu wajib, `preserve:["all"]` selalu bertentangan dengan target dan tidak punya penggunaan sehat; sekarang ditolak sebagai token tak dikenal.
- **System prompt diperketat**: `preserve` default `[]`; hanya diisi bila user eksplisit minta keep/preserve/jangan ubah; dilarang mengarang preserve dari konteks; dilarang men-preserve seluruh target. Disertai contoh eksplisit.
- **Validasi semantik** (`AISemanticValidate`) berjalan setelah validator struktural: target & preserve di-expand ke role, dan bila **semua** role target tertutup preserve (atau terkunci), proposal **ditolak sebelum tombol Apply aktif**. Overlap parsial tetap boleh (target drums + preserve kick ⇒ hat/perc masih writable).
- **Pesan error spesifik**: *"Model proposed a target that is fully preserved/locked — ask again."*
- **Preserve tidak pernah dibuang diam-diam**: bila keputusan model kontradiktif, kita reject, bukan menghapus preserve yang diminta user.

### Baru di 4.10.0 (koneksi AI multi-provider)

- **OpenAI bukan lagi provider hardcoded**, hanya connection profile default. Semua koneksi memakai **satu adapter OpenAI-compatible Chat Completions** — tanpa protocol baru, tanpa Anthropic native, tanpa Responses API.
- **Connection selector** dengan field: name, **full chat endpoint**, model, API key. Bisa tambah/hapus koneksi; tetap compact & mobile-friendly.
- **Profile config non-secret** (`{id,name,endpoint,model,protocol}`) boleh persist di `localStorage`, tetapi **tidak pernah** masuk `projectData`, export JSON, share link, atau history.
- **Secret per-profile**: key default memory-only, opsi *Remember this key for this tab* memakai `sessionStorage` per-profile. Pindah koneksi memakai key masing-masing tanpa menyalin/membocorkan antar-profile.
- **Endpoint safety**: HTTPS diizinkan; HTTP hanya untuk `localhost`/`127.0.0.1`; `user:pass@` dan scheme non-http(s) ditolak. Endpoint dianggap trust boundary dan UI memperingatkan bila bukan host OpenAI.
- **Request generik**: hanya `model`, `messages`, `temperature` (tanpa mewajibkan `response_format` yang tidak universal), `Authorization: Bearer <key>`, `fetch redirect:'error'`, plus timeout/AbortController/race guard yang sudah ada.
- **Preview privasi** menampilkan endpoint persis dan body tersanitasi — key tidak pernah tampil.
- **Test connection** lewat chat endpoint yang dikonfigurasi (bukan `/models`), dengan pesan error untuk auth/404/429/timeout/network-CORS.
- Invariant tetap: provider hanya menghasilkan typed Mode-1 decision; semua lewat `response → extractJson → AIValidator → proposal → Apply → LocalExecutor`. Tidak ada cabang provider di LocalExecutor.

### Perbaikan 4.9.6 (BPM otoritatif di semua jalur)

- **Export WAV dan MIDI ikut memakai `transportBpm`.** Sebelumnya `renderOffline()` (`+$('#bpm').value||112`) dan `buildMidiBlob()` (`Math.max(20,...)`) masih membaca DOM mentah, jadi input parsial seperti `"8"` bisa menghasilkan WAV 8 BPM (sangat panjang) atau MIDI 20 BPM meski transport realtime tetap 112. Sekarang hanya `transportBpm` yang jadi sumber tempo.
- **Tidak ada lagi konsumen tempo yang membaca `#bpm.value`** selain load project, `commitBpm` saat init, dan elemen UI editor.
- **UI tidak lagi berbohong**: saat blur/change, bila field invalid/parsial, field dipulihkan ke `transportBpm` sehingga yang terlihat sama dengan yang dimainkan/diekspor.

### Perbaikan 4.9.5 (transport BPM live)

- **Scheduler tidak lagi membaca BPM mentah dari DOM.** Ditambah `transportBpm` terpisah; `secondsPerStep()` memakai nilai itu. Input angka hanya boleh meng-*commit* nilai lengkap dalam rentang **40..240**.
- **Nilai parsial saat mengetik diabaikan, bukan di-clamp.** Mengetik `112 → "" → "8" → "80"` tidak pernah membuat tempo menjadi `8`; transport tetap 112 sampai `80` valid. Jadi tidak ada lagi step 1,8 detik yang membuat `nextNoteTime` melompat jauh (penyebab freeze saat menurunkan BPM).
- **Satu snapshot `stepSec` per step**, dipakai untuk increment `nextNoteTime`, swing, dan durasi note bernada — perubahan BPM di tengah tick tidak bisa membelah satu step.
- **Transport tidak di-reset saat BPM berubah**; phase dijaga, `nextNoteTime − ctx.currentTime` tetap dalam bound wajar (lookahead 200 ms dipertahankan).
- **Checkpoint per gestur**: satu gestur edit (fokus + beberapa ArrowUp, atau satu drag) = satu transaksi undo, bukan satu per keypress.

### Perbaikan 4.9.4 (determinisme per-track)

- **Dua track ber-role sama tidak lagi menghasilkan kandidat identik.** Dulu `genRoleTrack` dipanggil dengan `seeded(candSeed)` yang sama untuk setiap target, jadi bass 7 dan bass 8 (atau chord 9 dan chord 10) bisa dapat note/progresi identik dan saling menumpuk. Sekarang tiap track memakai child seed `candSeed ^ hash32('track|'+i+'|'+role)`: tetap reproducible untuk decision/project yang sama, tapi tidak identik antar-track.
- **Blend rank memakai `candSeed`** (yang sengaja tidak memuat `variation`), sehingga proposal berbeda tidak selalu menyerang step/note yang sama, sementara sifat monotonic tetap utuh.

### Perbaikan 4.9.3 (mesin blend variation)

- **Baseline kosong tidak lagi jadi saklar ON/OFF.** Dulu pitched track tanpa note langsung melompat ke kandidat penuh begitu `variation > 0`. Sekarang material kandidat masuk satu per satu sesuai ambang `variation`, jadi `0 → 0.1 → 0.3 → 1.0` benar-benar bertahap.
- **Monotonic benar-benar dijamin.** Tiap event/step punya *rank* deterministik yang tidak bergantung pada `variation`; slider hanya menaikkan ambang. Naikkan nilai = tambah perubahan, tidak pernah mengocok ulang pilihan lama. Seed kandidat juga tidak lagi memuat `variation`.
- **Invariant note tidak dilanggar.** Blend memakai semantik tumpang-tindih interval yang sama dengan `addNote()`: dua note dengan pitch sama tidak boleh saling menimpa waktu (kasus `C4@0 len4` vs `C4@2 len4` kini aman).
- **Stale fingerprint pakai part aktif utuh** (`JSON.stringify(state.patterns[currentPattern])`), jadi perubahan parameter suara apa pun di part aktif ikut terdeteksi, sementara edit part lain tetap tidak membuat stale.
- **Prompt sistem disamakan** dengan vocabulary validator: menyebut token grup `drums`, `bass`, `harmony`, `melodic`, `texture_fx`, `all`.

### Perbaikan 4.9.2 (semantik variation & preserve)

- **`variation` kini relatif ke pola yang sedang ada.** Sebelumnya executor menulis ulang track lalu memutasinya, jadi `variation 0%` bisa mengganti seluruh bass. Sekarang track saat ini menjadi **baseline**: kandidat dibuat terpisah, lalu `variation` menentukan berapa banyak kandidat menggantikan baseline (0% = baseline dipertahankan, 100% = kandidat dipakai penuh). "Buat bass lebih gelap tapi tetap mirip" sekarang benar-benar berarti demikian.
- **Tabrakan namespace role/group dihapus.** Token kini eksklusif: `melody`, `arp`, `texture`, `fx` **selalu** role tunggal; hanya token grup `drums`, `bass`, `harmony`, `melodic`, `texture_fx`, `all` yang diekspansi. Jadi `preserve ["melody"]` = melody saja (bukan melody+arp).
- **Stale-check tidak lagi berlebihan**: fingerprint hanya mencakup part aktif (role/lock/type + steps/noteEvents) dan key/scale. Mengedit part lain (B/C/D) tidak lagi membatalkan proposal untuk part aktif, tetapi mengedit part aktif tetap membatalkannya.

### Perbaikan 4.9.1 (stabilisasi Mode 1)

- **Typed preserve benar-benar typed**: tidak ada lagi konversi ke kalimat `"keep drums bass"`. `expandPreserve()` memperluas grup secara deterministik (`drums→kick/snare/hat/perc`, `harmony→chord`, dst.) dan `isPreserved()` mencocokkan role aktual, jadi `["kick","snare","hat"]` maupun `["drums","bass"]` dihormati persis.
- **Target berbasis role, bukan slot tetap**: `semanticTargetIndexes()` memilih track dari `role` aktual, sehingga memindahkan role antar baris tetap tertangkap AI.
- **`variation` tidak lagi mati**: dipetakan ke mesin Similar↔Wild yang sudah ada lewat `mutateTrack()`/`mutateSelectedTracks()`.
- **Race request AI diperbaiki**: `aiRequestId` + `AbortController` lokal per request; request baru membatalkan yang lama, `Forget key` membatalkan yang sedang jalan, dan respons lama tidak bisa menimpa proposal baru.
- **TOCTOU ditutup**: tiap proposal menyimpan konteks (`pattern/key/scale/state`); kalau konteks berubah sebelum **Apply**, proposal ditolak dengan pesan *stale*.
- **Preview payload persis**: panel menampilkan body yang benar-benar dikirim (termasuk `temperature` dan `response_format`); `Authorization` tidak pernah ditampilkan.

Keamanan (dinyatakan apa adanya): key tidak pernah melewati server milik LOOPFORGE — browser mengirimnya langsung ke OpenAI. Ini **bukan** mekanisme penyimpanan aman: kode pada origin yang sama, extension browser, atau akses perangkat bisa membacanya. Karena itu key tidak pernah masuk ke project, share link, export JSON, history, toast, atau console. OpenAI sendiri menonaktifkan penggunaan SDK di browser secara default karena risiko ini; fitur ini disediakan sebagai alat personal/advanced, bukan jaminan keamanan.

## Baru di 4.9.0 — AI Mode 1 (typed decisions)

Opsional, ada di **More → AI**. Model bahasa hanya menerjemahkan niat manusia menjadi **parameter typed**; generator lokal yang tetap menulis nada. Model menilai, kode mengeksekusi.

- **Provider: OpenAI saja**, endpoint resmi dipin (`https://api.openai.com/v1/chat/completions`), lewat `fetch` langsung tanpa SDK. Tanpa Anthropic, tanpa custom endpoint pada rilis ini.
- **Key default hanya di memori** (hilang saat refresh). **Remember for this tab** opsional memakai `sessionStorage`. Tidak ada penyimpanan permanen.
- **Schema ketat**: hanya `action, target, style, density, syncopation, humanizeMs, variation, preserve`. Field tak dikenal **ditolak**, bukan diabaikan; key `__proto__`/`constructor` diblokir; `target`/`style` enum dari kosakata lokal; `density/syncopation/variation` di-clamp 0..1, `humanizeMs` 0..40.
- **Preview transparan**: panel menampilkan payload yang benar-benar dikirim ke provider dan proposal hasil normalisasi, sebelum **Apply**.
- **Satu Apply = satu `checkpoint()`** — satu Undo mengembalikan keadaan sebelum AI.
- **Lock selalu menang**; `preserve` dihormati.
- **Tanpa fallback diam-diam**: kalau AI gagal atau output ditolak, tidak ada yang diterapkan. Tombol generator lokal tetap tersedia.

## Baru di 4.8.0 — bantuan menambah nada & beat

- **Chord helper di inline piano roll**: tombol maj / min / 7 / maj7 / min7 / sus2 / sus4, plus **Add chord** pada step aktif. Chord mengikuti kunci & skala yang dipilih.
- **Snap to scale**: tombol **Snap all to scale** memperbaiki nada yang di luar skala, dan opsi **Keep notes in scale** membuat klik di piano roll otomatis membulat ke nada terdekat dalam skala.
- **Next note ideas**: saran nada berikutnya yang harmonis (berdasarkan skala dan nada terakhir), sekali klik untuk menambah.
- **Similar ↔ Wild**: kontrol variasi baru menggantikan kekuatan `Variation → next` yang dulu tetap. Geser ke kiri untuk perubahan dekat, ke kanan untuk variasi liar.
- **Humanize sadar peran**: kick/snare minim pergeseran, hat/perc lebih longgar, chord stabil, bass rapat — lewat tabel `HUMANIZE_BY_ROLE`.

## Perbaikan 4.7.6

- **Lifecycle audition bersih**: pemilihan meter preview (`auditionTrack`) kini selalu dilepas saat audition selesai — termasuk **Stop sound** manual pada sample dan saat timer synth berakhir di tengah playback. Timer juga dijaga oleh index track agar timer lama tidak menghapus audition yang lebih baru. Penghentian meter RAF tetap terpisah: hanya berhenti bila tidak ada playback atau preview lain.
- **Analyser tidak basi antar context**: `createAudioContext()` mengatur ulang `previewAnalyser`, `auditionTrack`, dan `auditionTap`, sehingga analyser dari AudioContext lama tidak pernah dipakai pada graph context baru.

## Perbaikan 4.7.5

- **VU per-track ikut audition**: preview sample maupun synth dulu menembus track bus (agar bisa melewati Solo), jadi VU track tidak bergerak. Sekarang audition memakai analyser preview tersendiri dan hasilnya ditampilkan pada meter track yang sedang diaudition, tanpa mengembalikan routing preview ke track bus. Playback normal tetap membaca analyser track bus seperti sebelumnya.

## Perbaikan 4.7.4

- **Visualizer pada synth audition**: **Hear this sound** pada track synth kini menyalakan oscilloscope/VU seperti sample preview, lalu menghentikannya setelah nada selesai (berdasarkan attack+decay+release), kecuali sedang playing atau ada preview yang lebih baru.

## Perbaikan 4.7.3

- **Mixer per-part di export benar**: `renderOffline()` dulu menulis `gain`/`pan` langsung (nilai part terakhir menang untuk seluruh lagu). Sekarang dijadwalkan di tiap batas bar dengan `setValueAtTime(value, barStart)`, jadi volume/pan/mute/solo part A/B/C/D berubah tepat saat part berganti, sama seperti realtime.
- **Kick yang di-mute atau ter-exclude solo tidak lagi memicu duck**: blok sidechain untuk bass stem kini memeriksa `!tr.mute && (!anySolo || tr.solo)` sebelum menjadikan kick sebagai trigger, konsisten dengan full mix.

## Perbaikan 4.7.2

- **Lock menutup celah keyboard**: aktivasi step via Space/Enter (`e.detail===0`) kini menghormati `tr.locked`, bukan hanya klik pointer.
- **Parity level export**: `renderOffline` memakai gain master yang sama dengan realtime (0.82), jadi WAV/stem tidak lagi lebih keras dari yang terdengar saat Play.
- **Stem bass mengikuti sidechain**: saat mengekspor stem bass (atau track lain) dengan Duck aktif, kick dari part lain memicu duck yang terdengar di bus, jadi stem mencerminkan mix terproses, bukan terisolasi-dry.
- **Meter benar-benar start/stop**: loop `requestAnimationFrame` untuk oscilloscope/VU hanya hidup saat playing atau preview dan dihentikan saat Stop, tab disembunyikan, atau preview selesai.
- Catatan desain: membuka shared link lalu tidak mengedit apa pun tidak menimpa project lokal (hash dibersihkan setelah dibaca, `restoring` mencegah autosave) — disengaja agar link orang lain aman dibuka.

## Perbaikan 4.7.1

- **Share link tidak lagi menyandera reload**: **Copy share link** tidak mengubah address bar, dan setelah link dibuka hash dibersihkan (dengan penanda sesi sebagai cadangan) sehingga edit setelah membuka link tetap menang saat reload.
- **Autosave menyeluruh**: perubahan BPM, swing, echo, duck, mixer, slider Sound, microtiming, dan chain/key/scale kini memakai `markDirty()` terpusat, bukan hanya bergantung pada render.
- **Parity export**: WAV/stem offline sekarang menyertakan master echo dan sidechain duck, jadi hasil render sama dengan yang terdengar saat Play.
- **Gesture mobile**: sentuhan satu jari menggulir/mengetuk; long-press membuka editor; drag-paint dibatasi untuk mouse/pen.
- **Lock konsisten**: row terkunci tidak bisa di-paint atau diedit dari popover/Step; popover menawarkan **Unlock track**.
- **MIDI**: export memakai format 1 (satu track per baris + conductor) dengan drum di channel 10 General MIDI; track melodik menghindari channel 10.
- **Performa**: oscilloscope dan VU meter hanya berjalan saat playing atau preview, dan berhenti saat tab tidak aktif.
- Link share terkompresi memakai `deflate-raw`; browser tanpa `DecompressionStream` diberi pesan jelas alih-alih diam-diam membuka autosave.

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

## Menjalankan

Tidak perlu instalasi.

1. Unduh atau clone repository.
2. Buka `index.html` di browser.
3. Klik kotak pada sequencer lalu tekan **Play**.

Sebagian browser membatasi Web MIDI pada konteks tertentu. Fitur musik utama tetap berjalan dengan Web Audio API tanpa MIDI.

## Prinsip

Fokus proyek ini adalah workflow yang mudah dipahami, ukuran kecil, dan fitur yang benar-benar berguna. WebGPU tidak diperlukan untuk fungsi inti dan tidak menjadi dependency aplikasi.

## Status

Versi saat ini: **4.11.1**.
