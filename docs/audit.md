# Audit Situs (Fase 0)

> Tanggal audit: 30 September 2026
> Dasar: `PLAN-konsultan-ai.md` Bagian 3 (Fase 0) dan Bagian 11.
> Sifat dokumen: hanya temuan. **Belum ada kode yang diubah.** Fase 1 menunggu keputusan pemilik (lihat Bagian 5).

---

## 1. Temuan utama: repo ini bukan situs yang dideskripsikan di rencana

Rencana menyebut panji.tech sebagai situs satu halaman, dwibahasa ID/EN, dengan form kontak yang memicu email, kemungkinan HTML statis + Tailwind, dan section pendidikan bertuliskan "Expected Dec 2025". **Tidak satu pun ciri itu ada di repo `panjiallatief/portofolio`.**

Saya juga memeriksa repo publik lain milik pemilik yang paling mirip, `panjiallatief/panjiallatief.github.io` (hanya baca, tidak diubah). Hasilnya juga tidak cocok.

| Ciri di rencana | `portofolio` (repo ini) | `panjiallatief.github.io` |
|---|---|---|
| Domain | Tidak ada info domain; port 8087 | `CNAME` = `panjiallatief.me` (bukan panji.tech) |
| Framework | Spring Boot 2.7 + Thymeleaf + Bootstrap 5 + jQuery | HTML statis + Tailwind (CDN) |
| Toggle ID/EN | Tidak ada, hanya bahasa Inggris | Tidak ada. Toggle yang ada adalah **dark/light mode** |
| Form kontak yang memicu email | Tidak ada. Hanya link `mailto:` di menu terminal | Tidak ada. Hanya link `mailto:` |
| Section pendidikan | Tidak ada | Ada. M.Kom tertulis **"Apr 2024"**, bukan "Expected Dec 2025" |
| Proyek B-Leave, IT Solution Platform, WhatsApp bot, RFID, POS | Tidak ada | Sebagian (AMSIS, One Basket Ingest); lainnya tidak dicek satu per satu |
| Commit terakhir | 13 Nov 2023 | 5 Mar 2026 |

Situs live https://panji.tech tidak bisa dibuka dari lingkungan kerja ini (diblokir oleh kebijakan jaringan), jadi saya tidak bisa membandingkan langsung.

**Kesimpulan:** kode sumber panji.tech kemungkinan ada di repo lain (privat, di akun/organisasi lain, atau belum di GitHub). Fase 1 tidak bisa dikerjakan dengan aman sebelum repo yang benar diketahui, karena aturan Bagian 11 nomor 3 meminta mempertahankan mekanisme toggle ID/EN yang sudah ada, dan mekanisme itu tidak ditemukan.

---

## 2. Peta repo `portofolio`

```
portofolio/
├── pom.xml                         Spring Boot 2.7.17, Java 11
├── dockerfile                      openjdk:11, menyalin target/*.jar
├── mvnw, mvnw.cmd, .mvn/           Maven wrapper
├── .DS_Store                       file macOS ikut ter-commit
└── src/
    ├── main/java/com/portofolio/
    │   ├── PortofolioApplication.java
    │   └── maincontroller.java     GET "/" -> template "index"
    ├── main/resources/
    │   ├── application.yml         profil "panji", port 8087
    │   ├── templates/index.html    halaman utama (HTML + JS inline, ~330 baris)
    │   ├── templates/error.html    halaman error
    │   └── static/
    │       ├── css/styles.css      kartu, animasi latar, tampilan terminal
    │       └── images/IMG_7476.JPG foto profil (4,2 MB)
    └── test/.../PortofolioApplicationTests.java   hanya tes contextLoads
```

### Cara kerja halaman

- Satu kartu: foto, sapaan "Hi, I'm Panji Maulana Al Latief", deskripsi singkat, ikon HTML/JS/CSS, tombol LinkedIn, dan tombol "See My Project List".
- Tombol itu mengganti isi kartu dengan "terminal" beranimasi ketik. Menu: `[1] My Works`, `[2] Contact me`, `[B] Back`, bisa diklik atau lewat keyboard.
- Konten menu (daftar proyek dan kontak) ditulis langsung sebagai string HTML di objek `menus` dalam JavaScript (`index.html:155-171`).

### Bahasa

- Tidak ada mekanisme i18n. `<html lang="en">`, semua teks bahasa Inggris, dan hardcoded di HTML/JS.
- Thymeleaf dipakai hanya untuk render template. Fitur `messages.properties` / `#{...}` tidak dipakai.

### Kontak

- Hanya `mailto:panji.maulana.al@gmail.com` di menu terminal "[2] Contact me" (`index.html:158`).
- Tidak ada form, endpoint backend, atau layanan pihak ketiga (Formspree, EmailJS, dan sejenisnya).

### Deploy

- `dockerfile` mengharapkan JAR sudah dibangun di luar (`target/*.jar`), lalu menjalankannya dengan `openjdk:11`.
- Tidak ada `docker-compose`, reverse proxy, atau CI di repo.

---

## 3. Temuan yang terkait guardrail (Bagian 2 rencana)

Ini perlu perhatian pemilik **terlepas dari repo mana yang dipakai**, karena situs ini publik.

