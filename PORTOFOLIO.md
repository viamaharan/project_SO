# Portofolio Individu Project Akhir Sistem Operasi

## 1. Identitas Mahasiswa
* **Nama:** Desvia Maharani Kurniawan
* **NIM:** 2510512023
* **Kelompok:** 6A
* **Peran:** Anggota 4, Administrasi SSH
* **Judul Proyek:** Server Berkas Bersama dengan Backup dan Audit Akses (Pilihan 6A)
* **Mata Kuliah:** Sistem Operasi & Praktikum Sistem Operasi (Semester Ganjil 2026/2027)

---

## 2. Asal Proyek & Tim Kelompok
Proyek ini dikembangkan secara berkelompok sebagai solusi administrasi server Linux menggunakan Ubuntu Server.

* **Tautan Repository Utama Kelompok:** https://github.com/Project-SO-6A/project_SO
* **Tautan Repository Fork Pribadi:** https://github.com/viamaharan/project_SO
* **Daftar Anggota Kelompok 6A:**
  1. **Anggota 1:** Chesya Ramadhani Putri Widodo: *User, group, dan permission Linux*
  2. **Anggota 2:** Swastika Sisilia: *Konfigurasi Samba dan pengujian klien*
  3. **Anggota 3:** Ainnisya Rezka: *Pengujian hak akses pembaca (read-only)*
  4. **Anggota 4:** Desvia Maharani Kurniawan: *Administrasi SSH dan pembatasan akses tindakan admin*
  5. **Anggota 5:** Nazwa Auliya: *Backup dan pemulihan (restore) data*
  6. **Anggota 6:** Adelia khalimatul: *Monitoring layanan, log operasional, laporan, dan aktivitas Docker*

---

## 3. Kontribusi Pribadi
Sebagai **Anggota 4**, saya bertanggung jawab merancang, mengonfigurasi, dan menguji mekanisme administrasi jarak jauh menggunakan **SSH (Secure Shell)** pada server Linux agar pengelolaan layanan hanya dapat dilakukan oleh akun administrator yang berwenang.

Rincian kontribusi teknis yang saya kerjakan:
1. **Konfigurasi Akses & Hardening Dasar SSH:** Mengatur layanan `sshd` pada Ubuntu Server agar mendukung autentikasi jarak jauh yang aman serta membatasi hak eksekusi perintah administratif menggunakan mekanisme `sudoers` dan grup administrator.
2. **Pembuatan Shell Script Pengujian (`test_ssh.sh`):** Menyusun script pengujian otomatis di direktori `tests/anggota4/test_ssh.sh` (dan/atau `scripts/`) untuk memvalidasi koneksi SSH, pemeriksaan status layanan (`smbd`, `ssh`), pencatatan *exit status*, serta penyimpanan log uji akses.
3. **Dokumentasi Teknis Anggota 4:** Menyusun laporan skenario dan panduan reproduksi pengujian pada `tests/anggota4/README.md` serta berkontribusi pada penyusunan laporan akhir kelompok.

---

