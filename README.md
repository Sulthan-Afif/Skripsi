# Implementasi Algoritma ARIMA dan LSTM pada Peramalan Penjualan Obat di RSUD Arifin Achmad Provinsi Riau

> Skripsi | Program Studi Sistem Informasi, Universitas Riau | 2025
> Nama: Muhammad Sulthan Muqsith Afif | NIM: 2103125616

## 1. Ringkasan Eksekutif

Skripsi ini membandingkan dua algoritma peramalan, ARIMA dan LSTM, untuk meramalkan penjualan obat di RSUD Arifin Achmad Provinsi Riau.

Data yang dipakai adalah data transaksi penjualan obat agregat, dari Januari 2021 sampai Desember 2024, total 2.142 sample. Data yang dipakai bukan data pasien. Data yang dipakai adalah data penjualan per barang per tanggal, yang sudah diagregat per hari.

Hasilnya jelas. LSTM jauh lebih baik daripada ARIMA untuk data ini, baik untuk kategori Generik maupun Non Generik.

Hasil untuk kategori Generik:
- ARIMA: RMSE 13,99, MAE 9,79, MAPE 4,35 persen.
- LSTM: RMSE 1,91, MAE 1,51, MAPE 0,64 persen.

Hasil untuk kategori Non Generik:
- ARIMA: RMSE 2,02, MAE 1,28, MAPE 16,46 persen.
- LSTM: RMSE 0,34, MAE 0,27, MAPE 2,59 persen.

Artinya, untuk data penjualan obat yang polanya tidak linear dan ada fluktuasi tajam, LSTM lebih mampu menangkap pola tersebut daripada ARIMA.

Skripsi ini tidak mengklaim bahwa LSTM selalu lebih baik di semua kasus. Kesimpulannya terbatas pada data dan periode yang dipakai di penelitian ini.

**Status file di repo ini:** Repo ini berisi versi publik dari skripsi. Lampiran surat izin dan bukti pembayaran sudah dihapus dari versi publik karena berisi tanda tangan dan nomor induk pegawai. Link ke data mentah lengkap tidak dibagikan ke publik. Yang dibagikan adalah contoh data agregat 10 sampai 20 baris dan ringkasan hasil.

---

## 2. Latar Belakang dan Masalah

Rumah sakit harus menjaga stok obat agar tidak kosong dan tidak menumpuk.

Kalau stok kosong, pelayanan terganggu. Pasien harus menunggu atau mencari obat di luar. Kalau stok menumpuk, obat bisa kedaluwarsa dan uang rumah sakit tertahan di gudang.

Masalah yang ditemukan:

1. Peramalan stok masih dilakukan berdasarkan perkiraan manual dan pengalaman, bukan berdasarkan pola data historis.
2. Penjualan obat tiap hari naik turun dan tidak selalu mengikuti pola musiman yang sederhana.
3. Metode klasik seperti ARIMA sering gagal ketika pola datanya tidak linear dan ada lonjakan mendadak.
4. Rumah sakit butuh bukti, metode mana yang lebih akurat untuk data mereka, bukan sekadar teori.

Dari masalah itu, rumusan masalahnya menjadi:

1. Bagaimana mengimplementasikan ARIMA dan LSTM untuk peramalan penjualan obat.
2. Metode mana yang lebih akurat berdasarkan RMSE, MAE, dan MAPE.
3. Bagaimana hasil peramalan itu bisa dipakai untuk perencanaan stok.

---

## 3. Tujuan Penelitian

1. Mengimplementasikan ARIMA dan LSTM pada data penjualan obat di RSUD Arifin Achmad.
2. Membandingkan akurasi ARIMA dan LSTM menggunakan RMSE, MAE, dan MAPE.
3. Memberikan rekomendasi metode yang lebih cocok untuk data penjualan obat di lokasi penelitian.

Batasan masalah:

1. Data yang dipakai adalah data penjualan obat agregat per hari, bukan data resep per pasien.
2. Data dibagi menjadi 2 kategori, Generik dan Non Generik, karena perilakunya beda.
3. Periode data Januari 2021 sampai Desember 2024.
4. Evaluasi memakai RMSE, MAE, dan MAPE.
5. Penelitian ini tidak membahas pengadaan, distribusi, dan manajemen gudang secara penuh. Fokusnya hanya peramalan.

---

## 4. Data yang Dipakai

### 4.1. Gambaran Data

