# Hand Gesture Recognition and Face Detection with MediaPipe

Implementasi *pipeline* visi komputer menggunakan model pra-latih **MediaPipe Hand Landmarker** dan **MediaPipe Face Detector (BlazeFace)** untuk klasifikasi gestur tangan berbasis aturan geometris (*rule-based*) serta deteksi wajah pada citra sintetis. Proyek ini dikerjakan untuk mata kuliah **CIF61339 — Pengolahan Citra Digital dan Visi Komputer** di Fakultas Ilmu Komputer, Universitas Brawijaya.

## Daftar Isi
1. [Ringkasan Proyek](#ringkasan-proyek)
2. [Metodologi dan Arsitektur Pipeline](#metodologi-dan-arsitektur-pipeline)
3. [Hasil Eksperimen dan Evaluasi](#hasil-eksperimen-dan-evaluasi)
4. [Struktur Repositori](#struktur-repositori)
5. [Atribusi Dataset dan Referensi Ilmiah](#atribusi-dataset-dan-referensi-ilmiah)

---

## Ringkasan Proyek

Proyek ini berfokus pada pemanfaatan model *pre-trained* ringan dari Google MediaPipe tanpa proses pelatihan ulang (*zero-training baseline*) untuk menyelesaikan dua permasalahan utama:
* **Tugas 1 — Pengenalan Gestur Tangan (HaGRID Dataset)**: Mengekstraksi 21 titik *landmark* tangan (x, y, z) dan mengklasifikasikan gestur telapak tangan terbuka (**PALM**) serta kepalan tangan (**FIST**) pada 10 sampel citra beresolusi penuh.
* **Tugas 2 — Deteksi Wajah (Microsoft FaceSynthetics Dataset)**: Menjalankan detektor wajah berbasis CNN (*BlazeFace short-range*) pada 5 citra wajah sintetis utama untuk memetakan *bounding box* dan mengestimasi jumlah wajah beserta skor keyakinannya (*confidence score*).

---

## Metodologi dan Arsitektur Pipeline

### 1. Ekstraksi Landmark dan Aturan Geometris Gestur Tangan
Model `hand_landmarker.task` memprediksi 21 koordinat *keypoint* tangan yang kemudian didenormalisasi ke ukuran piksel citra asli. Klasifikasi antara **PALM** dan **FIST** ditentukan tanpa pelatihan model menggunakan rasio jarak Euclidean 2D terhadap titik pergelangan tangan (*wrist* / indeks `0`) untuk keempat jari utama (telunjuk, tengah, manis, dan kelingking):

$$\text{Rasio Jari} = \frac{\lVert \text{TIP} - \text{WRIST} \rVert}{\lVert \text{PIP} - \text{WRIST} \rVert}$$

* **Ambang Jari Lurus (`EXT_RATIO`)**: Jari dikategorikan lurus apabila rasio jarak ujung jari (`TIP`) terhadap sendi tengah (`PIP`) bernilai **> 1.15**.
* **Keputusan Kelas**: Citra diklasifikasikan sebagai **PALM** jika minimal **3 dari 4 jari** berstatus lurus, dan **FIST** jika kurang dari 3 jari lurus. Ibu jari dikecualikan karena arah geraknya menyamping secara anatomis.

### 2. Deteksi Wajah Berbasis BlazeFace Short-Range
Model `blaze_face_short_range.tflite` dijalankan pada mode `IMAGE` dengan ambang batas keyakinan minimum (`min_detection_confidence`) sebesar **0.5**. Model mengembalikan koordinat kotak pembatas `(origin_x, origin_y, width, height)` dalam satuan piksel serta skor probabilitas deteksi tanpa melakukan pengenalan identitas individu (*face recognition*).

---

## Hasil Eksperimen dan Evaluasi

### Evaluasi Tugas 1 (HaGRID — 5 PALM & 5 FIST)
* **Akurasi Klasifikasi**: **10/10 (100%)** prediksi benar dengan **0 `NOT DETECTED`**.
* **Temuan Analisis**: Seluruh citra berhasil diklasifikasikan dengan tepat, namun pada citra dengan posisi subjek yang berdiri cukup jauh dari kamera (`00086aac-e142-4c40-a7fe-5ad5e60f580b.jpg`), ukuran tangan yang sangat kecil membuat titik *landmark* menjadi kurang stabil sehingga 2 jari sempat terhitung lurus (rasio telunjuk `1.30`, kelingking `1.17`, dan jari tengah `1.14` yang hampir menyentuh ambang `1.15`).

### Evaluasi Tugas 2 (Microsoft FaceSynthetics — 5 Citra Utama)
* **Tingkat Deteksi**: **5 wajah terdeteksi** secara tepat pada 5 citra uji (`000000.png` hingga `000004.png`, masing-masing 1 wajah per citra).
* **Temuan Analisis**: Pose wajah frontal (`000000.png` dan `000001.png`) menghasilkan *confidence score* tinggi (**0.94** dan **0.93**) dengan kotak yang presisi, sedangkan pose kepala menyamping tajam yang membelakangi cahaya (`000002.png`) serta ekspresi mulut terbuka lebar dengan kepala menunduk (`000004.png`) menurunkan skor menjadi **0.84** dan **0.86**.

---

## Struktur Repositori

```text
├── 235150201111074_MediaPipe.ipynb   # Notebook utama lengkap dengan output visualisasi & analisis
└── README.md                         # Dokumentasi proyek dan atribusi sumber data

```

> **Catatan Lisensi Data**: Sesuai ketentuan lisensi penyedia dataset, berkas citra mentah dari HaGRID dan Microsoft FaceSynthetics tidak didistribusikan ulang di dalam repositori ini.

---

## Atribusi Dataset dan Referensi Ilmiah

1. **HaGRID (HAnd Gesture Recognition Image Dataset)**
* **Repositori Resmi**: [https://github.com/hukenovs/hagrid](https://github.com/hukenovs/hagrid)
* **Publikasi**: Kapitanov, A., Kvanchiani, K., Nagaev, A., Kraynov, R., & Makhliarchuk, A. (2024). *HaGRID — HAnd Gesture Recognition Image Dataset*. Proceedings of the IEEE/CVF Winter Conference on Applications of Computer Vision (WACV), 4572–4581.
* **Lisensi**: Varian CC BY-SA 4.0 (lihat folder `license` pada repositori resmi HaGRID).


2. **Microsoft FaceSynthetics**
* **Repositori Resmi**: [https://github.com/microsoft/FaceSynthetics](https://github.com/microsoft/FaceSynthetics)
* **Publikasi**: Wood, E., Baltrušaitis, T., Hewitt, C., Dziadzio, S., Cashman, T. J., & Shotton, J. (2021). *Fake It Till You Make It: Face Analysis in the Wild Using Synthetic Data Alone*. Proceedings of the IEEE/CVF International Conference on Computer Vision (ICCV), 3681–3691.
* **Lisensi**: Microsoft Research Use of Data Agreement (penggunaan riset nonkomersial).


3. **Google MediaPipe Pre-trained Models**
* [MediaPipe Hand Landmarker Documentation](https://ai.google.dev/edge/mediapipe/solutions/vision/hand_landmarker/python)
* [MediaPipe Face Detector Documentation](https://ai.google.dev/edge/mediapipe/solutions/vision/face_detector/python)



---

### Identitas Penulis

* **Nama**: Nadhif Rif'at Rasendriya
* **NIM**: 235150201111074
* **Program Studi**: S1 Teknik Informatika, Fakultas Ilmu Komputer, Universitas Brawijaya
