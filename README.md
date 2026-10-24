# bikun 🚌

Bikun is an AI agent for Universitas Indonesia students. It runs on Claude and reads EMAS3 (UI's Moodle) through your own login, twice a day.

Mornings, you get your deadlines and anything new on EMAS. Evenings, it tells you what's due tomorrow. On the right days it also writes a study guide: a short preview the night before a lecture, or a practice-heavy review once a kuis's material is done. Quiz dates come from each course's SAP, because most in-class quizzes never show up on the EMAS calendar.

It's unofficial and read-only, and everything it makes stays in your own Google Drive. The rest of this README is in Bahasa Indonesia.
---

## Apa yang dikerjakan Bikun

| Kapan | Apa |
|---|---|
| **Tiap pagi** (default 05:40) | *Morning brief*: deadline 7 hari ke depan (✓ sudah / ✗ belum dikumpul), materi baru di EMAS, dan maksimal 5 fokus hari ini |
| **Tiap malam** (default 18:42) | *H-1 reminder*: semua yang due besok dan belum dikumpul |
| **Malam sebelum kuliah** | *Preview diktat* (3–6 halaman, intuisi dulu) untuk matkul yang kamu pilih |
| **Hari bab terakhir kuis selesai** | *Review diktat*: ringkasan, arsip kuis tahun lalu (kalau ada), 20+ latihan, simulasi kuis |

Semua diktat masuk ke folder `Bikun/<matkul>` di Google Drive-mu. Jadwal kuis diambil dari **SAP/RPS di EMAS**, karena kuis di kelas biasanya tidak muncul di kalender EMAS.

## Yang kamu butuhkan

- **Claude** (paket berbayar) dengan **aplikasi desktop** terpasang di laptopmu
- **Google Drive** terhubung ke Claude (Settings → Connectors)
- Laptop **menyala** dan aplikasi Claude desktop **terbuka** saat jam brief dan run malam
- Akun EMAS3 aktif

## Pasang

1. Download `bikun.plugin` dari halaman [Releases](../../releases).
2. Tambahkan ke Claude sebagai plugin.
3. Buka chat baru di **aplikasi Claude desktop**, lalu ketik:

```
setup bikun
```

Bikun akan:
1. minta kamu **login EMAS sendiri** di browser Claude (SSO UI)
2. menampilkan matkul semester ini, lalu kamu pilih
3. membaca SAP/RPS tiap matkul dan membuat **sheet jadwal** di Drive-mu (cek tanggalnya, SAP kadang salah)
4. menanyakan **drive arsip soal** (lihat di bawah)
5. menanyakan jam brief dan matkul mana yang dapat Preview
6. membuat dua *scheduled task*: morning brief dan evening run

Setup memakan waktu sekitar 10–15 menit.

## Punya drive arsip soal? (bank soal himpunan, dll.)

Banyak himpunan punya Google Drive berisi kuis/UTS/UAS tahun-tahun sebelumnya. Kalau kamu punya akses ke salah satunya, ketik:

```
Bikun, tambahkan arsip: https://drive.google.com/drive/folders/<id-folder>
```

Bikun memetakan foldernya ke matkul-matkulmu, lalu memakai arsipnya untuk Review diktat dan diktat Ultimate. Kamu bisa menambah lebih dari satu arsip.

Pastikan:
- Folder itu bisa dibuka dengan akun Google **yang terhubung ke Claude**. Kalau dibagikan ke akun lain (misal akun UI), buka link-nya dengan akun yang terhubung, lalu klik **Add shortcut to Drive**.
- Arsip itu milik pembuatnya. Pakai untuk belajar sendiri, dan jangan disebar ulang.

## Perintah sehari-hari

Ketik biasa saja di chat Claude:

| Ketik | Hasil |
|---|---|
| `deadline minggu ini apa?` | Daftar deadline dari EMAS + SAP |
| `besok ada apa?` | H-1 versi instan |
| `buat diktat review kuis 2 PDB` | Review diktat sekarang juga |
| `preview materi kuliah besok Statmat` | Preview diktat sekarang juga |
| `buat diktat ultimate UTS Analisis` | Diktat UTS/UAS lengkap |
| `ambil slide pekan 9 Statmat` | Bikun membaca slide-nya dari EMAS |
| `kuis 3 Analisis pindah ke 7 Desember` | Jadwal dan Review diktat ikut pindah |
| `ubah jam brief ke 06:15` | Mengubah jadwal run |
| `stop preview Kimia` / `preview untuk Progmat` | Mengatur Preview per matkul |
| `matikan bikun` | Run pagi/malam dimatikan (bisa dinyalakan lagi) |

## Batasan

- **Laptop harus menyala**, karena Bikun membaca EMAS lewat browser di aplikasi Claude desktop. Kalau laptop mati, run hari itu hanya mengirim pengingat dari sheet jadwal.
- **EMAS kadang logout.** Bikun akan bilang "EMAS logout. Login dulu…". Kamu login sendiri, Bikun tidak pernah mengetik password.
- **Kuis tanpa tanggal di SAP** (kuis dadakan) tidak dapat Review diktat otomatis. Minta manual saja.
- **Slide hasil scan** tidak bisa dibaca teksnya. Bikun memakai buku teks dari SAP sebagai gantinya.
- Diktat dibuat AI. Jawaban kuantitatif dicek dengan kode, tapi tetap baca dengan kritis.

## Privasi

- Bikun tidak punya server. Semua data (config, jadwal, diktat) ada di **Google Drive-mu sendiri**, di folder `Bikun/`.
- Bikun hanya membaca akun EMAS-mu sendiri, dan tidak pernah membaca data mahasiswa lain.
- Pemakaiannya ringan: 2 run per hari, hanya halaman yang diperlukan.

## Isi plugin

| Skill | Fungsi |
|---|---|
| `bikun:setup` | Setup awal dan perubahan (matkul, jam, arsip, semester baru) |
| `bikun:emas` | Membaca EMAS3 lewat API Moodle di sesi login-mu, read-only |
| `bikun:arsip` | Mencari soal lama di drive arsip yang kamu daftarkan |
| `bikun:diktat` | Diktat Preview, Review, dan Ultimate (LaTeX → PDF) |
| `bikun:brief` | Morning brief dan evening run |

## Lisensi

MIT. Lihat [LICENSE](LICENSE). Dibuat oleh [@gianfarren](https://github.com/gianfarren), mahasiswa Matematika UI.
