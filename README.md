# docs-viswork-model

# 🎥 Viswork

## 📌 Overview

**Viswork** adalah platform **video analytics berbasis AI** yang memanfaatkan teknologi **computer vision** untuk melakukan analisis **real-time** dari kamera **CCTV**. Sistem ini dirancang untuk mengubah *video mentah* menjadi **insight terstruktur** yang dapat digunakan untuk kebutuhan operasional, monitoring, dan pengambilan keputusan.

Fokus utama use-case Viswork meliputi:

- 🚗 Vehicle Detection & Analytics
- 👥 Crowd Analysis
- 🚦 Traffic Monitoring
- 🅿️ Parking Analysis

---

## 🧩 AI Use Case – Viswork (End-to-End Pipeline)

Bagian ini menjelaskan **perjalanan AI (AI journey)** di Viswork, dari data video masuk hingga menjadi insight dan laporan.

---

### 📡 1. Data Source & Input Layer

**Sumber Data**
- CCTV

**Karakteristik Data**
- Continuous video stream
- Variasi pencahayaan (siang / malam)
- Occlusion dan overlapping object
- Perbedaan sudut kamera (fixed camera)

**Output**
- Frame video hasil sampling (mis. **15–20 FPS**)

---

### ⚙️ 2. Preprocessing Layer

Setiap frame video melalui tahap preprocessing:

- Resize ke ukuran input model (mis. **640 × 640**)
- Normalisasi pixel
- Konversi ke tensor format **NCHW**

Tujuan utama preprocessing adalah menjaga stabilitas inferensi dan mengoptimalkan performa **real-time detection**.

---

### 🧠 3. Object Detection Layer

Viswork menggunakan arsitektur **pluggable object detection**.

Model yang didukung misal:
- YOLOv11
- RT-DETRv4

Fungsi utama:
- Mendeteksi objek pada setiap frame
- Menghasilkan bounding box, class label, dan confidence score

---

### 🧹 4. Post-Processing Layer

- Confidence thresholding
- Non-Maximum Suppression (NMS) *(khusus dense detector)*

---

### 🔗 5. Object Tracking Layer

- Assign unique object ID
- Melacak arah dan durasi pergerakan objek

---

### 📊 6. Analytics Layer

#### 🚗 Vehicle Analytics
- Vehicle counting
- Vehicle type classification

#### 👥 Crowd Analytics
- People counting
- Gender estimation
- Crowd density

#### 🚦 Traffic Analytics
- Vehicle counting
- Estimasi kecepatan
- Kondisi lalu lintas

#### 🅿️ Parking Analytics
- Status lahan parkir
- Jumlah kendaraan per lokasi

---

### 🖥️ 7. Visualization & Output Layer

- Bounding box overlay (real-time)
- Dashboard analytics
- Laporan **PDF**

---

### 🔁 Use-case Summary

```text
Video → Frame → Detection → Tracking → Analytics → Insight → Report
```

---


# 🧠 YOLOv11 dan RT-DETRv4  
### Architecture & Detection Paradigm Overview

Dokumen ini menjelaskan **perbedaan arsitektur, paradigma deteksi, dan strategi training** antara **YOLOv11** dan **RT-DETRv4**.

---

## 📌 Scope Dokumen

Fokus utama:
- Arsitektur internal model
- Detection paradigm
- Assignment & matching strategy
- Loss function
- Implikasi terhadap real-time inference

Dokumen **tidak membahas benchmark angka**, melainkan **mekanisme desain model**.

---

# 🧠 YOLOv11 Architecture

## 📌 Overview

**YOLOv11 (You Only Look Once)** adalah model **single-stage object detector** berbasis CNN yang dirancang untuk mencapai keseimbangan optimal antara:

- ⚡ Kecepatan inferensi
- 🎯 Akurasi deteksi
- 🧮 Efisiensi komputasi

YOLOv11 sangat cocok untuk **real-time object detection** seperti:
- Vehicle detection
- Traffic monitoring
- CCTV analytics

---

## 🔁 Architecture Flow

```

Input Image
│
▼
Backbone
├─ CSP Blocks (C3k2)
├─ SPPF
└─ C2PSA
│
▼
Neck (PAFPN)
├─ Top-down
└─ Bottom-up
│
▼
Detection Head
├─ Bounding Box Regression
├─ Objectness Prediction
└─ Class Prediction
│
▼
Bounding Box + Class + Confidence
│
└─ (Training Only)
   ├─ CIoU Loss
   ├─ DFL
   └─ BCE 
```

---

## 🧩 Backbone

Backbone berfungsi sebagai **ekstraktor fitur utama**.

### Komponen Utama

**C3k2 Block**
- Evolusi dari C2f (YOLOv8)
- CSP-based bottleneck
- Kernel lebih kecil dan parameter lebih efisien

**SPPF (Spatial Pyramid Pooling – Fast)**
- Menangkap konteks multi-skala
- Lebih ringan dibanding SPP klasik

**C2PSA (Cross-stage Partial with Spatial Attention)**
- Integrasi spatial attention
- Fokus pada area penting tanpa overhead besar

---

## 🔗 Neck (PAFPN)

YOLOv11 menggunakan **Path Aggregation Feature Pyramid Network (PAFPN)** untuk fusi fitur multi-skala.

### Karakteristik
- **Top-down pathway**  
  Menggabungkan semantic feature tingkat tinggi ke resolusi lebih rendah
- **Bottom-up pathway**  
  Mengirim kembali detail spasial ke level fitur atas

Tujuan utama: meningkatkan deteksi objek **kecil, sedang, dan besar** secara seimbang.

