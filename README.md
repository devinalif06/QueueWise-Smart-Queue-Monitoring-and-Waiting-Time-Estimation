# QueueWise — Smart Queue Monitoring & Waiting Time Estimation

Sistem monitoring antrean berbasis Computer Vision yang mendeteksi orang, melacak pergerakannya, dan menghitung jumlah antrean serta estimasi waktu tunggu secara otomatis dari video.

**Mata kuliah:** Teknologi Berkembang
**Tugas:** Week 8 — Progress Submission
**Kelompok:** _5_
**Anggota:**
- _Devin Alif Fansyah Yahya – 5026241042_
- _Axellino Berwin Parlindungan Sinaga  – 5026241094_
- _Bina Ilyas Santosa - 5026241074_
- _Faiz Willy Abiyyu - 5026241166_
- _Muhammad Rafif Sabilillah - 5026241168_



## Alur Sistem

Skenario: satu loket, antrean FIFO (first-in-first-out).

```
Video -> YOLO11n (deteksi orang) -> ByteTrack (ID per orang) -> titik kaki
      -> lewat ENTRY LINE  = masuk antrean (+1)
      -> lewat EXIT LINE   = selesai dilayani (-1), waktu tunggu dicatat
      -> antrean = masuk - keluar - hilang
      -> estimasi tunggu  = antrean x selang rata-rata antar-keluar
      -> overlay dashboard + video output + log CSV
```

- **Deteksi**: YOLO11n pretrained (COCO), hanya kelas `person` — tanpa training ulang.
- **Tracking**: ByteTrack (via `model.track(..., persist=True)`), memberi ID unik per orang antar-frame.
- **Logika crossing**: titik kaki (tengah sisi bawah bounding box) dibandingkan posisinya terhadap dua garis virtual (entry & exit) menggunakan cross product, dengan deadband agar getaran bounding box tidak memicu penghitungan ganda.
- **Estimasi waktu tunggu**: `jumlah orang di antrean × rata-rata selang waktu antar-keluar` (butuh ≥ 2 orang keluar; sebelum itu status "Kalibrasi").


## Cara Menjalankan

1. Clone repo ini dan masuk ke foldernya.
2. Buat environment baru (opsional tapi disarankan):
   ```bash
   python -m venv venv
   source venv/bin/activate   # Windows: venv\Scripts\activate
   ```
3. Install dependencies:
   ```bash
   pip install -r requirements.txt
   ```
4. Jalankan notebook (`Source_Code/QueueWise_notebook.ipynb`) dengan **Kernel → Restart & Run All**.
   - Video contoh (`vtest.avi`, dari dataset resmi OpenCV) akan terunduh otomatis bila belum ada.
   - Model `yolo11n.pt` juga diunduh otomatis oleh Ultralytics saat notebook dijalankan pertama kali.
5. Hasil (video overlay, log CSV, screenshot event) akan tersimpan di folder `outputs/`.

> Notebook ini **berdiri sendiri** — tidak butuh file konfigurasi atau skrip tambahan di luar notebook. Semua parameter yang perlu diubah untuk video lain ada di sel **"2. Konfigurasi"**.

## Konfigurasi Utama

| Parameter | Keterangan |
|---|---|
| `VIDEO_PATH` | Path video input (ganti dengan video antrean sendiri, atau `0` untuk webcam) |
| `MODEL_NAME` | Nama model YOLO (`yolo11n.pt`, diunduh otomatis) |
| `CONF` | Ambang confidence deteksi |
| `ENTRY_LINE` / `EXIT_LINE` | Koordinat dua garis virtual (masuk & keluar antrean) |
| `DEADBAND_PX` | Pita mati di sekitar garis agar getaran bbox tidak memicu crossing palsu |
| `LOST_TIMEOUT_S` | Batas waktu seseorang dianggap hilang dari kamera |
| `MAX_FRAMES` | Batasi jumlah frame untuk uji cepat (`None` = seluruh video) |

## Evaluasi

Notebook mencakup:
1. Unit test sintetis untuk logika crossing (bagian 6, lintasan buatan, bukan hasil YOLO).
2. Pengukuran FPS proses end-to-end di perangkat yang menjalankan.
3. Perbandingan estimasi waktu tunggu vs waktu tunggu aktual (MAE) untuk setiap orang yang masuk dan keluar.
4. Perbandingan hasil counting vs ground truth manual (perlu diisi manual setelah menonton video — lihat variabel `GT_ENTRIES` / `GT_EXITS`).

## Keterbatasan & Rencana Week 9

- `vtest.avi` adalah video pejalan kaki umum (dataset resmi OpenCV), bukan antrean sungguhan — "waktu tunggu" yang terukur sebenarnya waktu jalan di antara dua garis.
- ID switch dari ByteTrack dapat membuat orang yang sama keluar dengan ID baru, sehingga exit diabaikan dan ID lama berstatus LOST.
- Orang yang sudah berada di antara dua garis saat video mulai tidak terhitung masuk.
- Estimasi waktu tunggu masih model sederhana (satu loket, FIFO, selang keluar rata-rata).

Rencana Week 9: merekam video antrean nyata (dengan izin pihak terkait), mengatur ulang koordinat garis sesuai lokasi, mengisi ground truth manual, membandingkan ByteTrack vs BoT-SORT, dan (opsional) menampilkan dashboard dengan Streamlit.

## Dependencies

Lihat [`requirements.txt`](requirements.txt). Library utama: `ultralytics` (YOLO11n + ByteTrack), `opencv-python`, `pandas`, `matplotlib`, `numpy`.

