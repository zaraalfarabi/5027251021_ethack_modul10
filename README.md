# PRAKTIKUM ETHICAL HACKING MODUL 10

| Keterangan | Isi |
| :--- | :--- |
| Nama | Az Zahra Fiddien Al Farabi |
| NRP | 5027251021 |
| Kelas | B |

## Daftar Isi

1. [Analisis Exploit — Bab 10](#analisis-exploit--bab-10)
2. [Analisis Rantai CVE → Advisory → Exploit — Bab 10](#analisis-rantai-cve---advisory---exploit--bab-10)
---
# Analisis Exploit — Bab 10

## Info Umum

- **Service target:** Apache HTTP Server / OpenSSL (mod_ssl)
- **Path exploit (SearchSploit):** `unix/remote/21671.c`
- **EDB-ID:** 21671
- **Nama exploit:** Apache mod_ssl < 2.8.7 OpenSSL - OpenFuck.c Remote Buffer Overflow
- **Tanggal analisis:** 12 November 2026

### Hasil Enumerasi Target

Berdasarkan version detection menggunakan Nmap terhadap Metasploitable2:

- IP target: `192.168.56.102`
- Port HTTP: `80/tcp`
- Service: Apache httpd
- Versi terdeteksi: Apache httpd 2.2.8 (Ubuntu) DAV/2

<img width="1254" height="740" alt="image" src="https://github.com/user-attachments/assets/a4d49659-07b9-488e-9691-456481efca57" />

Catatan: Nmap belum mengonfirmasi versi OpenSSL atau mod_ssl yang terpasang pada target.


## Analisis Kode (Baca, Jangan Jalankan Jika Memory-Corruption)

- **Bahasa/tipe exploit:** C (OpenFuck.c), Remote Buffer Overflow.

- **Apa yang dilakukan kode ini:**

  Exploit ini memanfaatkan kerentanan buffer overflow pada beberapa versi OpenSSL yang berkaitan dengan proses penanganan client key saat negosiasi protokol SSLv2.

  Berdasarkan komentar pada source code, kerentanan memungkinkan malicious client berpotensi mengeksekusi arbitrary code dengan hak akses proses server yang rentan.

  Dari pembacaan source code, ditemukan penggunaan library OpenSSL, antara lain:
  - `openssl/ssl.h`
  - `openssl/rsa.h`
  - `openssl/x509.h`
  - `openssl/evp.h`

  Pada bagian kode lainnya terlihat manipulasi struktur heap memory, penggunaan fake malloc chunk, manipulasi session SSL, serta referensi terhadap shellcode address.

- **Kondisi yang dibutuhkan agar berhasil:**

  1. Target harus menggunakan komponen OpenSSL/mod_ssl yang memiliki kerentanan terkait.
  2. Target harus menggunakan konfigurasi SSLv2 yang sesuai dengan kondisi vulnerability.
  3. Versi dan lingkungan sistem target harus sesuai dengan salah satu target yang didukung exploit.
  4. Service harus dapat dijangkau melalui jaringan.

  Kesesuaian exploit dengan Metasploitable2 belum dapat dipastikan karena versi OpenSSL/mod_ssl belum dikonfirmasi.

  **Dokumentasi:**

  <img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/c328d4a9-332a-406d-be5b-816e13226fb3" />

  <img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/b55ecada-ca84-4ae2-90f1-e0cd62fc406f" />


- **Tanda-tanda kepercayaan:**

  Source code mencantumkan referensi SecurityFocus dan menggunakan library OpenSSL yang relevan dengan fungsi exploit.

  Namun, keberadaan kode yang melakukan manipulasi heap memory, penggunaan shellcode, serta mekanisme pembentukan shell merupakan tanda bahwa kode ini berisiko tinggi apabila dijalankan.

  Pada bagian source code yang diperiksa, tidak ditemukan bukti yang cukup untuk menyatakan adanya obfuscation atau koneksi ke domain asing yang tidak berkaitan dengan mekanisme exploit.

  Source code hanya digunakan untuk analisis dan tidak dijalankan maupun dikompilasi.

  **Dokumentasi Source Code:**

  <img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/bbe865f1-73ae-4ef1-afef-cd23191520fa" />


## Studi Kasus: Backdoor vsftpd 2.3.4

- **Banner yang dikonfirmasi:**

  Pengujian banner grab menggunakan netcat:

  ```text
  nc 192.168.56.102 21

  220 (vsFTPd 2.3.4)
  QUIT
  221 Goodbye.
  ```
  Hasil menunjukkan bahwa service FTP berjalan pada port 21 dan menampilkan banner vsFTPd 2.3.4.

- **Pemahaman soal cara kerja backdoor:**

  Berdasarkan dokumentasi NVD dan informasi modul Metasploit resmi, backdoor ditemukan pada arsip unduhan `vsftpd-2.3.4.tar.gz` yang didistribusikan antara 30 Juni 2011 dan 3 Juli 2011.

  Backdoor tersebut dapat membuka shell pada port TCP 6200.

  Kerentanan ini berbeda dengan buffer overflow karena penyebabnya adalah kode backdoor berbahaya yang sengaja ditanam dalam distribusi software, bukan kesalahan pengelolaan memori.

  Referensi:
  - NVD: CVE-2011-2523.
  - Rapid7: VSFTPD 2.3.4 Backdoor Command Execution.
  - Original security alert: ScaryBeasts Security Blog.

- **Hasil eksekusi modul Metasploit resmi:**

  Modul yang berhasil ditemukan dan dibuka:

  `exploit/unix/ftp/vsftpd_234_backdoor`

  Hasil pemeriksaan menggunakan `info` menunjukkan:
  - Name: VSFTPD 2.3.4 Backdoor Command Execution.
  - Platform: Unix, Linux.
  - Architecture: cmd.
  - Privileged: Yes.
  - Rank: Excellent.
  - Target: Linux/Unix Command.
  - Referensi: CVE-2011-2523.

  Konfigurasi target yang telah dimasukkan:
  - RHOSTS: `192.168.56.102`
  - RPORT: `21`
 
  **Dokumentasi:**

  <img width="1919" height="740" alt="image" src="https://github.com/user-attachments/assets/28a73694-78ca-4766-af73-d8980b433396" />

  <img width="1919" height="765" alt="image" src="https://github.com/user-attachments/assets/93f7e056-1606-4bc8-ba21-1d496d95ce69" />

  Pemeriksaan `show options` berhasil dilakukan. Namun, modul belum dijalankan menggunakan perintah `run`, sehingga keberhasilan eksploitasi dan pembentukan sesi belum dapat dikonfirmasi.

- **Refleksi:**

  Backdoor logika aplikasi lebih aman dipelajari secara konseptual karena mekanisme dan penyebab kerentanannya dapat dipahami melalui dokumentasi, banner service, serta informasi modul resmi tanpa perlu memanipulasi memori atau membuat exploit sendiri.

  Pada praktikum ini, pembelajaran dilakukan melalui identifikasi service, pembacaan dokumentasi, dan pemeriksaan modul Metasploit.

  Sebaliknya, exploit memory-corruption memiliki risiko lebih tinggi karena melibatkan manipulasi memori, shellcode, dan kemungkinan eksekusi kode yang dapat menyebabkan gangguan sistem.

## Kesimpulan

Praktikum ini memberikan pemahaman mengenai metodologi pencarian dan evaluasi exploit publik menggunakan SearchSploit.

Exploit OpenFuck EDB-ID 21671 dianalisis melalui pembacaan source code tanpa menjalankannya. Studi kasus vsftpd 2.3.4 dilakukan melalui banner grabbing dan pemeriksaan modul Metasploit resmi.

Hasil identifikasi menunjukkan bahwa service target menggunakan vsFTPd 2.3.4, tetapi keberadaan backdoor pada binary target dan keberhasilan eksploitasi belum terkonfirmasi.


# Analisis Rantai CVE -> Advisory -> Exploit — Bab 10

## Info Umum

- **CVE ID yang dipilih:** CVE-2011-2523
- **Service/versi terkait:** vsftpd 2.3.4
- **IP target:** 192.168.56.102
- **Port:** 21/tcp
- **Tanggal analisis:** 12 November 2026

## Dari NVD

- **Deskripsi teknis singkat:**

  Berdasarkan NVD, vsftpd 2.3.4 yang diunduh pada periode 30 Juni 2011 sampai 3 Juli 2011 mengandung backdoor yang dapat membuka shell pada port TCP 6200.

  Backdoor ini merupakan kode berbahaya yang ditanam dalam arsip distribusi software vsftpd 2.3.4.

- **Skor CVSS resmi:**

  CVSS Version 3.1

  Base Score: 9.8 (CRITICAL)

  Vector:

  `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H`

- **Breakdown metrik (AV/AC/PR/UI/Impact):**

  | Metrik | Nilai | Penjelasan |
  |---|---|---|
  | AV | Network | Kerentanan dapat diakses melalui jaringan. |
  | AC | Low | Tidak membutuhkan kondisi eksploitasi yang kompleks. |
  | PR | None | Tidak membutuhkan hak akses sebelumnya. |
  | UI | None | Tidak membutuhkan interaksi pengguna. |
  | S | Unchanged | Dampak berada pada security authority yang sama. |
  | C | High | Kerahasiaan informasi dapat terdampak tinggi. |
  | I | High | Integritas sistem dapat terdampak tinggi. |
  | A | High | Ketersediaan sistem dapat terdampak tinggi. |

  **Weakness Enumeration:**

  - CWE-78: Improper Neutralization of Special Elements used in an OS Command (OS Command Injection).

### Referensi Advisory

NVD mencantumkan beberapa referensi yang berkaitan dengan CVE ini:

1. Red Hat Security Advisory:
   https://access.redhat.com/security/cve/cve-2011-2523

2. Debian Security Tracker:
   https://security-tracker.debian.org/tracker/CVE-2011-2523

3. Packet Storm Security:
   https://packetstormsecurity.com/files/102745/VSFTPD-2.3.4-Backdoor-Command-Execution.html

## Dari Kode PoC (SearchSploit/Exploit-DB)

- **Ringkasan cara kerja exploit:**

  Berdasarkan hasil pencarian SearchSploit, ditemukan referensi exploit untuk vsftpd 2.3.4, termasuk:

  - EDB-ID 49757: Python exploit.
  - EDB-ID 17491: Metasploit module.

  Modul Metasploit resmi yang diperiksa:

  `exploit/unix/ftp/vsftpd_234_backdoor`

  Berdasarkan hasil perintah `info`, modul tersebut memanfaatkan backdoor berbahaya yang ditambahkan ke arsip distribusi vsftpd 2.3.4.

  Modul ini ditujukan untuk melakukan command execution pada sistem Unix/Linux yang terdampak.

  Informasi NVD menjelaskan bahwa backdoor dapat membuka shell pada port TCP 6200.

  Catatan: source code PoC EDB-ID 49757 tidak diperiksa secara langsung. Analisis mekanisme pada bagian ini didasarkan pada dokumentasi NVD dan deskripsi modul Metasploit resmi.
  

- **Kondisi spesifik yang dibutuhkan:**

  1. Target menggunakan distribusi vsftpd 2.3.4 yang mengandung backdoor.
  2. Service FTP dapat diakses melalui jaringan.
  3. Port FTP yang digunakan sesuai dengan konfigurasi modul, yaitu port 21 pada pengujian ini.
  4. Sistem target harus memenuhi kondisi yang diperlukan oleh modul Metasploit.
  5. Versi software saja tidak cukup untuk memastikan bahwa instalasi target mengandung backdoor. Asal distribusi binary juga perlu diperhatikan.

## Kesimpulan

- **Apakah exploit ini applicable ke target Metasploitable2 kamu? Kenapa?**

  Berdasarkan hasil enumerasi menggunakan Nmap, Metasploitable2 memiliki service FTP pada port 21 dengan banner:

  `220 (vsFTPd 2.3.4)`

  Hal ini menunjukkan bahwa versi service target sesuai dengan versi yang disebutkan dalam CVE-2011-2523.

  Selain itu, modul Metasploit resmi untuk vsftpd 2.3.4 berhasil ditemukan dan diperiksa. Konfigurasi RHOSTS juga telah diarahkan ke IP Metasploitable2, yaitu `192.168.56.102`.

  Namun, applicability secara penuh belum dapat dipastikan karena belum ada bukti bahwa binary vsftpd yang terpasang berasal dari arsip yang telah disusupi backdoor. Modul Metasploit juga belum dijalankan sehingga keberhasilan eksploitasi belum diketahui.

  Dengan demikian, target memiliki kecocokan versi dan service, tetapi status kerentanan aktual belum terkonfirmasi.

- **Perbedaan antara apa yang dijelaskan advisory vs apa yang benar-benar ada di kode PoC (jika ada):**

  NVD menjelaskan kerentanan dari sudut pandang dampak dan identifikasi, yaitu adanya backdoor pada distribusi vsftpd 2.3.4 yang dapat membuka shell pada port 6200/tcp.

  Sementara itu, modul Metasploit memberikan gambaran teknis tentang bagaimana kerentanan tersebut dapat dimanfaatkan melalui modul command execution pada sistem Unix/Linux.

  Advisory berfokus pada deskripsi kerentanan, versi terdampak, tingkat keparahan, dan referensi penanganan. Modul exploit berfokus pada mekanisme pemanfaatan kerentanan.

  Keduanya saling melengkapi untuk mengevaluasi apakah suatu exploit relevan terhadap target.

## Referensi

1. NVD — CVE-2011-2523:
   https://nvd.nist.gov/vuln/detail/CVE-2011-2523

2. Red Hat — CVE-2011-2523:
   https://access.redhat.com/security/cve/cve-2011-2523

3. Rapid7 — VSFTPD 2.3.4 Backdoor:
   https://www.rapid7.com/db/modules/exploit/unix/ftp/vsftpd_234_backdoor/

4. Packet Storm — VSFTPD 2.3.4 Backdoor Command Execution:
   https://packetstormsecurity.com/files/102745/VSFTPD-2.3.4-Backdoor-Command-Execution.html

5. MITRE — CWE-78:
   https://cwe.mitre.org/data/definitions/78.html