- Sumber: RSUD Arifin Achmad Provinsi Riau, dengan izin pengambilan data untuk penelitian.
- Periode: Januari 2021 sampai Desember 2024.
- Jumlah: 2.142 sample setelah agregasi.
- Bentuk awal: Data transaksi per barang per tanggal.
- Bentuk setelah agregasi: Data penjualan per hari per kategori.

Contoh bentuk data setelah agregasi, bukan data asli lengkap:

- Tanggal: 2021-01-01, Kategori: Generik, Jumlah: 120.
- Tanggal: 2021-01-02, Kategori: Generik, Jumlah: 98.
- Tanggal: 2021-01-01, Kategori: Non Generik, Jumlah: 15.
- Tanggal: 2021-01-02, Kategori: Non Generik, Jumlah: 18.

Di laporan asli, contoh data mentah yang ditampilkan adalah tanggal, kode barang, nama barang, satuan, dan jumlah. Contoh nama barang yang muncul adalah Glaucon, Diazepam, dan obat lain yang umum. Data tersebut adalah data penjualan barang, bukan data pasien.

### 4.2. Kenapa Dibagi 2 Kategori

Generik dan Non Generik dipisah karena:

1. Volume penjualannya beda jauh. Generik volumenya besar dan lebih stabil.
2. Pola fluktuasinya beda. Non Generik lebih kecil volumenya tapi lebih sensitif terhadap lonjakan.
3. Jika digabung, pola yang besar akan menutupi pola yang kecil, dan hasil peramalan jadi bias.

Karena itu, semua proses dilakukan dua kali, sekali untuk Generik dan sekali untuk Non Generik.

### 4.3. Pembagian Data Latih dan Uji

Data dibagi menjadi data latih dan data uji dengan pembagian waktu, bukan acak. Alasannya, data deret waktu harus menghormati urutan waktu. Data masa lalu dipakai untuk latih, data masa depan dipakai untuk uji.

Pembagian yang dipakai di penelitian ini mengikuti pembagian yang umum di skripsi, yaitu sebagian besar data awal untuk latih dan sebagian data akhir untuk uji. Detail lengkap ada di laporan bab 3 dan bab 4.

---

## 5. Metodologi

### 5.1. Tahapan Penelitian

1. **Pengumpulan data:** Mengambil data transaksi penjualan obat agregat dengan izin resmi.
2. **Preprocessing:** Membersihkan data yang kosong, duplikat, dan tidak konsisten. Mengagregat data per hari per kategori. Menangani nilai yang hilang dengan cara yang didokumentasikan di laporan.
3. **Eksplorasi data:** Melihat pola, tren, dan fluktuasi. Mengecek apakah data stasioner atau tidak.
4. **Pemodelan ARIMA:** Menentukan orde ARIMA dengan melihat ACF dan PACF, lalu menguji beberapa kombinasi dan memilih yang paling baik berdasarkan kriteria informasi dan error.
5. **Pemodelan LSTM:** Menyusun arsitektur jaringan, menentukan jumlah neuron, lookback, epoch, dan batch, lalu melatih model.
6. **Evaluasi:** Menghitung RMSE, MAE, dan MAPE untuk data uji.
7. **Perbandingan dan rekomendasi:** Membandingkan hasil ARIMA dan LSTM dan memberikan rekomendasi.

### 5.2. Cara Kerja ARIMA, Versi Sederhana

ARIMA adalah metode klasik untuk data deret waktu.

Cara berpikirnya:

1. Lihat apakah data sudah stasioner. Kalau belum, lakukan differencing.
2. Lihat pola autokorelasi dengan ACF dan PACF untuk menentukan berapa lag yang dipakai.
3. Buat model dengan kombinasi p, d, dan q yang paling masuk akal.
4. Ramalkan dan hitung error-nya.

Kelebihan ARIMA: Mudah dijelaskan, cepat, dan bagus untuk data yang polanya linear dan stabil.

Kekurangan ARIMA: Sulit menangkap pola yang tidak linear dan lonjakan tajam yang sering muncul di data penjualan obat.

### 5.3. Cara Kerja LSTM, Versi Sederhana

LSTM adalah jenis Recurrent Neural Network yang dirancang untuk mengingat pola jangka panjang.

Cara berpikirnya:

1. Ubah data deret waktu menjadi urutan. Contohnya, pakai 7 hari sebelumnya untuk meramalkan hari berikutnya.
2. Beri model urutan tersebut dan biarkan model belajar pola mana yang penting dan pola mana yang bisa dilupakan.
3. Model akan belajar bahwa lonjakan kemarin mungkin berpengaruh ke hari ini, tapi lonjakan 2 tahun lalu mungkin sudah tidak relevan.
4. Setelah dilatih, model dipakai untuk meramalkan data uji.

