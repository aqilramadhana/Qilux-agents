<div align="center">

<img src="https://arr-official.id/uploads/Pdaqet.jpg" alt="QILUX AGENT" width="100%">

<br>

**AI coding agent untuk terminal Linux, dengan pelindung VPS bawaan dan persetujuan di setiap langkah berisiko.**

*Satu file Python. Berjalan di VPS lewat SSH. Tampilan piksel.*

<br>

![versi](https://img.shields.io/badge/versi-5.1.0-a06bff?style=for-the-badge)
![python](https://img.shields.io/badge/python-3.10%2B-00b8ff?style=for-the-badge&logo=python&logoColor=white)
![platform](https://img.shields.io/badge/Ubuntu%20%2F%20Debian-00ffd5?style=for-the-badge&logo=ubuntu&logoColor=black)
![lisensi](https://img.shields.io/badge/lisensi-MIT-6a8cff?style=for-the-badge)

[Fitur](#fitur-unggulan) · [Kenapa berbeda](#kenapa-qilux-berbeda) · [Instalasi](#instalasi) · [Provider](#provider-api) · [Shield](#qilux-shield) · [Perintah](#perintah) · [Mode](#mode) · [Batasan](#batasan)

</div>

---

## Fitur unggulan

| | |
|---|---|
| **Qilux Shield** | Pelindung VPS aktif: deteksi brute force dan port scan, blokir lewat **fail2ban + firewall (ufw)**, skor keamanan 0-100, hardening yang disetujui per langkah. |
| **Daemon + Telegram** | Shield bisa berjalan sebagai service systemd yang tetap hidup saat Qilux ditutup. Peringatan serangan dikirim ke Telegram, dan kamu bisa memberi perintah dari sana (`/status`, `/bans`, `/unban`, `/hunt`). |
| **Persetujuan bertingkat** | Agent boleh mengerjakan apa pun, **tapi bagian sensitif selalu berhenti** dan menunggu pilihanmu: *Lanjut sekali / Lanjut & izinkan / Lewati / Hentikan*. |
| **Pemburu ancaman** | Mencari proses mencurigakan (reverse shell, cryptominer, program dari `/tmp`) dan mendeteksi persistence baru: file SUID/SGID, service systemd, `ld.so.preload`, kunci SSH, cron. |
| **Berpikir dalam** | Tiga level (`normal`, `dalam`, `ekstrem`) dengan rencana, antisipasi gagal, dan **verifikasi mandiri** sebelum menyatakan selesai. |
| **Tampilan terminal** | Pikiran mengalir real time, kode yang sedang ditulis tampil live dengan syntax highlight, animasi berpikir, dan maskot kucing hitam. |
| **Multi-provider** | Satu agent, banyak penyedia API kompatibel OpenAI. API key tersimpan di `~/.qilux/.env.local`. |
| **Dua engine** | Engine **qilux** (API langsung) atau engine **opencode** (meneruskan ke `opencode run`). |
| **Plugin dan `/evolve`** | Minta fitur baru, Qilux menuliskannya sebagai plugin terpisah. Inti tidak tersentuh. |

---

## Kenapa Qilux berbeda?

Banyak agent terminal sudah bisa membaca file dan menjalankan perintah. Yang membuat Qilux berbeda adalah **kombinasi** hal berikut dalam satu file yang mudah dibaca dan dimodifikasi:

1. **Agent yang menjaga servernya sendiri.** Shield bekerja bareng fail2ban dan firewall, lengkap dengan pengaman supaya kamu tidak terkunci dari VPS-mu.
2. **Persetujuan, bukan sekadar izin.** Rahasia, kirim data keluar, ubah sistem, hapus data, dan install dari internet selalu butuh keputusanmu, bahkan saat mode `AUTO`.
3. **Tetap menjaga saat kamu pergi.** Mode daemon membuat perlindungan jalan terus, dan Telegram mengabarimu saat ada serangan.
4. **Tahan gangguan.** Stream putus? Retry otomatis, lalu pindah ke model aktif lain. `/health` menandai model yang hidup dan yang mati.
5. **Aman terhadap dirinya sendiri.** Inti `qilux.py` dilindungi, ada backup otomatis, `/rollback`, `/undo`, dan penyamaran API key di output tool.

> Catatan jujur: ini bukan klaim "satu-satunya di dunia". Sebagian fitur ada di agent lain. Nilai Qilux ada pada gabungannya.

---

## Instalasi

Butuh Python 3.10+ (Ubuntu/Debian).

```bash
apt install -y python3-venv git
python3 -m venv ~/qilux-env
~/qilux-env/bin/pip install -U openai rich pillow

git clone GANTI-DENGAN-URL-REPO.git
cd NAMA-FOLDER-REPO
echo "alias qilux='~/qilux-env/bin/python ~/NAMA-FOLDER-REPO/qilux.py'" >> ~/.bashrc
source ~/.bashrc
```

`pillow` opsional (dipakai untuk lampiran gambar).

Jalankan di **tmux** supaya sesi tidak mati saat koneksi putus:

```bash
tmux new -s qilux
qilux
```

Periksa kondisi semua komponen kapan saja dengan `/doctor`.

> Pengguna lama Myosa: folder `~/.myosa` otomatis dipindahkan ke `~/.qilux` saat pertama dijalankan.

---

## Provider API

Qilux bisa memakai penyedia API apa pun yang kompatibel dengan OpenAI. Key tiap provider disimpan di satu file:

```bash
nano ~/.qilux/.env.local
chmod 600 ~/.qilux/.env.local
```

```ini
# ~/.qilux/.env.local  (RAHASIA, jangan pernah di-commit)
ZYVOR_API_KEY=isi_key_lengkap
APINEX_API_KEY=isi_key_lengkap
```

Nama variabel dibuat dari nama provider: `nama` menjadi `NAMA_API_KEY`.

```text
/provider                  pilih / tambah / hapus provider (menu panah)
/provider add apinex       tambah provider (preset APInex tersedia)
/provider add NAMA         tambah provider lain yang kompatibel OpenAI
/provider use NAMA         beralih provider
/provider list | test      daftar provider / uji koneksi
/env                       lihat status key (disamarkan)
/env reload                muat ulang setelah mengedit .env.local
```

- Key yang berisi karakter tidak valid (misalnya `…` dari key yang terpotong) otomatis ditolak dengan pesan yang jelas.
- Status model dan model terakhir disimpan **terpisah per provider**.
- Alternatif cepat tanpa file: `ZKEY=isi_key python qilux.py`. Alamat router bisa diganti dengan variabel `ZURL`.

---

## Qilux Shield

Shield memantau VPS dan bekerja sama dengan fail2ban dan firewall.

| Level | Perilaku |
|-------|----------|
| `pasif` | Hanya memberi peringatan, tidak memblokir apa pun. |
| `waspada` | Blokir otomatis jika skor brute force >= 15 per 10 menit atau scan >= 25 port. Menyalakan ulang fail2ban jika mati. |
| `agresif` *(bawaan)* | Blokir di skor >= 8 atau scan >= 12 port. Pelanggar berulang masuk **firewall** dengan durasi 7 kali lebih lama. fail2ban dan ufw dinyalakan ulang otomatis jika mati. |

**Yang dipantau:** brute force SSH, port scan, login berhasil setelah banyak gagal, akun UID 0 asing, port baru terbuka, perubahan `sshd_config`, `cron`, `sudoers`, `authorized_keys`, serta persistence (SUID/SGID, systemd, `ld.so.preload`).

**Pengaman supaya kamu tidak terkunci:**
- IP whitelist, **semua koneksi SSH yang sedang aktif**, IP server sendiri, dan jaringan privat **tidak pernah diblokir**.
- Blokir otomatis dibatasi per jam dan kedaluwarsa otomatis.
- Yang diblokir adalah **IP tunggal**, bukan seluruh ASN atau rentang besar.

```text
/shield                  menu Shield
/shield status           skor keamanan dan kondisi pertahanan
/shield harden           perkuat VPS (kamu setujui tiap langkah)
/shield level            pilih pasif / waspada / agresif
/shield allow IP         whitelist IP (anti-terkunci)
/shield bans             IP yang sedang diblokir
/shield ban IP           blokir manual   (/shield unban IP untuk membuka)
/shield hunt             cari proses mencurigakan
/shield persist          cek perubahan SUID, systemd, kunci SSH, cron
/shield daemon install   pasang Shield sebagai service (butuh root)
/telegram setup          hubungkan bot Telegram
```

### Daemon dan Telegram

Tanpa daemon, Shield hanya aktif selama Qilux terbuka. Dengan daemon, Shield berjalan sebagai service systemd dan tetap hidup walau SSH putus.

```text
/shield daemon install     pasang dan jalankan service
/shield daemon             status service
/shield daemon logs        log terbaru
/shield daemon uninstall   hapus service
```

Setelah `/telegram setup` (buat bot lewat `@BotFather`, Qilux membantu mendeteksi ID owner-mu), peringatan serangan masuk ke Telegram. Saat daemon aktif, kamu juga bisa memberi perintah dari chat:

```text
/status   ringkasan server        /bans      IP yang diblokir
/alerts   5 peringatan terakhir   /unban IP  buka blokir
/hunt     proses mencurigakan     /level     pasif|waspada|agresif
/persist  cek persistence         /kill PID  hentikan proses yang ditandai Shield
```

Hanya chat pribadi dari owner yang dilayani. Pesan dari ID lain diabaikan dan dicatat di log. `/kill` hanya bisa menghentikan PID yang sudah ditandai Shield.

**Urutan aktivasi yang disarankan:**

```bash
tmux new -s qilux
qilux
/shield allow IP-KAMU      # daftarkan IP-mu dulu
/shield harden             # fail2ban agresif, log firewall, ufw limit
/shield level              # pilih level respons
/telegram setup            # notifikasi serangan
/shield daemon install     # perlindungan tetap hidup
```

---

## Perintah

| Perintah | Fungsi |
|----------|--------|
| `/help` | Daftar perintah |
| `/model`, `/models` | Pilih model (menu panah, status aktif/mati) |
| `/health [kata]` | Cek model mana yang hidup |
| `/provider`, `/env`, `/key` | Kelola penyedia API dan key |
| `/engine` | Pilih engine: qilux / opencode |
| `/mode` | `dev`, `ops`, `cyber` |
| `/deep` | Kedalaman berpikir: normal / dalam / ekstrem |
| `/shield` | Pelindung VPS |
| `/telegram` | Notifikasi dan bot Telegram |
| `/audit [host]` | Audit keamanan read-only, lalu laporan oleh agent |
| `/guard on\|off\|reset` | Pengaman bagian sensitif |
| `/evolve <fitur>` | Minta Qilux membuat plugin baru |
| `/plugin` | Kelola plugin (`/plugin reload`) |
| `/doctor` | Periksa semua komponen |
| `/rollback`, `/undo` | Pulihkan inti Qilux / batalkan edit file terakhir |
| `/anim`, `/logo`, `/hidup` | Animasi berpikir, animasi logo, mode maskot hidup |
| `/host` | Kelola server SSH terdaftar |
| `/image`, `/view` | Lampirkan dan lihat gambar |
| `/cd`, `/ls` | Pindah dan lihat folder kerja |
| `/auto`, `/think` | Mode otomatis, tampilan proses berpikir |
| `/save`, `/load`, `/sessions`, `/clear` | Kelola sesi |
| `/config`, `/version`, `/exit` | Utilitas |
| `!perintah` | Jalankan shell langsung |
| `/sh` | Shell interaktif penuh |
| `@file.png` | Lampirkan gambar atau file di pesan |

---

## Mode

Ganti dengan `/mode`.

- **dev**: coding dan manajemen file.
- **ops**: administrasi server (instalasi, konfigurasi, monitoring).
- **cyber**: audit dan hardening **defensif** untuk server milik sendiri. Tidak ada exploit, brute force, atau pemindaian target pihak lain.

Kedalaman berpikir diatur terpisah lewat `/deep`:

| Level | Perilaku |
|-------|----------|
| `normal` | Cepat. |
| `dalam` | Berpikir teliti dan verifikasi mandiri. |
| `ekstrem` | Rencana wajib, maksimal 45 langkah, verifikasi. |

---

## Batasan

- **Guard bukan sandbox.** Deteksi bagian sensitif berbasis pola teks. Perintah yang disamarkan bisa lolos. Awasi sendiri di server penting.
- Tanpa daemon, **Shield hanya aktif selama Qilux terbuka**. Untuk perlindungan yang tetap hidup, pasang `/shield daemon install` atau andalkan fail2ban dan ufw hasil `/shield harden`.
- Langkah **SSH key-only** bisa mengunci kamu di luar VPS. Uji login dengan key di sesi kedua sebelum menyetujui.
- Deteksi port scan butuh langkah **"Catat log firewall"** dari `/shield harden`.
- Deteksi brute force membaca `/var/log/auth.log`. Di sistem tanpa file itu, deteksi tidak berjalan.
- `.env.local` adalah **teks biasa** yang dilindungi izin file, bukan enkripsi. Token bot Telegram juga disimpan di sana.
- Model kecil (mis. varian `flash`) kadang kurang rapi memanggil tool.
- Dirancang untuk Ubuntu/Debian. Di Termux atau Android, tampilan dan notifikasi Telegram bisa dicoba, tapi fitur Shield tidak punya yang dipantau. Belum diuji luas di distro lain.

---

## Berkas konfigurasi

```text
~/.qilux/.env.local      API key tiap provider dan token Telegram (izin 600)
~/.qilux/config.json     pengaturan (tanpa key)
~/.qilux/hosts.json      server SSH terdaftar
~/.qilux/telegram.json   pengaturan Telegram (ID owner, level)
~/.qilux/plugins/        plugin buatanmu / hasil /evolve
~/.qilux/backup/         cadangan inti qilux.py
~/.qilux/bans.json       IP yang diblokir Shield
~/.qilux/shield.log      log peringatan Shield
```

**Jangan pernah meng-commit folder `~/.qilux/` atau file `.env*`.**

---

<div align="center">

<img src="https://arr-official.id/uploads/baB2PJ.png" alt="Qilux" width="96">

**QILUX AGENT** · dibuat oleh [Aqill](https://github.com/aqilramadhana) 

Lisensi [MIT](LICENSE)

</div>
