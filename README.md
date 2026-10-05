# Olist E-Commerce: Analisis Pengiriman, Kepuasan, dan Retensi Pelanggan

**Final Project Kelompok Alpha · Purwadhika**

Proyek ini menganalisis hubungan ketepatan waktu pengiriman dengan rating pelanggan dan pembelian ulang pada data historis Olist. Analisis mencakup persiapan data relasional, eksplorasi ulasan, evaluasi wilayah dan seller, cohort retention, serta penyusunan rekomendasi bisnis.

**[Buka notebook analisis](Final_Project_Alpha.ipynb)**

## Ringkasan hasil

Dari **96.446 pesanan delivered yang lolos cleaning**, tingkat keterlambatan mencapai **6,77%**, dengan rata-rata keterlambatan **10,6 hari** pada pesanan yang terlambat. Kelompok terlambat memiliki rata-rata rating lebih rendah, sementara tingkat keterlambatan berbeda antarwilayah dan periode transaksi.

| Indikator | Hasil |
|---|---:|
| Pesanan dalam master table | 96.446 |
| Pelanggan unik | 93.329 |
| Tingkat keterlambatan | 6,77% |
| Rata-rata rating: tepat waktu / terlambat | 4,29 / 2,27 |
| Proporsi bintang 1: tepat waktu / terlambat | 6,61% / 53,78% |
| GMV pesanan delivered | R$13,22 juta |
| Pelanggan dengan lebih dari satu pesanan | 2.798 atau sekitar 3,0% |
| Kontribusi repeat buyer terhadap GMV | 5,5% |
| Repeat rate: pesanan pertama tepat waktu / terlambat | 3,04% / 2,49% |

Perbandingan rating menggunakan pesanan yang memiliki skor ulasan. Repeat rate dihitung dari transaksi delivered yang teramati dalam periode dataset. Seluruh nilai uang menggunakan **real Brasil (BRL/R$)**; GMV adalah nilai barang, bukan laba atau pendapatan bersih Olist.

## Pertanyaan bisnis

1. Bagaimana perbedaan rating antara pesanan tepat waktu dan terlambat, serta kosakata apa yang sering muncul dalam ulasan?
2. Wilayah dan seller mana yang perlu diprioritaskan untuk evaluasi keterlambatan?
3. Bagaimana kontribusi GMV pelanggan one-time dan repeat, serta hubungan keterlambatan pesanan pertama dengan pembelian ulang?
4. Pada periode mana tingkat keterlambatan meningkat?

Hasil analisis ditujukan untuk **Operations & Logistics**, **Customer Experience**, **Seller Management**, dan manajemen strategis.

## Dataset

Sumber: **[Brazilian E-Commerce Public Dataset by Olist — Kaggle](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce)**, dengan data transaksi tahun **2016–2018**.

Notebook memuat delapan file berikut. Empat tabel utama membentuk basis analisis; tabel pendukung digunakan dalam pemeriksaan awal data.

| File | Peran |
|---|---|
| `olist_orders_dataset.csv` | Status pesanan dan timestamp pengiriman |
| `olist_customers_dataset.csv` | Identitas pelanggan dan wilayah tujuan |
| `olist_order_items_dataset.csv` | Harga barang, seller, dan batas penyerahan |
| `olist_order_reviews_dataset.csv` | Rating dan teks ulasan |
| `olist_order_payments_dataset.csv` | Data pembayaran pendukung |
| `olist_products_dataset.csv` | Data produk pendukung |
| `olist_sellers_dataset.csv` | Data seller pendukung |
| `product_category_name_translation.csv` | Terjemahan kategori produk |

Tabel geolocation ditampilkan dalam skema dataset, tetapi tidak dibutuhkan oleh proses loading notebook ini. Analisis wilayah menggunakan `customer_state`, tanpa perhitungan jarak koordinat. Unduh data dari sumber di atas dan ikuti ketentuan penggunaan yang tercantum pada halaman dataset.

## Persiapan data dan definisi metrik

- Memeriksa duplikasi, missing value, tipe data, dan outlier.
- Membatasi populasi ke pesanan `delivered` dengan timestamp penyerahan dan penerimaan yang lengkap.
- Menghapus kasus tanggal penerimaan mendahului penyerahan ke kurir.
- Memilih satu ulasan terbaru per `order_id`, berdasarkan urutan timestamp jawaban dan pembuatan ulasan.
- Mengagregasikan item sebelum merge agar master table memiliki **satu baris per pesanan**.
- Memvalidasi keunikan `order_id`, jumlah baris, serta kesesuaian total GMV sebelum dan sesudah merge.

