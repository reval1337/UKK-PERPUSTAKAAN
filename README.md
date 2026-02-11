# 📊 LAPORAN SISTEM PERPUSTAKAAN
# SMKN12 - Library Management System

---

## 📑 Daftar Isi

1. [Laporan Anggota](#1-laporan-anggota)
2. [Laporan Buku](#2-laporan-buku)
3. [Laporan Peminjaman](#3-laporan-peminjaman)
4. [Laporan Statistik](#4-laporan-statistik)
5. [Laporan Keuangan](#5-laporan-keuangan)
6. [Laporan Analitik](#6-laporan-analitik)

---

## 1. 👥 LAPORAN ANGGOTA

### 1.1 Daftar Anggota Aktif

**Tujuan:** Menampilkan semua anggota dengan status aktif

**Query SQL:**
```sql
SELECT 
    id_anggota,
    nama,
    nis,
    kelas,
    email,
    telepon,
    tanggal_daftar,
    (SELECT COUNT(*) FROM PEMINJAMAN 
     WHERE id_anggota = ANGGOTA.id_anggota 
     AND status = 'dipinjam') as buku_dipinjam
FROM ANGGOTA
WHERE status = 'aktif'
ORDER BY nama ASC;
```

**Format Laporan:**

| No | ID Anggota | Nama | NIS | Kelas | Email | Telepon | Tgl Daftar | Buku Dipinjam |
|----|------------|------|-----|-------|-------|---------|------------|---------------|
| 1 | A001 | Ahmad Fauzi | 12345 | XII RPL 1 | ahmad@smkn12.sch.id | 081234567890 | 01/09/2025 | 2 |
| 2 | A002 | Budi Santoso | 12346 | XII RPL 2 | budi@smkn12.sch.id | 081234567891 | 01/09/2025 | 1 |
| 3 | A003 | Citra Dewi | 12347 | XI RPL 1 | citra@smkn12.sch.id | 081234567892 | 01/09/2025 | 0 |

**Total Anggota Aktif:** 150 siswa

---

### 1.2 Daftar Anggota Tidak Aktif

**Tujuan:** Menampilkan anggota yang sudah tidak aktif (lulus/pindah)

**Query SQL:**
```sql
SELECT 
    id_anggota,
    nama,
    nis,
    kelas,
    tanggal_daftar,
    status,
    alasan_nonaktif
FROM ANGGOTA
WHERE status = 'tidak aktif'
ORDER BY nama ASC;
```

**Format Laporan:**

| No | ID Anggota | Nama | NIS | Kelas | Status | Alasan | Tgl Nonaktif |
|----|------------|------|-----|-------|--------|--------|--------------|
| 1 | A100 | Dedi Kurniawan | 11234 | XII RPL 1 | Tidak Aktif | Lulus | 15/06/2025 |
| 2 | A101 | Eka Putri | 11235 | XII TKJ 1 | Tidak Aktif | Pindah Sekolah | 20/03/2025 |

**Total Anggota Tidak Aktif:** 25 siswa

---

### 1.3 Anggota dengan Peminjaman Terbanyak

**Query SQL:**
```sql
SELECT 
    A.id_anggota,
    A.nama,
    A.kelas,
    COUNT(P.id_peminjaman) as total_peminjaman,
    COUNT(CASE WHEN P.status = 'dipinjam' THEN 1 END) as sedang_dipinjam
FROM ANGGOTA A
LEFT JOIN PEMINJAMAN P ON A.id_anggota = P.id_anggota
WHERE A.status = 'aktif'
GROUP BY A.id_anggota, A.nama, A.kelas
ORDER BY total_peminjaman DESC
LIMIT 10;
```

**Top 10 Peminjam Aktif:**

| Rank | Nama | Kelas | Total Peminjaman | Sedang Dipinjam |
|------|------|-------|------------------|-----------------|
| 1 | Ahmad Fauzi | XII RPL 1 | 45 | 3 |
| 2 | Budi Santoso | XII RPL 2 | 38 | 2 |
| 3 | Citra Dewi | XI RPL 1 | 32 | 1 |

---

## 2. 📚 LAPORAN BUKU

### 2.1 Daftar Buku Tersedia

**Tujuan:** Menampilkan buku yang tersedia untuk dipinjam

**Query SQL:**
```sql
SELECT 
    B.id_buku,
    B.judul,
    B.penulis,
    B.penerbit,
    B.tahun_terbit,
    B.stok,
    P.nama as pengarang,
    GROUP_CONCAT(K.nama_kategori) as kategori
FROM BUKU B
LEFT JOIN PENGARANG P ON B.id_pengarang = P.id_pengarang
LEFT JOIN BUKU_KATEGORI BK ON B.id_buku = BK.id_buku
LEFT JOIN KATEGORI K ON BK.id_kategori = K.id_kategori
WHERE B.status = 'tersedia' AND B.stok > 0
GROUP BY B.id_buku
ORDER BY B.judul ASC;
```

**Format Laporan:**

| No | ID Buku | Judul | Penulis | Penerbit | Tahun | Stok | Kategori |
|----|---------|-------|---------|----------|-------|------|----------|
| 1 | B001 | Pemrograman Flutter | John Doe | Gramedia | 2023 | 5 | Teknologi, Pemrograman |
| 2 | B002 | Basis Data MySQL | Jane Smith | Erlangga | 2022 | 3 | Database, Teknologi |
| 3 | B003 | Algoritma & Struktur Data | Bob Johnson | Andi | 2024 | 7 | Pemrograman, Algoritma |

**Total Buku Tersedia:** 450 eksemplar (dari 120 judul)

---

### 2.2 Daftar Buku Sedang Dipinjam

**Tujuan:** Menampilkan buku yang sedang dalam status peminjaman

**Query SQL:**
```sql
SELECT 
    B.id_buku,
    B.judul,
    B.penulis,
    A.nama as peminjam,
    A.kelas,
    P.tanggal_peminjaman,
    P.tanggal_kembali,
    DATEDIFF(CURRENT_DATE, P.tanggal_kembali) as hari_terlambat,
    CASE 
        WHEN CURRENT_DATE > P.tanggal_kembali THEN 'Terlambat'
        ELSE 'Normal'
    END as status_peminjaman
FROM BUKU B
INNER JOIN PEMINJAMAN P ON B.id_buku = P.id_buku
INNER JOIN ANGGOTA A ON P.id_anggota = A.id_anggota
WHERE P.status = 'dipinjam'
ORDER BY P.tanggal_peminjaman ASC;
```

**Format Laporan:**

| No | Judul Buku | Peminjam | Kelas | Tgl Pinjam | Tgl Kembali | Status | Hari Terlambat |
|----|------------|----------|-------|------------|-------------|--------|----------------|
| 1 | Pemrograman Flutter | Ahmad Fauzi | XII RPL 1 | 01/02/2026 | 08/02/2026 | Terlambat | 2 hari |
| 2 | Basis Data MySQL | Budi Santoso | XII RPL 2 | 05/02/2026 | 12/02/2026 | Normal | - |

**Total Buku Dipinjam:** 85 eksemplar

---

### 2.3 Daftar Buku Hilang/Rusak

**Tujuan:** Menampilkan buku yang hilang atau rusak

**Query SQL:**
```sql
SELECT 
    id_buku,
    judul,
    penulis,
    penerbit,
    tahun_terbit,
    status,
    keterangan,
    tanggal_status
FROM BUKU
WHERE status IN ('hilang', 'rusak')
ORDER BY tanggal_status DESC;
```

**Format Laporan:**

| No | ID Buku | Judul | Penulis | Status | Keterangan | Tgl Status |
|----|---------|-------|---------|--------|------------|------------|
| 1 | B050 | Jaringan Komputer | Ahmad Yani | Hilang | Tidak dikembalikan | 15/01/2026 |
| 2 | B051 | Web Programming | Siti Nurhaliza | Rusak | Halaman sobek | 20/01/2026 |

**Total Buku Hilang:** 5 eksemplar  
**Total Buku Rusak:** 3 eksemplar

---

### 2.4 Buku Paling Populer

**Query SQL:**
```sql
SELECT 
    B.id_buku,
    B.judul,
    B.penulis,
    COUNT(P.id_peminjaman) as total_dipinjam,
    AVG(DATEDIFF(P.tanggal_pengembalian, P.tanggal_peminjaman)) as rata_rata_hari
FROM BUKU B
INNER JOIN PEMINJAMAN P ON B.id_buku = P.id_buku
WHERE P.tanggal_pengembalian IS NOT NULL
GROUP BY B.id_buku, B.judul, B.penulis
ORDER BY total_dipinjam DESC
LIMIT 10;
```

**Top 10 Buku Terpopuler:**

| Rank | Judul | Penulis | Total Dipinjam | Rata-rata Hari Pinjam |
|------|-------|---------|----------------|----------------------|
| 1 | Pemrograman Flutter | John Doe | 125 | 6.5 hari |
| 2 | Basis Data MySQL | Jane Smith | 98 | 7.2 hari |
| 3 | Algoritma & Struktur Data | Bob Johnson | 87 | 5.8 hari |

---

## 3. 🔄 LAPORAN PEMINJAMAN

### 3.1 Daftar Peminjaman Aktif

**Tujuan:** Menampilkan semua peminjaman yang sedang berjalan

**Query SQL:**
```sql
SELECT 
    P.id_peminjaman,
    A.nama as peminjam,
    A.kelas,
    B.judul as buku,
    P.tanggal_peminjaman,
    P.tanggal_kembali,
    DATEDIFF(P.tanggal_kembali, CURRENT_DATE) as sisa_hari
FROM PEMINJAMAN P
INNER JOIN ANGGOTA A ON P.id_anggota = A.id_anggota
INNER JOIN BUKU B ON P.id_buku = B.id_buku
WHERE P.status = 'dipinjam'
ORDER BY P.tanggal_kembali ASC;
```

**Format Laporan:**

| No | ID Peminjaman | Peminjam | Kelas | Judul Buku | Tgl Pinjam | Tgl Kembali | Sisa Hari |
|----|---------------|----------|-------|------------|------------|-------------|-----------|
| 1 | P001 | Ahmad Fauzi | XII RPL 1 | Pemrograman Flutter | 01/02/2026 | 08/02/2026 | -2 (Terlambat) |
| 2 | P002 | Budi Santoso | XII RPL 2 | Basis Data MySQL | 05/02/2026 | 12/02/2026 | 2 hari |

**Total Peminjaman Aktif:** 85 transaksi

---

### 3.2 Daftar Peminjaman Terlambat

**Tujuan:** Menampilkan peminjaman yang melewati tanggal kembali

**Query SQL:**
```sql
SELECT 
    P.id_peminjaman,
    A.nama as peminjam,
    A.telepon,
    A.email,
    B.judul as buku,
    P.tanggal_peminjaman,
    P.tanggal_kembali,
    DATEDIFF(CURRENT_DATE, P.tanggal_kembali) as hari_terlambat,
    (DATEDIFF(CURRENT_DATE, P.tanggal_kembali) * 1000) as denda
FROM PEMINJAMAN P
INNER JOIN ANGGOTA A ON P.id_anggota = A.id_anggota
INNER JOIN BUKU B ON P.id_buku = B.id_buku
WHERE P.status = 'dipinjam' 
  AND CURRENT_DATE > P.tanggal_kembali
ORDER BY hari_terlambat DESC;
```

**Format Laporan:**

| No | Peminjam | Kontak | Judul Buku | Tgl Kembali | Hari Terlambat | Denda |
|----|----------|--------|------------|-------------|----------------|-------|
| 1 | Ahmad Fauzi | 081234567890 | Pemrograman Flutter | 08/02/2026 | 2 hari | Rp 2.000 |
| 2 | Dewi Lestari | 081234567891 | Web Programming | 05/02/2026 | 5 hari | Rp 5.000 |

**Total Peminjaman Terlambat:** 12 transaksi  
**Total Denda Tertunggak:** Rp 45.000

---

### 3.3 Daftar Peminjaman Selesai

**Tujuan:** Menampilkan riwayat peminjaman yang sudah dikembalikan

**Query SQL:**
```sql
SELECT 
    P.id_peminjaman,
    A.nama as peminjam,
    B.judul as buku,
    P.tanggal_peminjaman,
    P.tanggal_kembali,
    P.tanggal_pengembalian,
    DATEDIFF(P.tanggal_pengembalian, P.tanggal_peminjaman) as lama_pinjam,
    P.denda,
    CASE 
        WHEN P.tanggal_pengembalian > P.tanggal_kembali THEN 'Terlambat'
        ELSE 'Tepat Waktu'
    END as status_pengembalian
FROM PEMINJAMAN P
INNER JOIN ANGGOTA A ON P.id_anggota = A.id_anggota
INNER JOIN BUKU B ON P.id_buku = B.id_buku
WHERE P.status = 'dikembalikan'
ORDER BY P.tanggal_pengembalian DESC
LIMIT 50;
```

**Format Laporan (50 Terakhir):**

| No | Peminjam | Judul Buku | Tgl Pinjam | Tgl Kembali | Tgl Pengembalian | Lama Pinjam | Denda | Status |
|----|----------|------------|------------|-------------|------------------|-------------|-------|--------|
| 1 | Citra Dewi | Algoritma | 25/01/2026 | 01/02/2026 | 31/01/2026 | 6 hari | Rp 0 | Tepat Waktu |
| 2 | Eko Prasetyo | Jaringan | 20/01/2026 | 27/01/2026 | 30/01/2026 | 10 hari | Rp 3.000 | Terlambat |

**Total Peminjaman Selesai (Bulan Ini):** 156 transaksi  
**Persentase Tepat Waktu:** 87%

---

### 3.4 Laporan Peminjaman Per Periode

**Tujuan:** Menampilkan statistik peminjaman dalam periode tertentu

**Query SQL:**
```sql
SELECT 
    DATE_FORMAT(tanggal_peminjaman, '%Y-%m') as bulan,
    COUNT(*) as total_peminjaman,
    COUNT(CASE WHEN status = 'dikembalikan' THEN 1 END) as selesai,
    COUNT(CASE WHEN status = 'dipinjam' THEN 1 END) as aktif,
    SUM(denda) as total_denda
FROM PEMINJAMAN
WHERE tanggal_peminjaman >= DATE_SUB(CURRENT_DATE, INTERVAL 6 MONTH)
GROUP BY DATE_FORMAT(tanggal_peminjaman, '%Y-%m')
ORDER BY bulan DESC;
```

**Laporan 6 Bulan Terakhir:**

| Bulan | Total Peminjaman | Selesai | Aktif | Total Denda |
|-------|------------------|---------|-------|-------------|
| Feb 2026 | 45 | 30 | 15 | Rp 12.000 |
| Jan 2026 | 156 | 145 | 11 | Rp 45.000 |
| Des 2025 | 178 | 178 | 0 | Rp 67.000 |
| Nov 2025 | 165 | 165 | 0 | Rp 52.000 |

---

## 4. 📊 LAPORAN STATISTIK

### 4.1 Statistik Umum

**Dashboard Statistik Perpustakaan:**

```
╔════════════════════════════════════════════════════════════╗
║           STATISTIK PERPUSTAKAAN SMKN12                    ║
║                Per 10 Februari 2026                        ║
╠════════════════════════════════════════════════════════════╣
║                                                            ║
║  📚 KOLEKSI BUKU                                           ║
║  ├─ Total Judul Buku        : 120 judul                   ║
║  ├─ Total Eksemplar         : 450 eksemplar               ║
║  ├─ Buku Tersedia           : 365 eksemplar (81%)         ║
║  ├─ Buku Dipinjam           : 85 eksemplar (19%)          ║
║  ├─ Buku Hilang             : 5 eksemplar                 ║
║  └─ Buku Rusak              : 3 eksemplar                 ║
║                                                            ║
║  👥 ANGGOTA                                                ║
║  ├─ Total Anggota           : 175 siswa                   ║
║  ├─ Anggota Aktif           : 150 siswa (86%)             ║
║  ├─ Anggota Tidak Aktif     : 25 siswa (14%)              ║
║  └─ Anggota Baru (Bulan Ini): 5 siswa                     ║
║                                                            ║
║  🔄 PEMINJAMAN                                             ║
║  ├─ Peminjaman Aktif        : 85 transaksi                ║
║  ├─ Peminjaman Terlambat    : 12 transaksi (14%)          ║
║  ├─ Total Bulan Ini         : 45 transaksi                ║
║  ├─ Total Tahun Ini         : 201 transaksi               ║
║  └─ Rata-rata per Hari      : 3.2 transaksi               ║
║                                                            ║
║  💰 KEUANGAN                                               ║
║  ├─ Denda Bulan Ini         : Rp 12.000                   ║
║  ├─ Denda Tahun Ini         : Rp 57.000                   ║
║  └─ Denda Tertunggak        : Rp 45.000                   ║
║                                                            ║
╚════════════════════════════════════════════════════════════╝
```

---

### 4.2 Statistik Per Kategori

**Query SQL:**
```sql
SELECT 
    K.nama_kategori,
    COUNT(DISTINCT BK.id_buku) as jumlah_buku,
    COUNT(P.id_peminjaman) as total_peminjaman,
    ROUND(COUNT(P.id_peminjaman) * 100.0 / 
          (SELECT COUNT(*) FROM PEMINJAMAN), 2) as persentase
FROM KATEGORI K
LEFT JOIN BUKU_KATEGORI BK ON K.id_kategori = BK.id_kategori
LEFT JOIN PEMINJAMAN P ON BK.id_buku = P.id_buku
GROUP BY K.id_kategori, K.nama_kategori
ORDER BY total_peminjaman DESC;
```

**Statistik Kategori Buku:**

| Kategori | Jumlah Buku | Total Peminjaman | Persentase | Popularitas |
|----------|-------------|------------------|------------|-------------|
| Teknologi | 45 | 450 | 35% | ⭐⭐⭐⭐⭐ |
| Pemrograman | 38 | 380 | 29% | ⭐⭐⭐⭐⭐ |
| Database | 25 | 210 | 16% | ⭐⭐⭐⭐ |
| Jaringan | 20 | 180 | 14% | ⭐⭐⭐⭐ |
| Desain | 15 | 80 | 6% | ⭐⭐⭐ |

---

### 4.3 Trend Peminjaman

**Grafik Trend (Data):**

```
Peminjaman Per Bulan (6 Bulan Terakhir)

200 │                    ●
    │                   ╱ ╲
180 │                  ╱   ╲
    │                 ╱     ●
160 │                ╱       ╲
    │               ●         ╲
140 │              ╱           ╲
    │             ╱             ●
120 │            ╱
    │           ╱
100 │          ●
    │
 80 │
    └─────────────────────────────────
      Sep  Okt  Nov  Des  Jan  Feb
     2025 2025 2025 2025 2026 2026
```

**Analisis:**
- Puncak peminjaman: November 2025 (178 transaksi)
- Penurunan: Desember-Januari (libur semester)
- Trend naik: Februari 2026 (awal semester baru)

---

## 5. 💰 LAPORAN KEUANGAN

### 5.1 Laporan Denda

**Denda Per Bulan:**

| Bulan | Jumlah Terlambat | Total Denda | Denda Terbayar | Denda Tertunggak |
|-------|------------------|-------------|----------------|------------------|
| Feb 2026 | 8 | Rp 12.000 | Rp 8.000 | Rp 4.000 |
| Jan 2026 | 15 | Rp 45.000 | Rp 30.000 | Rp 15.000 |
| Des 2025 | 22 | Rp 67.000 | Rp 50.000 | Rp 17.000 |

**Total Denda Tahun 2026:** Rp 57.000  
**Total Terbayar:** Rp 38.000 (67%)  
**Total Tertunggak:** Rp 19.000 (33%)

---

### 5.2 Top Denda Tertunggak

| No | Nama Peminjam | Kelas | Jumlah Keterlambatan | Total Denda |
|----|---------------|-------|----------------------|-------------|
| 1 | Ahmad Fauzi | XII RPL 1 | 3 kali | Rp 8.000 |
| 2 | Dewi Lestari | XI TKJ 2 | 2 kali | Rp 5.000 |
| 3 | Eko Prasetyo | XII MM 1 | 2 kali | Rp 4.000 |

---

## 6. 📈 LAPORAN ANALITIK

### 6.1 Analisis Perilaku Peminjam

**Segmentasi Peminjam:**

| Segmen | Kriteria | Jumlah | Persentase |
|--------|----------|--------|------------|
| Heavy User | >10 peminjaman/semester | 25 | 17% |
| Regular User | 5-10 peminjaman/semester | 60 | 40% |
| Light User | 1-4 peminjaman/semester | 45 | 30% |
| Inactive | 0 peminjaman/semester | 20 | 13% |

---

### 6.2 Analisis Waktu Peminjaman

**Hari Tersibuk:**

| Hari | Jumlah Peminjaman | Persentase |
|------|-------------------|------------|
| Senin | 45 | 22% |
| Selasa | 38 | 19% |
| Rabu | 42 | 21% |
| Kamis | 35 | 17% |
| Jumat | 41 | 20% |

**Jam Tersibuk:**
- 08:00 - 10:00: 35%
- 10:00 - 12:00: 40%
- 13:00 - 15:00: 25%

---

### 6.3 Rekomendasi

Berdasarkan analisis data, berikut rekomendasi untuk perpustakaan:

1. **Penambahan Koleksi**
   - Tambah buku kategori Teknologi & Pemrograman (demand tinggi)
   - Pertimbangkan buku digital untuk kategori populer

2. **Optimasi Operasional**
   - Tambah petugas di jam 10:00-12:00 (jam sibuk)
   - Sistem reminder otomatis untuk peminjaman mendekati jatuh tempo

3. **Engagement Anggota**
   - Program reward untuk heavy users
   - Campaign untuk mengaktifkan inactive members

4. **Pengelolaan Koleksi**
   - Review buku dengan peminjaman rendah
   - Ganti buku rusak dengan judul populer

---

## 📝 Catatan Penutup

**Periode Laporan:** 10 Februari 2026  
**Dibuat oleh:** Sistem Perpustakaan SMKN12  
**Frekuensi Update:** Harian (otomatis)

**Kontak:**
- Email: perpustakaan@smkn12.sch.id
- Telepon: (021) 1234-5678
- Website: library.smkn12.sch.id

---

**Disclaimer:** Data dalam laporan ini bersifat dinamis dan dapat berubah sesuai dengan transaksi yang terjadi.

---

*Laporan ini dibuat secara otomatis oleh Sistem Manajemen Perpustakaan Digital SMKN12*