---

## 🎯 Detection Head

Detection head menghasilkan prediksi akhir berupa:

- **Bounding Box Regression**
  - Menggunakan **Distribution Focal Loss (DFL)**
- **Objectness Score**
  - Menentukan keberadaan objek
- **Class Probabilities**
  - Multi-class / multi-label detection

---

## 📌 Detection Paradigm: Dense, Grid-based

YOLOv11 menggunakan **dense detection paradigm**:

- Prediksi dilakukan pada **setiap posisi grid** di feature map
- Satu objek dapat menghasilkan **banyak prediksi**
- Prediksi bersifat **lokal** (berdasarkan receptive field CNN)

Yang akan terjadi adalah:
- Inferensi cepat
- Menghasilkan prediksi tumpang tindih

---

## ⚙️ Assignment Strategy: Heuristic-based

YOLOv11 menggunakan **heuristic (task-aligned) label assignment**, bukan Hungarian matching.

Selama training:
- Setiap ground truth dibandingkan dengan banyak kandidat prediksi
- Seleksi positive sample berdasarkan:
  - IoU
  - Center prior
  - Task-aligned score (classification × IoU)
- **Top-k kandidat** dipilih sebagai positive

Assignment bersifat **lokal dan rule-based**, bukan optimisasi global.

---

## ✂️ Non-Maximum Suppression (NMS)

Karena dense prediction menghasilkan banyak bounding box tumpang tindih, YOLOv11 memerlukan **NMS** untuk:

- Menghapus duplicate detection
- Mempertahankan bounding box terbaik

---

## 📉 Loss Function (YOLOv11)

- **CIoU Loss** → bounding box alignment
- **BCE Loss** → classification & objectness
- **DFL** → meningkatkan presisi regresi box

---

# 🧠 RT-DETRv4 Architecture

## 📌 Overview

**RT-DETRv4** adalah model **end-to-end object detector berbasis query**, turunan dari paradigma **DETR**.

Karakteristik utama:
- Set-based prediction
- Tanpa anchor
- Tanpa NMS
- Optimisasi global

---

## 🔁 Architecture Flow

```

Input Image
│
▼
CNN Backbone (S3, S4, S5)
│
▼
Hybrid Encoder
├─ AIFI (Self-Attention on S5)
└─ CCFF (Cross-scale Feature Fusion)
│
▼
Transformer Decoder
└─ Object Queries
│
▼
Predictions
├─ Bounding Box
├─ Class Probability
└─ Confidence
│
└─ (Training Only)
   ├─ Hungarian Matching
   ├─ Classification Loss
   ├─ L1 Loss
   ├─ GIoU Loss
   ├─ Deep Semantic Injector (DSI)
   └─ Gradient-guided Adaptive Modulation (GAM)

```

---

## 🧩 Backbone

Lightweight CNN menghasilkan fitur multi-skala:

- **S3** → low-level
- **S4** → mid-level
- **S5** → high-level

---

## 🔀 Hybrid Encoder

### AIFI (Attention-based Intra-scale Feature Interaction)
- Self-attention hanya pada S5
- Menangkap **global context** dengan overhead minimal

### CCFF (Cross-scale Feature Fusion)
- Menyebarkan semantic information dari F5 ke fitur resolusi lebih rendah
- Menghasilkan P3, P4, P5

---

## 🔁 Transformer Decoder & Object Queries

- Menggunakan **fixed object queries**
- 1 query bertanggung jawab terhadap maksimal 1 objek
- Prediksi dilakukan secara **global**

Tidak ada grid, anchor, atau center prior.

---

## 🔗 Matching Strategy: Hungarian Algorithm

RT-DETRv4 menggunakan **Hungarian matching** untuk mencocokkan prediksi dan ground truth:

- One-to-one assignment
- Optimisasi global
- Tidak ada duplicate detection
- ❌ Tidak memerlukan NMS
- ❌ Tidak memerlukan anchor

---

## 🏋️ Training-only Enhancements

Aktif hanya saat training:

**Deep Semantic Injector (DSI)**
- Injeksi semantic knowledge dari vision foundation models
- Mempercepat konvergensi

**Gradient-guided Adaptive Modulation (GAM)**
- Menyeimbangkan semantic supervision secara dinamis
- Meningkatkan stabilitas training

> Kedua modul **tidak aktif saat inferensi**.

---

## 📉 Loss Function (RT-DETRv4)

- **Classification Loss**
- **L1 Loss** → regresi box
- **GIoU Loss** → kualitas geometri box

Loss dihitung setelah Hungarian matching.

---

# ⚖️ YOLOv11 vs RT-DETRv4

| Aspek | YOLOv11 | RT-DETRv4 |
|------|--------|-----------|
| Detection Paradigm | Dense, grid-based | Set-based, query-based |
| Architecture | CNN | CNN + Transformer |
| Global Context | Terbatas | Sangat kuat |
| Anchor | Ya (anchor-free, heuristic) | Tidak |
| NMS | Ya | Tidak |
| Matching Strategy | Heuristic | Hungarian |
| Loss Style | CIoU + DFL + BCE | CLS + L1 + GIoU |
| Real-time Suitability | Sangat tinggi | Tinggi (lebih berat) |

---

---

## 📚 References

- Ultralytics YOLOv11 Documentation  
  https://docs.ultralytics.com/models/yolo11/
- YOLOv11 Paper  
  https://arxiv.org/pdf/2506.14696  
  https://arxiv.org/abs/2410.17725
- RT-DETRv4 Paper  
  https://arxiv.org/abs/2510.25257
```

---

