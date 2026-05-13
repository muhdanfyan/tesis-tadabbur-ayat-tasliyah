# 🎯 TAHAPAN DETAIL PEMBUATAN TESIS — MSc PIC IOU
## Tadabbur Ayat Tasliyah sebagai Intervensi Psikospiritual

**Tools Stack yang Digunakan:**
| Tool | Fungsi | Resource |
|------|--------|----------|
| `rclone` | Akses Google Drive (baca/tulis) | ✅ Terinstall |
| `yt-dlp` | Download video/audio YouTube | ✅ Perlu install |
| `python3 + requests/bs4` | Web scraping link | ✅ Bawaan |
| `sub-agent (Gemini)` | Crawling, searching, kompilasi | ✅ Unlimited |
| `exec background` | Download, convert, extract | ✅ VPS |
| `pandoc` | Convert docx/md/pdf | ✅ Perlu install |
| `pdftotext` (poppler) | Extract teks dari PDF | ✅ Perlu install |
| `ripgrep` (rg) | Search cepat dalam file | ✅ Perlu install |
| `zotero` CLI / manual | Manajemen referensi | 🔄 Opsional |
| `web_fetch` tool | Ambil konten web | ✅ Bawaan |
| `web_search` tool | Google Search | ✅ Bawaan |

**Catatan Resource:**
- VPS: 103.150.196.183 (pondokinformatika) — CPU, RAM, storage
- File cache sementara: `/tmp/tesis-cache/`
- Cleanup otomatis setelah selesai per tahap

---

## FASE 0: SETUP LINGKUNGAN (Estimasi: 1-2 jam)

### 0.1 — Install Tools Tambahan
```bash
# yt-dlp — download video YouTube
pip3 install yt-dlp

# pandoc — convert docx ↔ md ↔ pdf
sudo apt install pandoc -y

# poppler-utils — pdftotext
sudo apt install poppler-utils -y

# ripgrep — search cepat dalam file
sudo apt install ripgrep -y

# Buat cache directory
mkdir -p /tmp/tesis-cache/{video,pdf,teks,draft}
```

### 0.2 — Verifikasi Koneksi GDrive
```bash
# Test akses rclone ke semua folder yang diperlukan
rclone lsd gdrive:"Kuliah_Psikologi_Islam_IOU/Tesis_Ad-Dhuha_Al-Insyirah/"
rclone lsd gdrive:"Kuliah_Psikologi_Islam_IOU/PMDS 901 Technial Guide/"
```

**✅ Output Terverifikasi:**
- 7 subfolder tesis ✅
- PMDS 901 ✅ (guidelines + contoh tesis)
- Folder kuliah ✅

---

## FASE 1: STUDI PANDUAN & CONTOH TESIS IOU (Estimasi: 1-2 hari)

### 1.1 — Copy Guidelines IOU ke Lokal
**Action:** Copy file panduan tesis IOU ke workspace untuk analisis.
```bash
rclone copyto gdrive:"Kuliah_Psikologi_Islam_IOU/PMDS 901 Technial Guide/Dokumen Rujukan Thesis/Masters Thesis guidelines-Final_KATA PENGANTAR REVISED.pdf" /tmp/tesis-cache/pdf/guidelines-iou.pdf
```
**Output:** `guidelines-iou.pdf` — dianalisis strukturnya

### 1.2 — Analisis Guidelines IOU
Baca dokumen panduan, ekstrak:
- Format halaman (margin, font, spacing)
- Struktur BAB yang disyaratkan
- Aturan sitasi (APA 7th?)
- Jumlah halaman minimal
- Format abstrak
- Format daftar pustaka
- Persyaratan ethical clearance

**Output:** `aturan-format-iou.md`

### 1.3 — Analisis Ethical Clearance
```bash
rclone copyto "gdrive:Kuliah_Psikologi_Islam_IOU/PMDS 901 Technial Guide/Dokumen Rujukan Thesis/Ethical Clearance Application.pdf" /tmp/tesis-cache/pdf/ethical-clearance-iou.pdf
```
Baca & catat persyaratan etik yang harus dipenuhi.

### 1.4 — Analisis 3 Contoh Tesis MSc PIC
```bash
# Copy semua contoh tesis
rclone copy "gdrive:Kuliah_Psikologi_Islam_IOU/PMDS 901 Technial Guide/Contoh Thesis Psikologi MScPIC/" /tmp/tesis-cache/pdf/ --progress
```
- **Ferzya Farhan** (3MB PDF) — baca struktur & gaya
- **Aisyah Amini** (8MB PDF) — Quran Journaling, metodologi
- **Alifa Ashaira** (DOCX) — format Word asli