1. **Detail pemberi kerja di materi publik** (Prinsip 3). Teks hero menyebut "Currently work at B-Universe (BTV)" (`index.html:34`). Daftar proyek menyebut sistem internal BTV: AMS, vCard karyawan BTV, Daily Reporting IT BTV, One Basket, Project Management, Information Systems studio, Schedule Information (`index.html:159-170`).
2. **Link ke host internal pemberi kerja.** Item vCard mengarah ke `http://onebasket.b-universe.id:8110/Person/panji.maulana` (`index.html:163`), yaitu URL di domain pemberi kerja, lewat HTTP biasa dan port non-standar. Perlu dicek apakah boleh dipublikasikan.
3. **Link demo AMS** ke `ams.panjimaulana.my.id`, disebut "Asset Management System for BTV". Perlu dipastikan tidak memuat data internal.
4. `error.html` memuat footer "Team IT APPLICATION 2023" dan ikon `BTV.ico`, tampaknya disalin dari proyek kantor.

Rekomendasi: tinjau poin 1-4 bersama klausul kontrak kerja (checklist Bagian 2) sebelum situs dipakai untuk menawarkan jasa konsultasi.

---

## 4. Temuan teknis (untuk dicatat, **bukan** untuk di-refactor sekarang)

| # | Temuan | Lokasi | Dampak |
|---|---|---|---|
| T1 | Tombol LinkedIn memakai `<button href=...>`. Atribut `href` tidak berlaku di `<button>`, jadi tombol **tidak membuka LinkedIn** | `index.html:46` | Link kontak utama tidak berfungsi |
| T2 | `error.html` merujuk `images/bg404.png` dan `images/BTV.ico` yang tidak ada di repo | `error.html:8,18` | Halaman error menampilkan gambar rusak |
| T3 | Foto profil 4,2 MB tanpa kompresi | `static/images/IMG_7476.JPG` | Halaman lambat di mobile |
| T4 | Spring Boot 2.7 dan Java 11 sudah lewat masa dukungan komunitas; image `openjdk:11` sudah deprecated di Docker Hub | `pom.xml`, `dockerfile` | Tidak ada patch keamanan |
| T5 | Dependensi tidak terpakai: `spring-data-commons`, `javax.persistence` | `pom.xml` | Kecil |
| T6 | Tag `<script>` diletakkan setelah `</html>` | `index.html:86-333` | HTML tidak valid, meski browser tetap menjalankannya |
| T7 | Tidak ada meta SEO/deskripsi, Open Graph, atau favicon di halaman utama | `index.html:4-19` | Tampilan buruk saat link dibagikan |
| T8 | Salah ketik dan karakter liar: "œ" setelah Tata Cargo, "Bussiness", "An Platform", "Who is glizzy" di komentar | `index.html:162-166,198` | Kesan kurang rapi |
| T9 | `.DS_Store` ter-commit di beberapa folder, tidak ada di `.gitignore` | root, `src/` | Kecil |
| T10 | Nama kelas `maincontroller` tidak mengikuti konvensi Java (PascalCase) | `maincontroller.java` | Kecil |
| T11 | `multipart max-file-size: -1` dan `max-http-header-size: 10MB` tidak dibutuhkan situs statis | `application.yml` | Permukaan serangan lebih lebar dari perlu |
| T12 | Teks menyebut keahlian utama HTML/CSS/JS. Belum menyebut Go, Spring Boot, cloud, atau AI seperti di rencana | `index.html:33-39` | Positioning belum sesuai |

---

## 5. Keputusan yang dibutuhkan dari pemilik

1. **Di mana kode sumber panji.tech?**
   - Jika ada di repo lain: sebutkan nama repo (dan tambahkan ke sesi). Audit akan diulang di repo itu.
   - Jika panji.tech belum ada / ingin dibangun ulang: pilih basisnya.
     - (a) Repo `portofolio` ini (Spring Boot). Kelebihan: backend sudah ada untuk form kontak nanti. Kekurangan: stack usang, harus membangun i18n dari nol.
     - (b) Repo `panjiallatief.github.io` (HTML statis + Tailwind). Lebih dekat dengan deskripsi di rencana, konten CV lebih baru. Kekurangan: belum ada ID/EN dan form kontak; GitHub Pages tidak punya backend.
     - (c) Repo baru.
2. **Tanggal lulus M.Kom.** Rencana menyebut "lulus Januari 2025", sedangkan situs CV `panjiallatief.github.io` menulis "Apr 2024". Mana yang benar? (Bisa jadi tanggal sidang vs wisuda.)
3. **Penyebutan pemberi kerja dan proyek BTV** (Bagian 3 dokumen ini): dihapus, dianonimkan, atau dipertahankan setelah ada izin?

---

## 6. Ringkasan Fase 0

- **Selesai:** pemetaan struktur, framework, mekanisme bahasa, mekanisme kontak, deploy, dan temuan guardrail untuk repo `portofolio`; pembandingan dengan `panjiallatief.github.io`.
- **Tertunda:** audit repo panji.tech yang sebenarnya (belum ditemukan); pengecekan situs live (tidak bisa diakses dari lingkungan kerja).
- **Butuh keputusan pemilik:** tiga poin di Bagian 5. Fase 1 belum dimulai sesuai aturan Bagian 11 nomor 1.