| Metrik | Definisi |
|---|---|
| `is_late` | Tanggal diterima pelanggan melewati tanggal estimasi; perbandingan pada level tanggal |
| `delay_days` | Selisih tanggal penerimaan dan estimasi; nilai positif berarti terlambat |
| `order_gmv` | Jumlah `price` seluruh item dalam pesanan, tanpa ongkos kirim |
| Repeat buyer | `customer_unique_id` dengan lebih dari satu pesanan dalam master table |
| Cohort | Kelompok pelanggan menurut bulan transaksi pertama yang teramati dalam master table |
| Retensi cohort | Pelanggan aktif pada bulan relatif tertentu dibagi jumlah pelanggan pada bulan akuisisi |

## Alur analisis

| Bagian | Analisis dan metode |
|---|---|
| Sesi 1 — Kepuasan pelanggan | Rata-rata rating, komposisi bintang, dan Welch's independent t-test (`equal_var=False`) |
| Sesi 1 Lanjutan — Ulasan | Cakupan komentar, word count, bigram, dan pengelompokan berdasarkan rating |
| Sesi 2 — Logistik | Kategori keterlambatan berdasarkan timestamp, perbandingan wilayah, evaluasi seller, dan uji Chi-Square |
| Sesi 3 — GMV dan retensi | Cohort heatmap, segmentasi one-time/repeat buyer, kontribusi GMV, dan uji Chi-Square untuk keterlambatan pesanan pertama versus repeat order |
| Sesi 4 — Tren waktu | Volume transaksi dan late rate bulanan |
| Rekomendasi bisnis | Prioritas stakeholder, target SMART, dan skenario cost-benefit |

Versi notebook ini juga memuat eksplorasi tema keluhan berbasis regex. Hasil pencocokan tersebut bersifat indikatif dan membutuhkan pembacaan konteks komentar sebelum digunakan sebagai kesimpulan tema atau akar masalah.

## Temuan utama

- **Rating berbeda menurut status pengiriman.** Rata-rata rating sebesar 4,29 pada kelompok tepat waktu dan 2,27 pada kelompok terlambat. Welch's t-test menunjukkan perbedaan yang signifikan pada ambang 5%.
- **Risiko keterlambatan tidak merata antarwilayah.** Alagoas mencatat late rate sekitar 21,4%, Maranhão 17,4%, dan Sergipe 15,2%. Rio de Janeiro perlu diperhatikan karena late rate sekitar 12,1% disertai volume 12.350 pesanan.
- **Repeat buyer berjumlah kecil dalam data yang teramati.** Sekitar 3,0% pelanggan melakukan lebih dari satu transaksi. Rata-rata GMV kumulatif per repeat buyer mencapai R$259,97, dibandingkan R$137,95 pada one-time buyer.
- **Keterlambatan pesanan pertama berkaitan dengan repeat rate.** Repeat rate sebesar 3,04% pada kelompok pertama tepat waktu dan 2,49% pada kelompok pertama terlambat; uji Chi-Square menghasilkan p-value 0,0149.
- **Terdapat konsentrasi keterlambatan pada periode tertentu.** Late rate mencapai 18,96% pada Maret 2018, 14,13% pada Februari 2018, dan 12,40% pada November 2017.
- **Ketersediaan komentar berbeda menurut rating.** Sebanyak 76,0% ulasan rating 1–2 menyertakan teks, dibandingkan 36,7% pada rating 4–5. Analisis teks perlu memperhitungkan perbedaan partisipasi ini.

## Arah rekomendasi

| Stakeholder | Fokus usulan |
|---|---|
| Operations & Logistics | Evaluasi SLA dan proses pengiriman pada wilayah prioritas; perencanaan kapasitas berdasarkan pola historis |
| Customer Experience | Pilot notifikasi keterlambatan dan kompensasi tertarget, lalu evaluasi hasil pembelian ulang |
| Seller Management | Pembinaan seller dengan tingkat keterlambatan penyerahan tinggi |
| Manajemen | Evaluasi alokasi anggaran akuisisi dan retensi berdasarkan hasil pilot |