**Yang Dianalisis per Tesis:**
1. Judul & abstrak
2. BAB I — bagaimana framing masalah?
3. BAB II — bagaimana struktur tinjauan pustaka?
4. BAB III — metodologi apa yang dipakai?
5. BAB IV — bagaimana penyajian hasil?
6. BAB V — bagaimana diskusi & kesimpulan?
7. Daftar pustaka — jumlah & jenis referensi
8. Lampiran

**Output:**
- `analisis-contoh-tesis.md` — tabel perbandingan 3 tesis
- Template struktur tesis yang akan diikuti

### 1.5 — Ekstrak Logbook
```bash
rclone copyto "gdrive:Kuliah_Psikologi_Islam_IOU/PMDS 901 Technial Guide/Dokumen Rujukan Thesis/PMDS901 Logbook_Bahasa_Lembar Bimbingan.docx" /tmp/tesis-cache/pdf/logbook.docx
```
Baca format logbook, siapkan template untuk diisi.

---

## FASE 2: INVENTARISASI & DOWNLOAD REFERENSI (Estimasi: 2-3 hari)

### 2.1 — Scan Semua Folder Kuliah untuk Referensi
```bash
# Buat daftar lengkap semua file di Kuliah_Psikologi_Islam_IOU
rclone lsf gdrive:"Kuliah_Psikologi_Islam_IOU/kuliah" -R --files-only > /tmp/tesis-cache/daftar-file-kuliah.txt
# Kategorikan otomatis
grep -i "jurnal\|artikel\|paper\|research" /tmp/tesis-cache/daftar-file-kuliah.txt > /tmp/tesis-cache/daftar-jurnal.txt
grep -i "modul\|m01\|m02\|module" /tmp/tesis-cache/daftar-file-kuliah.txt > /tmp/tesis-cache/daftar-modul.txt
grep -i "tesis\|thesis\|proposal" /tmp/tesis-cache/daftar-file-kuliah.txt > /tmp/tesis-cache/daftar-tesis-proposal.txt
```

### 2.2 — Copy Modul Relevan ke Lokal
Prioritaskan modul yang langsung relevan:
```bash
# PIPP 702 (Psikoterapi Islami) — 15 modul
rclone copy "gdrive:Kuliah_Psikologi_Islam_IOU/kuliah/Psikologi_dan_Psikoterapi_Islami_PIPP702/Psikologi dan Psikoterapi Islami (PIPP 702)/pdf/" /tmp/tesis-cache/pdf/modul-pipp702/ --progress

# Psikologi Islam — modul terpilih
rclone copy "gdrive:Kuliah_Psikologi_Islam_IOU/kuliah/Psikologi_Islam/Psikologi Islam/" /tmp/tesis-cache/pdf/modul-psy101/ --progress

# Jurnal penelitian
rclone copy "gdrive:Kuliah_Psikologi_Islam_IOU/kuliah/metode penelitian/Jurnal/" /tmp/tesis-cache/pdf/jurnal-kuliah/ --progress

# Jurnal tadabbur
rclone copy "gdrive:Kuliah_Psikologi_Islam_IOU/kuliah/Proposal_Kuliah/research/" /tmp/tesis-cache/pdf/jurnal-tadabbur/ --progress
```

### 2.3 — Ekstrak Teks dari Semua PDF
```bash
# Batch extract
find /tmp/tesis-cache/pdf -name "*.pdf" | while read f; do
  outdir="/tmp/tesis-cache/teks/$(dirname "${f#/tmp/tesis-cache/pdf/}")"
  mkdir -p "$outdir"
  pdftotext "$f" "$outdir/$(basename "${f%.pdf}.txt")"
done
```

### 2.4 — Search Cepat dengan Ripgrep
```bash
# Cari kata kunci penting di semua teks
rg -i "tadabbur\|tasliyah\|adh-dhuha\|al-insyirah\|cognitive restructuring\|restrukturisasi kognitif\|cbt islam" /tmp/tesis-cache/teks/ > /tmp/tesis-cache/hasil-search-utama.txt

# Cari metodologi
rg -i "quasi.experimental\|rct\|pls-sem\|mediation\|intervensi\|protokol" /tmp/tesis-cache/teks/ > /tmp/tesis-cache/hasil-search-metodologi.txt

# Cari nama tokoh
rg -i "keshavarzi\|rothman\|hamdan\|badri\|malik badri\|langgulung\|hude" /tmp/tesis-cache/teks/ > /tmp/tesis-cache/hasil-search-tokoh.txt
```

