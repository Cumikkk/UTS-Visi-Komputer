# Segmentasi Citra Daun Tomat
### UTS — Mata Kuliah Visi Komputer

Proyek ini mengimplementasikan pipeline segmentasi citra daun tomat menggunakan metode **Color Thresholding berbasis HSV**, dikombinasikan dengan **GrabCut**, **Operasi Morfologi**, dan **Contour Filtering**. Implementasi mengacu pada paper *Azli et al., Jurnal Informatika Poltekharber, 2025*.

---

## Tujuan

Mendeteksi dan mengisolasi area daun tomat dari citra, termasuk daun yang terinfeksi penyakit, dengan meminimalkan noise dari background dan objek non-daun (seperti tomat itu sendiri).

---

## Dataset

Tiga kategori citra digunakan sebagai input:

| Label | File | Kondisi Daun |
|---|---|---|
| Daun Sehat | `sehat.webp` | Daun normal, dominan hijau |
| Septoria Leaf | `septoria.webp` | Daun terinfeksi Septoria, ada bercak kuning |
| Early Blight | `early_blight.jpeg` | Daun terinfeksi Early Blight, warna gelap kebiruan |

Semua gambar di-resize ke ukuran seragam **300×300 piksel** sebelum diproses.

---

## Teknologi & Library

- **Python 3**
- **OpenCV (`cv2`)** — preprocessing, GrabCut, color thresholding, morfologi, contour detection
- **NumPy** — manipulasi array dan masking
- **Matplotlib** — visualisasi hasil segmentasi

---

## Alur Pipeline Segmentasi

Pipeline terdiri dari 9 langkah utama:

```
Step 1  →  GrabCut            : Estimasi awal ROI foreground (area daun)
Step 2  →  Exclude Bright BG  : Buang background putih/terang (S rendah + V tinggi)
Step 3  →  HSV Thresholding   : Mask warna hijau + kuning-coklat + abu-hijau
Step 4  →  Combine Mask       : GrabCut × HSV × exclude-bright (+ fallback jika terlalu kosong)
Step 5  →  Opening            : Buang noise kecil di luar daun (kernel ellipse 5×5, iter=2)
Step 6  →  Closing            : Tutup lubang di dalam daun (kernel ellipse 11×11, iter=3)
Step 7  →  Shape Filter       : Buang objek bulat (tomat) berdasarkan solidity & aspect ratio
Step 8  →  Largest Contour    : Ambil kontur terbesar sebagai objek daun utama
Step 9  →  Bitwise Masking    : Terapkan mask final ke citra asli
```

### Rentang HSV yang Digunakan

| Warna | Lower Bound | Upper Bound | Tujuan |
|---|---|---|---|
| Hijau | `[20, 20, 20]` | `[95, 255, 255]` | Daun sehat & sebagian besar area daun |
| Kuning-coklat | `[8, 20, 40]` | `[28, 255, 255]` | Area penyakit, daun menguning |
| Abu-hijau | `[85, 10, 40]` | `[140, 80, 180]` | Daun Early Blight (gelap kebiruan) |

---

## Cara Menjalankan

### 1. Clone atau download notebook ini

```bash
git clone <url-repo>
```

### 2. Upload gambar ke Google Colab

Upload tiga file gambar ke direktori `/content/` di Colab:
- `sehat.webp`
- `septoria.webp`
- `early_blight.jpeg`

### 3. Jalankan semua sel secara berurutan

Pastikan library berikut tersedia (sudah tersedia secara default di Google Colab):

```bash
pip install opencv-python numpy matplotlib
```

### 4. Output

Setelah semua sel berjalan, dua file PNG akan tersimpan di direktori aktif:

| File | Isi |
|---|---|
| `hasil_segmentasi_daun_tomat.png` | Grid 3×5: perbandingan citra asli, channel Hue, mask sebelum morfologi, mask setelah morfologi, dan hasil segmentasi akhir untuk ketiga kategori daun |
| `detail_morfologi.png` | Detail 4 tahap morfologi khusus untuk gambar Early Blight |

---

## Output & Analisis Kuantitatif

Notebook mencetak tabel analisis coverage untuk setiap gambar:

```
============================================================
ANALISIS KUANTITATIF HASIL SEGMENTASI
============================================================
Label                Area Mask (px)     Total Pixel     Coverage (%)
------------------------------------------------------------
Daun Sehat           ...                90000           ...%
Septoria Leaf        ...                90000           ...%
Early Blight         ...                90000           ...%
============================================================
```

- **Area Mask** : jumlah piksel yang terdeteksi sebagai daun
- **Total Pixel** : total piksel gambar (300×300 = 90.000)
- **Coverage (%)** : persentase area daun terhadap keseluruhan gambar

---

## Catatan Teknis

- **GrabCut** menggunakan margin dinamis sebesar 8% sisi terpendek gambar (minimum 10px) untuk menghindari clipping pada gambar kecil.
- **Fallback HSV** aktif jika hasil kombinasi GrabCut × HSV menghasilkan coverage < 5% — kondisi ini bisa terjadi pada gambar dengan pencahayaan tidak standar.
- **Filter objek bulat**: objek dengan solidity > 0.88 *dan* aspect ratio antara 0.75–1.25 dianggap tomat dan dibuang dari mask.
- Range HSV hijau sengaja diperluas (`H: 20–95`) untuk mengakomodasi variasi pencahayaan di lapangan.

---

## Referensi

> Azli et al., *Segmentasi Citra Daun Tomat Berbasis Warna*, Jurnal Informatika Poltekharber, 2025.
