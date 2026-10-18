# igcli — Instagram CLI ala wacli

Command-line tool untuk Instagram: baca profil, kirim/baca DM, posting
reels/foto/story, verifikasi prospek, dan outreach DM otomatis — dengan
struktur perintah gaya [wacli](https://wacli.sh) (subcommand, `--json`,
`doctor`, local store).

> **Butuh konektor:** `igcli` adalah pembungkus di atas CLI konektor Instagram
> yang sudah terhubung (`instagram-cli` + `instagram-messages-cli`). Ia tidak
> menyimpan kredensial sendiri.

## Instalasi

```bash
cp igcli ~/workspace/bin/igcli   # atau ke PATH lain
chmod +x ~/workspace/bin/igcli
igcli doctor
```

## Perintah

```
igcli accounts                          # akun + status messaging/profesional
igcli auth status                       # cek autentikasi
igcli profile --username <handle>       # lihat profil
igcli resolve <handle>                  # handle -> fbid (+ followers, private)
igcli prospect <handle>                 # verifikasi calon prospek + heuristik

igcli dm inbox [--first N]              # daftar thread DM
igcli dm thread --id <thread_fbid>      # baca thread
igcli dm send --to <@handle|fbid> --text "..." [--dry-run]
igcli dm outreach [--dry-run] [--max-per-day N]

igcli post reel --file v.mp4 --cover c.jpg --caption "..." [--dry-run]
igcli post photo --file a.jpg [--file b.jpg] --caption "..." [--dry-run]
igcli post story --file s.jpg [--editor-json ...] [--draft]

igcli posts --username <handle> [--first N]   # daftar postingan akun
igcli comments --post-id <id> [--first N]     # ambil komentar postingan
igcli insights [--period last_7_days]         # insight akun sendiri
igcli bio --text "..." [--dry-run]            # update bio (maks 150 karakter)

igcli sync [--first N]                  # sync inbox -> store lokal
igcli store                             # info store lokal
igcli doctor                            # diagnostik
```

Flag global: `--account <@username|fbid>` (default `@ahliwebmurah_id`),
`--json` untuk output machine-readable.

## Catatan penting

- **DM ke orang asing via API hanya bisa dari akun profesional.**
  Akun non-profesional diblokir server IG (HTTP 403). Karena itu
  `dm outreach` default mengirim via akun profesional yang terhubung.
- Perintah `post` memakai lockfile (1 posting dalam satu waktu) dan selalu
  mendukung `--dry-run`. Hormati limit harian Instagram.
- Baca `REVIEW.md` untuk analisis keamanan lengkap (cap, jeda, risiko flag).

## Batasan (dari konektor Instagram)

Tidak bisa via CLI: search user/hashtag, follow/unfollow, like, comment,
reply comment, edit caption, hapus post. Untuk itu gunakan browser.

## Lisensi

MIT