Target yang diusulkan dalam notebook adalah menurunkan late rate dari **6,77% menjadi ≤5,75%** dan meningkatkan repeat rate dari sekitar **3,00% menjadi ≥3,15%** dalam 12 bulan. Target tersebut masing-masing mewakili perubahan sekitar **15% relatif** dan **5% relatif**, bukan perubahan 15 atau 5 poin persentase.

Target dan estimasi cost-benefit merupakan **skenario usulan**, bukan hasil intervensi yang sudah dicapai.

## Tools

- **Python** dan **Jupyter Notebook / Google Colab**
- **pandas** dan **NumPy** untuk pengolahan data
- **Matplotlib** dan **Seaborn** untuk visualisasi
- **SciPy** untuk uji statistik
- **re** dan **collections.Counter** untuk pemrosesan serta penghitungan kata

## Cara menjalankan

### Google Colab

1. Unduh `Final_Project_Alpha.ipynb` dari repository ini, lalu buka melalui menu upload notebook di [Google Colab](https://colab.research.google.com/).
2. Unduh dataset Olist dan simpan delapan CSV yang disebutkan di atas dalam satu folder Google Drive.
3. Jalankan cell mount Google Drive dan berikan akses ke akun yang menyimpan dataset.
4. Sesuaikan variabel `DATA_DIR` dengan lokasi folder. Path yang digunakan notebook saat ini:

   ```python
   DATA_DIR = "/content/drive/MyDrive/Final Project Alpha/Dataset Olist"
   ```

5. Jalankan seluruh cell secara berurutan dari atas. Jika ada dependency yang belum tersedia, pasang `pandas`, `numpy`, `matplotlib`, `seaborn`, dan `scipy` terlebih dahulu.

### Jupyter lokal

1. Siapkan lingkungan Python dan pasang dependency:

   ```bash
   python -m pip install pandas numpy matplotlib seaborn scipy jupyterlab
   ```

2. Simpan CSV dalam folder lokal, misalnya `data/` di samping notebook.
3. Lewati cell `from google.colab import drive` dan `drive.mount(...)`, lalu ubah:

   ```python
   DATA_DIR = "./data"
   ```

4. Buka notebook dari direktori proyek:

   ```bash
   jupyter lab Final_Project_Alpha.ipynb
   ```

5. Jalankan seluruh cell dari atas. Tautan daftar isi dengan format `#scrollTo=...` dirancang untuk Colab; untuk navigasi lokal gunakan daftar heading pada JupyterLab.

## Batas interpretasi

- Analisis menggunakan data observasional historis. Perbedaan rating dan asosiasi repeat order belum membuktikan hubungan sebab-akibat.
- Populasi utama hanya mencakup pesanan delivered yang lolos cleaning. Hasil tidak langsung mewakili pesanan dibatalkan atau tidak terkirim.
- Tidak adanya pembelian ulang selama periode pengamatan tidak memastikan pelanggan churn permanen. Pelanggan yang baru masuk memiliki waktu observasi lebih pendek.
- Kategori seller/kurir merupakan aturan berbasis timestamp, bukan verifikasi tanggung jawab operasional. Penggabungan pesanan multi-seller dan pemakaian batas penyerahan maksimum membatasi ketepatan atribusi.
- Label sentimen berasal dari skor bintang. Word count, bigram setelah stopword removal, dan regex tidak memahami konteks kalimat secara penuh; waktu penulisan ulasan juga dapat mendahului penerimaan barang.
- GMV kumulatif per pelanggan selama periode pengamatan bukan estimasi lifetime value penuh. Dataset tidak menyediakan seluruh biaya dan margin untuk menyimpulkan laba, ROI aktual, atau manfaat kausal intervensi.

## Berkas utama

| Berkas | Isi |
|---|---|
| [Final_Project_Alpha.ipynb](Final_Project_Alpha.ipynb) | Kode, visualisasi, interpretasi, dan rekomendasi lengkap |
| [README.md](README.md) | Ringkasan proyek dan panduan menjalankan notebook |

## Kredit

Analisis disusun oleh **Kelompok Alpha** sebagai final project Purwadhika. Data disediakan oleh **Olist** melalui [Kaggle](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce). Proyek ini merupakan studi analisis data untuk pembelajaran dan portofolio, bukan laporan resmi Olist.
