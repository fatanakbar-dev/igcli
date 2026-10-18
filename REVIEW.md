# Safety Review — igcli v0.1.0

Analisis keamanan untuk automasi Instagram via `igcli`.
Ditulis 2026-10-18. Prinsip: **mulai kecil, pantau, naikkan hanya berdasar data.**

## 1. DM Outreach (`dm send`, `dm outreach`)

**Risiko utama:** Instagram menandai akun sebagai spam → pembatasan kirim,
message request diblokir massal, atau shadowban.

**Fakta yang sudah diverifikasi:**
- Akun **non-profesional** (`@ahliwebmurah_id`) DIBLOKIR server IG untuk DM ke
  orang asing via API (HTTP 403 "invitation to chat is accepted"). Thread
  undangan terbentuk tapi pesan tidak terkirim.
- Akun **profesional** (`@bikinweb.murah`) BISA — pesan diterima server
  (message_id dikembalikan). Karena itu `dm outreach` default pakai akun ini.

**Pengaman yang sudah dipasang:**
| Lapisan | Detail |
|---|---|
| Cap harian keras | 10/hari, dihitung dari log file (bukan memori) |
| Jendela waktu | Hanya 07:00–15:00 WIB |
| Jeda | Interval 45 mnt + jitter acak 0–20 mnt |
| Anti-duplikat | 1 prospek tidak pernah di-DM 2x (cek log) |
| Validasi target | Skip otomatis bila handle tidak bisa di-resolve ke fbid |
| Template | Personal per kategori usaha, tanpa link, minta izin dulu |
| Default mati | Cron dalam keadaan DISABLED sampai pemilik menyalakan |

**Yang masih perlu diwaspadai:**
- Jika ada penerima me-report sebagai spam, **turunkan volume** (mis. 5/hari)
  atau jeda beberapa hari. Jangan naikkan cap tanpa data 1–2 minggu.
- Pengiriman via CLI memicu kartu konfirmasi (`userConfirmation`). Pada cron
  detached yang tidak bisa menjawab kartu, kirim bisa menggantung — pastikan
  alur approval beres sebelum mengaktifkan automasi penuh.
- Pelajaran dari insiden Threads 2026-10-17 (23 reply/hari → risiko flag):
  cap kecil + jeda acak bukan jaminan, tetap pantau.

## 2. Posting (`post reel`, `post photo`, `post story`)

**Risiko utama:** dobel-publish (IG punya limit harian; duplikat merusak feed).

**Pengaman yang sudah dipasang:**
- `post.lock` — hanya 1 proses posting dalam satu waktu; lock kedaluwarsa
  1 jam bila proses crash.
- File media WAJIB di `~/workspace/` (ditolak bila di `/tmp`).
- `--dry-run` / `--draft` untuk cek sebelum publish betulan.
- Tidak ada retry otomatis (`--retries` selalu 0) — retry bisa menduplikasi.

**Aturan pakai:**
- Selalu `--dry-run` dulu untuk format/konten baru.
- Jangan jadwalkan 2 posting berbarengan.
- Publish meminta approval tiap kali kecuali pemilik mengizinkan untuk task itu.

## 3. Perintah baca (`profile`, `posts`, `comments`, `insights`, `inbox`, …)

Aman (read-only). Hormati `429 Too Many Requests`: setop semua panggilan
bila kena rate limit, jangan diakali lewat retry cepat.

## 4. Kredensial & data

- `igcli` **tidak menyimpan kredensial sendiri** — auth 100% lewat connector
  Hatch yang sudah terhubung. Tidak ada secret di kode → aman untuk repo publik.
- Store lokal di `~/.local/state/igcli/` dan log outreach di
  `~/workspace/ahliwebmurah/db/` — keduanya DI LUAR repo (lihat `.gitignore`).
- Jangan commit: `__pycache__/`, `*.pyc`, `*.log`, file `*.bak`.