### 2.5 — Verifikasi & Lengkapi Daftar Pustaka
Bandingkan daftar pustaka target (48 referensi) dengan yang sudah ada di folder. Buat status:
- ✅ **SUDAH ADA** — file sudah di folder
- ⚠️ **PERLU DICARI** — belum ada, perlu didownload dari web
- ❌ **TIDAK DITEMUKAN** — mungkin tidak tersedia gratis

**Sumber untuk mencari PDF:**
1. **Google Scholar** — `web_search` + download
2. **Sci-Hub** — untuk jurnal berbayar
3. **ResearchGate** — profil akademisi
4. **Academia.edu** — paper sharing
5. **DOI langsung** — `https://doi.org/...`
6. **IIIT (International Institute of Islamic Thought)** — buku Badri, Al-Faruqi
7. **Internet Archive** — buku klasik

---

## FASE 3: PENGAYAAN TAFSIR (Estimasi: 3-5 hari)

### 3.1 — Kumpulkan Link Video Ulama
Gunakan sub-agent Gemini + web_search untuk mencari:
```bash
# Format pencarian per ulama
web_search "tafsir surah adh dhuha syaikh as-sa'di"
web_search "tafsir surat al insyirah nouman ali khan"
web_search "تفسير سورة الضحى الشيخ عبد العزيز الطريفي"
web_search "تفسير سورة الشرح للشيخ ناصر العمر"
...dst untuk semua ulama di daftar
```

**Format Output per Video:**
```markdown
| Ulama | Judul | Link | Durasi | Bahasa | Poin Utama |
|-------|-------|------|--------|--------|------------|
| As-Sa'di | Tafsir Ad-Dhuha | youtube.com/... | 15:20 | Arab | ... |
```

### 3.2 — Kumpulkan Link Tafsir Tertulis
Cari link teks tafsir untuk kitab-kitab yang sulit didapat PDF-nya:
```bash
web_search "tafsir al razi mafatih al ghaib surah al insyirah online"
web_search "tafsir al baghawi surah adh dhuha"
web_search "ibnu katsir surah adh dhuha tafsir web"
web_search "tafsir al mishbah quraish shihab surah adh dhuha pdf"
```

### 3.3 — Download Tafsir yang Perlu (via Sub-agent)
Prioritas download PDF tafsir yang paling penting:
1. **As-Sa'di** — Taisir Al-Karim Ar-Rahman (paling aplikatif)
2. **Asy-Sya'rawi** — Tafsir Asy-Sya'rawi (psikologis-bahasa)
3. **Hamka** — Tafsir Al-Azhar (konteks Nusantara)
4. **Quraish Shihab** — Tafsir Al-Mishbah (akademis)
5. **Sayyid Qutb** — Fi Zhilal Al-Quran (harakah)
6. **Al-Razi** — Mafatih Al-Ghaib (filosofis-psikologis)

### 3.4 — Kompilasi Tafsir Lengkap
Buat dokumen komprehensif yang berisi untuk setiap ayat:
1. **Teks Arab & Terjemahan**
2. **Asbabun Nuzul** (riwayat Shahih)
3. **Tafsir Klasik** (Ath-Thabari, Ibn Katsir, Al-Razi, Zamakhsyari)
4. **Tafsir Modern** (As-Sa'di, Asy-Sya'rawi, Sayyid Qutb)
5. **Tafsir Nusantara** (Hamka, Quraish Shihab)
6. **Analisis Balaghah** (gaya bahasa, iltifat, qasam)
7. **Respon Salaf & Khalaf**
8. **Efek Psikologis** (analisis per ayat)

**Output:** `03_Tafsir_Mendalam/tafsir-lengkap-ad-dhuha.md` + `tafsir-lengkap-al-insyirah.md`

---

## FASE 4: PENGAYAAN PSIKOLOGI (Estimasi: 3-5 hari)

### 4.1 — Download Referensi Psikologi
Prioritas download:
1. **Frankl** — Man's Search for Meaning (PDF bebas)
2. **Seligman** — Learned Optimism / Flourish
3. **Beck** — Cognitive Therapy (buku klasik)
4. **Ellis** — REBT (Rational Emotive Behavior Therapy)
5. **Keshavarzi & Haque** — Islamic Psychology and Integration
6. **Rothman** — Islamic Psychology (buku baru)
7. **Badri** — Contemplation (sudah ada? cek!)
8. **Hamdan** (2008) — Cognitive Restructuring: Islamic Perspective (jurnal)

