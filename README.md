# E-Commerce Analysis

## 📌 Deskripsi Proyek

Menggunakan **Power BI Desktop**, proyek ini mensimulasikan peran seorang *BI Developer* dalam mengubah data mentah yang padat menjadi sebuah *Executive Dashboard* interaktif guna mendukung pengambilan keputusan strategis oleh manajemen puncak (*C-Level Executive*).

---

## 🏗️ Arsitektur Data & Pemodelan (Star Schema)
Proyek ini meninggalkan pendekatan tabel tunggal konvensional (*flat table*) dan mengimplementasikan prinsip **Star Schema** untuk menjamin efisiensi memori, optimalisasi penyimpanan, dan kecepatan kalkulasi kueri di Power BI:

* **Fact Table (`Fact_Sales`):** Berada di pusat skema, menyimpan data inti transaksi numerik seperti `Quantity`, `UnitPrice`, `InvoiceNo`, dan `InvoiceDate`, serta kunci relasi (*foreign keys*).
* **Dimension Tables (`Dim_Product`):** Berisi data master produk unik yang dibersihkan dari duplikasi, menyimpan atribut `StockCode` dan `Description`.
* **Dimension Tables (`Dim_Customer`):** Berisi data master pelanggan unik yang menyimpan atribut `CustomerID` dan wilayah asal `Country`.

> **Relasi Data:** Kedua tabel dimensi dihubungkan ke tabel fakta menggunakan relasi **One-to-Many ($1:\infty$)** melalui kolom kunci `StockCode` dan `CustomerID` dengan arah filter tunggal (*Single Directional Filter*).

---

## 🛠️ Proses ETL & Pembersihan Data (Power Query)
Sebelum dimuat ke dalam model data, data mentah diekstraksi dan dibersihkan melalui **Power Query Editor** dengan langkah-langkah berikut:
1. **Normalisasi Tabel:** Melakukan teknik duplikasi dan eliminasi kolom untuk memecah dataset tunggal menjadi 3 struktur tabel terpisah (Star Schema).
2. **Koreksi Tipe Data Kritis:** 
   * Mengubah tipe data `UnitPrice` menjadi *Fixed Decimal Number (Currency)* agar nilai desimal produk presisi dan tidak terbulat otomatis.
   * Mengubah tipe data `CustomerID` dan `InvoiceNo` menjadi *Text* untuk menghemat memori kalkulasi, mengingat kolom ID tidak memerlukan operasi aritmatika (seperti penjumlahan).
3. **Data Deduplication:** Menerapkan fungsi *Remove Duplicates* pada kolom kunci di tabel dimensi (`Dim_Product[StockCode]` dan `Dim_Customer[CustomerID]`) untuk menjamin integritas referensial data.

---

## 🧠 Analisis Formula Bisnis (Advanced DAX Measures)
Seluruh metrik finansial dan operasional dihitung secara dinamis menggunakan ekspresi DAX (*Data Analysis Expressions*) kustom yang disimpan di dalam *dedicated table* (`_All Measures`):

### 1. Total Revenue (Pendapatan Kotor)
Menghitung total omset dengan melakukan iterasi perkalian antara kuantitas dan harga satuan di setiap baris pada tabel fakta, kemudian menjumlahkan totalnya.
$$Total\ Revenue = SUMX(Fact\_Sales, Fact\_Sales[Quantity] * Fact\_Sales[UnitPrice])$$

### 2. Total Quantity (Total Unit Terjual)
Menghitung akumulasi volume produk yang berhasil didistribusikan ke pasar.
$$Total\ Quantity = SUM(Fact\_Sales[Quantity])$$

### 3. Total Orders (Total Transaksi Unik)
Menghitung jumlah pesanan unik berdasarkan nomor invoice tanpa terjadi *double counting* akibat variasi item dalam satu struk.
$$Total\ Orders = DISTINCTCOUNT(Fact\_Sales[InvoiceNo])$$

---

Dashboard ini dirancang dengan pendekatan *clean and functional UI/UX layout* yang dibagi menjadi beberapa zona:
* **KPI Ribbon Header:** Menampilkan metrik utama bisnis secara instan menggunakan komponen *Card* untuk menyajikan **Total Revenue (911M)**, **Total Quantity (5M)**, dan **Total Orders (26K)**.
* **Geographical Distribution (Donut Chart):** Menganalisis kontribusi pendapatan berdasarkan negara asal konsumen (`Dim_Customer[Country]`), menonjolkan dominasi pasar tertentu secara visual.
* **Top 10 High-Value Products (Clustered Bar Chart):** Menggunakan fitur **Top N Filter** yang dikombinasikan dengan ukuran `Total Revenue` untuk memetakan 10 produk utama yang menjadi mesin pencetak profit tertinggi.
* **Macro Time-Series Trend (Line Chart):** Menyajikan pergerakan naik-turun omset bulanan menggunakan hierarki tanggal (`Fact_Sales[InvoiceDate]`) untuk mendeteksi pola musiman penjualan.

---

## 💡 Insight Utama Bisnis
* **Cross-Filtering Power:** Berkat arsitektur Star Schema yang matang, seluruh visualisasi bersifat interaktif. Pengguna dapat mengeklik satu negara pada *Donut Chart* untuk langsung mengubah seluruh angka KPI, grafik tren bulanan, dan daftar Top 10 produk khusus untuk negara tersebut secara *real-time*.
* **Skalabilitas Data:** Penggunaan formula DAX `SUMX` terbukti mampu mengeksekusi perhitungan matriks kompleks pada database berukuran setengah juta baris dengan sangat optimal tanpa penurunan performa (*latency*).

---