## 4. Bukti Kontribusi Git
Seluruh perubahan kode dan dokumentasi telah dikontribusikan ke repository utama kelompok dengan riwayat yang dapat ditelusuri:
* **Tautan Pull Request / Commit Utama:** `[Tempelkan link Pull Request atau Commit kamu dari repo Project-SO-6A/project_SO di sini]`
* **Berkas Utama yang Dikerjakan:**
  * `tests/anggota4/test_ssh.sh` (Script pengujian skenario positif & negatif SSH)
  * `tests/anggota4/README
 
## 5. Skenario Pengujian Individu (Positif & Negatif)

### Tujuan Skenario
Memastikan administrator dapat melakukan pemeriksaan status layanan dan administrasi server secara jarak jauh melalui SSH, sekaligus membuktikan bahwa pengguna biasa (non-admin) ditolak saat mencoba menjalankan perintah administratif yang dibatasi.

### A. Kondisi Positif (Admin Melakukan Pemeriksaan via SSH)
* **Prasyarat:** Layanan `ssh` aktif pada server; akun administrator (misal: `admin_so`) memiliki hak `sudo` yang sesuai dan terhubung ke jaringan lokal VM.
* **Langkah Pengujian:**
  1. Klien melakukan koneksi SSH menggunakan akun admin (`ssh admin_so@<ip-server>`).
  2. Menjalankan perintah pemeriksaan status layanan dan sistem (misal: `systemctl status smbd` dan script pemeriksaan layanan).
  3. Memeriksa *exit status* (`echo $?`) serta catatan log pengujian.
* **Hasil yang Diharapkan & Aktual:** Pemeriksaan berhasil dijalankan tanpa penolakan izin, status layanan tampil akurat, pencatatan log berhasil, dan perintah selesai dengan **exit status `0`**.

### B. Kondisi Negatif (Akun Biasa Ditolak Menjalankan Tindakan Admin)
* **Prasyarat:** Akun pengguna biasa / pembaca (misal: `pembaca1`) terdaftar di sistem tanpa hak akses grup `sudo` atau administrator.
* **Langkah Pengujian:**
  1. Klien melakukan koneksi SSH menggunakan akun biasa (`ssh pembaca1@<ip-server>`).
  2. Mencoba menjalankan tindakan administratif yang dibatasi (misal: `sudo systemctl restart smbd` atau mengakses direktori konfigurasi terbatas `/etc/samba/`).
  3. Memeriksa pesan galat dan *exit status* (`echo $?`) segera setelah perintah dijalankan.
* **Hasil yang Diharapkan & Aktual:** Sistem menolak eksekusi perintah dengan pesan *permission denied / user is not in the sudoers file*, layanan tidak terganggu, dan perintah menghasilkan **exit status bukan nol (non-zero)** yang tercatat dalam log audit.

### Pemulihan Kondisi Uji
Membersihkan sesi SSH aktif setelah pengujian selesai dan memastikan file log kembali ke status siap pakai tanpa mengubah konfigurasi layanan utama.

---

## 6. Analisis Konsep Sistem Operasi
Implementasi administrasi SSH pada proyek ini berkaitan langsung dengan beberapa konsep inti Sistem Operasi:
1. **Autentikasi dan Otorisasi:** SSH memverifikasi identitas pengguna (*authentication*) sebelum membuka sesi shell jarak jauh. Setelah masuk, kernel Linux dan mekanisme `sudo` menegakkan otorisasi (*authorization*) berdasarkan UID, GID, dan kebijakan `/etc/sudoers` untuk menentukan perintah apa yang boleh dijalankan.
2. **Proteksi Sistem dan Manajemen Hak Akses (*Privilege Separation*):** Pemisahan antara akun administrator dan akun biasa menerapkan prinsip *least privilege*. Pengguna biasa berjalan dalam ruang pengguna (*user space*) dengan hak terbatas sehingga tidak dapat memodifikasi proses layanan sistem atau berkas konfigurasi sensitif.
3. **Manajemen Proses dan Layanan (*Daemon*):** Layanan `sshd` berjalan sebagai proses latar belakang (*daemon*) yang mendengarkan koneksi masuk pada port 22 dan melakukan *fork* proses anak baru untuk setiap sesi SSH klien yang terhubung, lengkap dengan pencatatan PID dan *exit status* saat sesi berakhir.
4. **Sistem Berkas dan Log I/O:** Setiap aktivitas login, penolakan akses `sudo`, dan keluaran script pengujian ditulis ke sistem berkas log (`/var/log/auth.log` serta log khusus proyek) sebagai jejak audit operasional.

---

## 7. Kendala dan Penyelesaian
* **Kendala 1 — Kesalahan Penamaan dan Penempatan Berkas di Repository:** Pada tahap awal kolaborasi Git, berkas script pengujian sempat terunggah langsung di luar folder khusus anggota dengan nama yang belum seragam (`Anggota 4.sh`).
  * **Penyelesaian:** Menata ulang struktur direktori menjadi `tests/anggota4/test_ssh.sh` dan menambahkan `README.md` agar konsisten dengan struktur standar seluruh anggota kelompok.
* **Kendala 2 — Pengujian Akses Antar-Akun VM:** Saat menguji skenario negatif, perlu dipastikan bahwa akun uji biasa benar-benar tidak mewarisi hak grup `sudo` dan pesan penolakan tetap tertangkap rapi oleh script pengujian tanpa menghentikan eksekusi log.
  * **Penyelesaian:** Melakukan verifikasi keanggotaan grup menggunakan perintah `groups <username>` sebelum uji coba serta menambahkan penanganan *exit status* (`$?`) dan validasi input pada shell script.

---

## 8. Refleksi Pembelajaran
* **Hal yang Sudah Dikuasai:** Memahami cara kerja layanan SSH di Ubuntu Server, penerapan pemisahan hak akses antara akun admin dan non-admin, penulisan shell script pengujian dengan pengecekan *exit status*, serta alur kolaborasi Git menggunakan *Fork*, *Branch*, *Commit*, dan *Pull Request*.
* **Hal yang Masih Perlu Dipelajari:** Pendalaman konfigurasi *hardening* SSH tingkat lanjut seperti penggunaan autentikasi berbasis kunci publik (*SSH key-pair*) secara menyeluruh, pembatasan akses berbasis IP/firewall (`ufw`), serta integrasi monitoring log otomatis secara *real-time*.