**Sumber Download:**
- Google Scholar → PDF langsung atau DOI
- `https://sci-hub.se/` + DOI
- `https://libgen.is/` — untuk buku
- ResearchGate → profile authors

### 4.2 — Ekstrak Konsep Kunci dari Setiap Referensi
Baca & ekstrak untuk setiap referensi:
```markdown
## [Judul Referensi]
- **Penulis:** [nama]
- **Tahun:** [tahun]
- **Konsep Utama:** [1-2 kalimat]
- **Relevansi dengan Tesis:**
  - Bagaimana teori ini berhubungan dengan tadabbur?
  - Mekanisme apa yang dijelaskan?
  - Apakah sudah ada penelitian serupa?
  - Apa yang bisa diadaptasi?
- **Kutipan Kunci:** "..." (1-2 kutipan penting)
- **Link/DOI:** [link]
```

### 4.3 — Buat Peta Korelasi Ayat-Psikologi (3 Saringan)
Untuk setiap korelasi, terapkan 3 saringan:

```markdown
## Korelasi: [Ayat] ↔ [Teori Psikologi]

### ✅ Saringan 1: Otentisitas Tafsir
- Apakah makna ini sudah disebut ulama klasik?
- Siapa saja? Ath-Thabari? Ibn Katsir? Al-Razi?
- **Verdict:** LULUS / TIDAK

### ✅ Saringan 2: Konsistensi Bahasa
- Kata kunci Arab: [kata]
- Akar kata: [3 huruf]
- Makna dalam kamus: [definisi]
- Apakah mendukung makna psikologi?
- **Verdict:** LULUS / TIDAK

### ✅ Saringan 3: Independensi Psikologi
- Apakah teori psikologi ini ditemukan independen?
- Tahun ditemukan: [tahun]
- Populasi riset: [siapa yang diteliti]
- Apakah ada bias konfirmasi?
- **Verdict:** LULUS / TIDAK

### 📊 Kesimpulan: [LAYAK/TIDAK LAYAK masuk tesis]
```

### 4.4 — Finalisasi Daftar Korelasi yang Lolos
Hanya korelasi yang lolos 3 saringan yang masuk dokumen final.

---

## FASE 5: PENGEMBANGAN METODOLOGI (Estimasi: 2-3 hari)

### 5.1 — Review Modul Metodologi dari Kuliah
Baca & ekstrak prinsip-prinsip dari:
- Modul PPTA 702 — Research Methods
- Tugas-tugas riset Bang Dadan
- Jurnal metodologi di folder

### 5.2 — Desain Penelitian Final
```markdown
## Desain Penelitian

### Jenis
- [ ] Eksperimen murni (RCT)
- [ ] Quasi-eksperimen (nonequivalent control group)
- [ ] Pre-experimental (one group pre-post)

### Variabel
- **IV:** Tadabbur Ayat Tasliyah (8 minggu, 16 sesi)
- **DV1:** Positive Thinking (LOT-R)
- **DV2:** Kecemasan (BAI)
- **Mediator:** Restrukturisasi Kognitif (CRS)

### Hipotesis (Final)
**H1:** ...
**H2:** ...
**H3:** ...
**H4:** ...

### Sampel
- Populasi: santri Pondok Informatika
- Jumlah: [hasil power analysis]
- Teknik sampling: [purposive/random]
- Grup: [eksperimen vs kontrol]

### Instrumen
1. **CRS** — Cognitive Restructuring Scale
   - Adaptasi dari Hamdan (2008)
   - Jumlah item: [x]
   - Uji validasi: pilot study
2. **LOT-R** — Life Orientation Test (Scheier et al., 1994)
3. **BAI** — Beck Anxiety Inventory (Beck & Steer, 1993)

### Analisis
- Uji asumsi (normalitas, homogenitas)
- Uji hipotesis: paired t-test / ANCOVA
- Uji mediasi: PROCESS macro (Hayes) / PLS-SEM
```