Kelebihan LSTM: Mampu menangkap pola tidak linear dan ketergantungan jangka panjang.

Kekurangan LSTM: Butuh data lebih banyak, butuh tuning, dan lebih sulit dijelaskan ke orang non teknis daripada ARIMA.

---

## 6. Hasil Penelitian

### 6.1. Hasil Kategori Generik

| Metode | RMSE | MAE | MAPE |
| --- | --- | --- | --- |
| ARIMA | 13,99 | 9,79 | 4,35 persen |
| LSTM | 1,91 | 1,51 | 0,64 persen |

Interpretasi:
- LSTM error-nya jauh lebih kecil di semua metrik.
- MAPE LSTM 0,64 persen artinya rata-rata selisih peramalan terhadap nilai asli sangat kecil untuk kategori Generik.
- ARIMA masih cukup baik dengan MAPE 4,35 persen, tapi kalah jauh dari LSTM untuk data ini.

### 6.2. Hasil Kategori Non Generik

| Metode | RMSE | MAE | MAPE |
| --- | --- | --- | --- |
| ARIMA | 2,02 | 1,28 | 16,46 persen |
| LSTM | 0,34 | 0,27 | 2,59 persen |

Interpretasi:
- Di kategori Non Generik, ARIMA MAPE-nya melonjak sampai 16,46 persen. Artinya ARIMA kesulitan menangkap fluktuasi kecil yang tajam.
- LSTM tetap stabil dengan MAPE 2,59 persen.
- Perbedaan ini menunjukkan bahwa volume kecil tidak berarti mudah diramalkan. Justru fluktuasinya bisa lebih liar.

### 6.3. Apa Artinya untuk Rumah Sakit

1. Untuk perencanaan stok obat Generik, LSTM bisa dipakai sebagai acuan utama karena error-nya kecil.
2. Untuk obat Non Generik, LSTM juga lebih baik, tapi tetap perlu pengawasan manusia karena volume kecil dan mudah terpengaruh kejadian mendadak.
3. ARIMA tetap bisa dipakai sebagai pembanding dan sebagai baseline. Kalau LSTM error-nya tiba-tiba membesar, ARIMA bisa jadi alarm bahwa ada perubahan pola.
4. Peramalan bukan pengganti keputusan manusia. Peramalan adalah alat bantu. Keputusan akhir tetap harus mempertimbangkan faktor yang tidak ada di data, seperti kebijakan baru, wabah, dan perubahan supplier.

---

## 7. Visualisasi Hasil

Di laporan asli, visualisasi yang dipakai:

1. **Grafik data asli vs hasil ramalan ARIMA:** Menunjukkan di mana ARIMA mulai meleset saat ada lonjakan.
2. **Grafik data asli vs hasil ramalan LSTM:** Menunjukkan LSTM lebih mengikuti pola asli, termasuk saat ada lonjakan.
3. **Grafik perbandingan error:** Menunjukkan perbedaan RMSE, MAE, dan MAPE secara visual.
4. **Tabel hasil peramalan beberapa periode ke depan:** Menunjukkan contoh angka ramalan yang bisa dipakai untuk perencanaan.

Di repo ini, yang dibagikan adalah screenshot grafik yang sudah disensor dari data sensitif. Grafik lengkap ada di laporan versi publik.

---

## 8. Teknologi dan Tools yang Dipakai

- **Bahasa:** Python
- **Library utama:** Untuk ARIMA, library statistik deret waktu. Untuk LSTM, library deep learning berbasis TensorFlow atau Keras.
- **Pengolahan data:** Pandas dan NumPy untuk agregasi dan preprocessing.
- **Visualisasi:** Matplotlib untuk grafik.
- **Evaluasi:** Perhitungan manual RMSE, MAE, dan MAPE dari hasil prediksi dan nilai asli.

Penelitian ini tidak memakai framework web. Fokusnya adalah pemodelan dan evaluasi, bukan pembuatan aplikasi web. Aplikasi web yang memakai hasil model ini bisa dibuat terpisah.

---

## 9. Cara Berpikir Data di Penelitian Ini

Ini bagian yang paling penting buat yang baca repo ini sebagai portfolio.

Sebelum modeling, pertanyaan yang dipakai:

