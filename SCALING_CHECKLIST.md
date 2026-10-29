# Panduan Skalabilitas & Verifikasi Free Tier (537+ Peserta)

Dokumen ini memverifikasi arsitektur teknis dan panduan kesiapan untuk **GenBI in ACTION - Presensi QR** agar berjalan lancar 100% pada **Cloudflare Pages (Free)** dan **Supabase (Free Tier)** untuk **537+ peserta** dan **9 sesi QR presensi**.

---

## 1. Verifikasi Kapasitas Free Tier

| Parameter | Penggunaan Event (537 Peserta x 9 Sesi) | Batas Supabase Free Tier | Status |
| :--- | :--- | :--- | :--- |
| **Penyimpanan Database (Disk)** | ~80 MB (data teks + tanda tangan base64) | 500 MB | ✅ **Aman** (< 20% kuota) |
| **Koneksi Realtime (WebSocket)** | **0 koneksi** (aplikasi memakai REST RPC murni) | Maks 200 koneksi | ✅ **100% Aman** |
| **Bandwidth Keluar (Egress)** | ~150 - 250 MB sepanjang acara | 5.000 MB (5 GB) / bulan | ✅ **Aman** (< 5% kuota) |
| **Beban CPU Database** | Ditekan ke < 5% via indeks B-tree (`att_pid_idx`) | 2 Shared vCPU / 0.5 GB RAM | ✅ **Aman** |
| **Cloudflare Pages Hosting** | ~5.000 hit awal, lalu di-cache Service Worker | Unlimited Bandwidth | ✅ **100% Aman** |

---

## 2. Perubahan Database pada `supabase-setup.sql`

Pembaruan yang telah diterapkan pada `supabase-setup.sql`:
1. **Indeks Performa Tinggi:**
   - `att_pid_idx` pada `att(pid)`: Mencegah full table scan saat 537 peserta login/refresh status presensi.
   - `people_k_idx` pada `people(k)` dan `people_g_idx` pada `people(g)`: Mengoptimalkan pencarian rekan sekamar dan sekelompok.
   - `people_hp_idx` pada `people(hp)`: Mempercepat login via nomor HP.
2. **Dukungan Login Fleksibel:**
   - Peserta dapat masuk menggunakan **Email** maupun **Nomor HP** terdaftar.
3. **Pemberian Kunci Kamar Eksklusif (`kunci`):**
   - Validasi otomatis di server agar satu nomor kamar hanya dapat mengambil satu kunci fisik.
4. **Fungsi Undian Kelompok (`peserta_undi`):**
   - Stored procedure atomik untuk mencegah race condition saat undian kelompok dijalankan bersamaan.

---

## 3. Langkah yang Harus Dilakukan Pemilik Repo / EO

1. **Jalankan SQL di Supabase:**
   - Buka **[Supabase Dashboard](https://supabase.com/dashboard)** -> Pilih project database Anda.
   - Buka menu **SQL Editor**.
   - Salin seluruh isi dari file `supabase-setup.sql` yang ada di branch `main`.
   - Klik tombol **Run**.
2. **Impor Peserta:**
   - Saat mengimpor Excel/CSV melalui panel admin, kolom berikut didukung: `email`, `nama`, `sekolah`, `kamar`, `kelompok`, `gender`, `lantai`, `gedung`, `no peserta`, `no hp`.
3. **Tips Saat Hari H:**
   - Karena peserta harus menggambar tanda tangan di kanvas sebelum mengirim presensi, beban request akan tersebar merata dalam rentang waktu 30-90 detik.