### 5.3 — Protokol Intervensi Detail
```markdown
## Protokol Tadabbur 8 Minggu

### Minggu 1: Orientasi
- Sesi 1: Pre-test (LOT-R, BAI, CRS) + pengenalan program
- Sesi 2: Konsep tadabbur + pengenalan ayat tasliyah

### Minggu 2-3: QS Ad-Dhuha (Ayat 1-5)
- Sesi 3: Tilawah + tafsir ayat 1-3 (wadhduha, wallayli, ma wadda'aka)
- Sesi 4: Refleksi terbimbing — "Allah tidak meninggalkanmu"
- Sesi 5: Tafsir ayat 4-5 (walal-aakhiratu, walasawfa yu'thika)
- Sesi 6: Journaling — harapan setelah masa sulit

### Minggu 4-5: QS Ad-Dhuha (Ayat 6-11)
- Sesi 7: Tafsir ayat 6-8 (yatim, 'a'il, dhaallan)
- Sesi 8: Refleksi — nikmat masa lalu sebagai kekuatan
- Sesi 9: Tafsir ayat 9-11 (amal shalih)
- Sesi 10: Journaling — syukur & perilaku prososial

### Minggu 6-7: QS Al-Insyirah
- Sesi 11: Tafsir ayat 1-4 (alam nasyrah, wizrak)
- Sesi 12: Refleksi — kelapangan dada & pelepasan beban
- Sesi 13: Tafsir ayat 5-6 (ma'a al-'usri yusra)
- Sesi 14: Refleksi — harapan di tengah kesulitan

### Minggu 8: Penutup
- Sesi 15: Tafsir ayat 7-8 (fanshab, farghab) + review
- Sesi 16: Post-test (LOT-R, BAI, CRS) + sesi refleksi akhir
```

---

## FASE 6: PENULISAN DRAFT (Estimasi: 7-10 hari)

### 6.1 — BAB I: Pendahuluan (Revisi dari Proposal)
Ambil dari proposal existing, perkaya dengan:
- Temuan baru dari analisis tafsir
- Data yang lebih spesifik dari Pondok Informatika
- Gap yang lebih tajam setelah review literatur

### 6.2 — BAB II: Tinjauan Pustaka (Diperkaya)
Gabungkan dari Fase 3 (tafsir) + Fase 4 (psikologi):
- 2.1 Landasan Qur'ani: ayat tasliyah (dari tafsir lengkap)
- 2.2 Landasan Psikologis: CBT, Positive Psychology, Psikologi Islam
- 2.3 Konsep Utama & Definisi Operasional
- 2.4 Penelitian Terdahulu (review 15+ penelitian)
- 2.5 Kerangka Konseptual (model mediasi)
- 2.6 Hipotesis

### 6.3 — BAB III: Metodologi (Diperkuat)
Dari Fase 5:
- Desain, sampel, instrumen, prosedur, analisis
- Protokol intervensi detail
- Ethical clearance

### 6.4 — Literatur Review Paper (Bonus)
Dari review literatur di Fase 2-4, buat artikel:
"Tadabbur Ayat Tasliyah sebagai Mekanisme Restrukturisasi Kognitif: Sebuah Literature Review Terintegrasi"
→ Bisa untuk publikasi jurnal

---

## FASE 7: REVIEW & REVISI (Estimasi: 3-5 hari)

### 7.1 — Self-Review
Cek list:
- [ ] Format sesuai guidelines IOU
- [ ] Sitasi konsisten (APA 7th?)
- [ ] Daftar pustaka lengkap
- [ ] Tidak ada plagiarisme
- [ ] Bahasa akademis konsisten

### 7.2 — Review dengan Bang Dadan
- Presentasi hasil per BAB
- Diskusi revisi

### 7.3 — Finalisasi
- Proofreading final
- Export PDF
- Upload ke GDrive

---

## FASE 8: CLEANUP (Estimasi: 1 jam)

```bash
# Hapus cache lokal
rm -rf /tmp/tesis-cache/

# Hapus file sementara di workspace
rm -f /home/pondokinformatika/.openclaw/workspace/*.py
rm -f /home/pondokinformatika/.openclaw/workspace/*.html
rm -f /home/pondokinformatika/.openclaw/workspace/*.js

# Uninstall tools yang tidak perlu lagi (optional)
# pip3 uninstall yt-dlp -y
# sudo apt remove pandoc poppler-utils ripgrep -y
```

---

## 📊 RINGKASAN RESOURCE

| Resource | Estimasi | Keterangan |
|----------|----------|------------|
| Storage cache | ~500MB-2GB | File sementara di /tmp |
| Download data | ~200MB-1GB | PDF, video transkrip |
| Sub-agent calls | ~20-50 kali | Searching + kompilasi |
| Eksekusi bash | ~50-100 kali | Copy, extract, search |
| Waktu total | ~20-30 hari | Background bertahap |
| Cleanup akhir | ~100MB tersisa | Hanya file final di GDrive |

---

> **Status: DOKUMEN SIAP — Eksekusi dimulai**
> Versi 1.0 — 13 Mei 2026