1. **Data apa yang dipakai:** Apakah ini data pasien atau data penjualan agregat. Jawabannya data penjualan agregat, bukan data pasien.
2. **Data apa yang tidak boleh dipakai:** Data yang berisi identitas pasien tidak boleh dipakai dan tidak diminta.
3. **Apa yang mau diramalkan:** Penjualan per hari per kategori, bukan per pasien.
4. **Bagaimana cara mengukur benar atau salah:** Pakai RMSE, MAE, dan MAPE, bukan sekadar feeling.
5. **Apa yang bisa salah:** Kalau data ada yang hilang, kalau ada lonjakan karena kejadian luar biasa, model bisa salah. Karena itu, hasil harus dibaca dengan konteks.

Setelah modeling, pertanyaan tambahannya:

1. **Apakah model ini overfitting:** Apakah model ini bagus di data latih tapi jelek di data uji. Karena itu, evaluasi harus di data uji yang belum pernah dilihat model.
2. **Apakah hasilnya masuk akal:** Kalau model meramalkan penjualan negatif, itu jelas salah dan harus diperbaiki.
3. **Apakah hasilnya bisa dipakai:** Kalau error-nya kecil tapi modelnya tidak bisa dijelaskan ke atasan, model itu sulit dipakai di dunia nyata.

Cara berpikir ini yang membedakan penelitian yang rapi dengan penelitian yang asal jadi.

---

## 10. Keterbatasan Penelitian

1. Data hanya dari satu rumah sakit dan satu periode. Hasilnya belum tentu sama di rumah sakit lain.
2. Model hanya memakai data historis penjualan. Faktor luar seperti kebijakan, wabah, dan perubahan supplier tidak dimasukkan.
3. LSTM butuh tuning. Hasil yang didapat adalah hasil dari kombinasi yang dicoba di penelitian ini, bukan hasil terbaik yang mungkin ada.
4. Penelitian ini tidak membahas implementasi model ke sistem gudang secara penuh. Implementasi ke sistem operasional adalah pekerjaan lanjutan.

Menulis keterbatasan itu bukan kelemahan. Itu tanda penelitian dilakukan dengan jujur.

---

## 11. Struktur Laporan di Repo Ini

```
peramalan-obat-arima-lstm/
  README.md
  docs/
    SKRIPSI-PERAMALAN-OBAT-Versi-Publik.pdf
  screenshots/
    01-grafik-generik-arima.png
    02-grafik-generik-lstm.png
    03-grafik-nongenerik-lstm.png
    04-tabel-perbandingan.png
  data-contoh/
    contoh-data-agregat-generik.csv
    contoh-data-agregat-nongenerik.csv
```

Aturan:
- File PDF di folder docs adalah versi publik yang sudah dihapus lampiran surat izin dan bukti pembayaran.
- File data contoh hanya berisi 10 sampai 20 baris data agregat yang sudah disensor. File data lengkap tidak dibagikan ke publik.
- File screenshot tidak boleh berisi data pasien atau data yang bisa mengidentifikasi individu.

---

## 12. Pelajaran yang Didapat

1. Metode yang lebih canggih tidak otomatis lebih baik. Harus dibuktikan dengan metrik yang sama.
2. Memisahkan data berdasarkan perilaku itu penting. Generik dan Non Generik tidak bisa disamakan.
3. Preprocessing itu separuh pekerjaan. Kalau data agregatnya salah, model sebagus apa pun akan salah.
4. Visualisasi itu bukan hiasan. Grafik yang jelas bisa menjelaskan lebih cepat daripada 3 paragraf.
5. Etika data itu bukan tambahan. Meminta izin, tidak mengambil data pasien, dan tidak membagikan data mentah ke publik adalah bagian dari penelitian.

---

## 13. Catatan Privasi dan Etika Data

- Penelitian ini memakai data penjualan obat agregat, bukan data pasien.
- Izin pengambilan data sudah didapatkan dari RSUD Arifin Achmad sebelum penelitian dimulai.
- Surat izin dan bukti pembayaran tidak dibagikan ke publik di repo ini.
- Data mentah lengkap tidak dibagikan ke publik.
- Jika ada yang menemukan data sensitif di repo ini, mohon hubungi pemilik repo agar segera dihapus.

---

## 14. Kontak

- Nama: Muhammad Sulthan Muqsith Afif
- Peran: Junior Web Developer, fokus Laravel dan React, dengan pengalaman penelitian data time series
- Pendidikan: S1 Sistem Informasi, Universitas Riau
- GitHub: https://github.com/sulthanafif
- LinkedIn: https://www.linkedin.com/in/sulthanafif

Jika ada yang ingin diskusi soal ARIMA, LSTM, atau peramalan penjualan obat, silakan hubungi lewat GitHub atau LinkedIn. Saya lebih nyaman diskusi lewat tulisan dan dokumen, bukan telepon dadakan.
