# LITERATURE_REVIEW.md 


## 1. Peta literatur singkat

Pemilihan kanal untuk motor imagery (MI) adalah topik mapan. Review Abdullah dkk. (2022) menyatakan tujuan utamanya: menurunkan kompleksitas komputasi, mengurangi overfitting, dan memperpendek waktu pemasangan; metodenya dibagi menjadi pendekatan berbasis klasifikasi dan tanpa klasifikasi [abdullah2022]. Keluarga metode yang muncul pada sumber:

1. **Berbasis CSP:** CSP-rank memeringkat elektrode lewat bobot filter CSP, dan akurasi dipakai sebagai kriteria berhenti [feng2019].
2. **Berbasis statistik per kanal:** entropi rata-rata kanal [shiam2024].
3. **Berbasis hubungan antar-kanal:** koefisien korelasi [parkchung2020], Granger causality [varsehi2021], graf konektivitas dengan GCN [liang2023], dan graf fungsional dalam model deep learning [lou2023].

Kelompok ke-3 menunjukkan bahwa **menghubungkan informasi antar-kanal dengan pemilihan kanal bukan ide baru**.

## 2. Hierarki anchor

| Peran | Makalah | Alasan |
|---|---|---|
| **Primary anchor** | Lou, Ye, Yao & Zhang (2023), *Less Is More: Brain Functional Connectivity Empowered Generalizable Intention Classification With Task-Relevant Channel Selection*, IEEE TNSRE. DOI 10.1109/TNSRE.2023.3252610 | Paling dekat: memilih kanal berbasis functional connectivity, memakai EEGMMI, dan menyimpulkan performa serupa dengan 20% data mentah |
| Supporting 1 | Varsehi & Firoozabadi (2021), Neural Networks 133 | Pemilihan kanal berbasis Granger causality pada PhysioNet MI, 8 kanal |
| Supporting 2 | Park & Chung (2020), IEEE Access 8 | Selisih koefisien korelasi antar-kelas dipakai memilih kanal untuk CSP; kerabat konseptual CDS |
| Supporting 3 | Feng dkk. (2019), Comput Intell Neurosci | Baseline CSP-rank + LDA |
| Supporting 4 | Shiam dkk. (2024), Brain Sciences 14(5) | CSP + SVM, tugas right hand vs right foot (mirip hands vs feet kita) |
| Supporting 5 | Liang dkk. (2023), Cognitive Neurodynamics | Konektivitas inter-kanal sebagai dasar seleksi kanal (GCN) |
| **Review** | Abdullah, Faye & Islam (2022), Bioengineering 9(12):726. DOI 10.3390/bioengineering9120726 | Lanskap metode seleksi kanal MI |
| Metodologis | Vinck dkk. (2011); Nolte dkk. (2004) | Dasar wPLI dan imaginary coherency |

## 3. Matriks literatur

### 3.1 Lou dkk. (2023) — primary anchor 
- **Judul/DOI:** *Less Is More: …* — 10.1109/TNSRE.2023.3252610
- **Dataset:** EEGMMI (eegmmidb, PhysioNet), pengujian subject-independent, tugas imajinasi buka-tutup kepalan kiri/kanan.
- **Seleksi kanal:** task-adaptive, dari graf yang dibangun dengan functional connectivity (bukan jarak elektrode).
- **Klasifikasi:** model graf/deep learning (SIFT-EEG), dibandingkan dengan model berbasis CNN dan GNN.
- **Konektivitas:** functional connectivity untuk membangun graf. **[CEK]** definisi ukuran konektivitas spesifik memerlukan verifikasi dari metode lengkap.
- **Evaluasi:** subject-independent; tabel membandingkan seleksi task-adaptive, acak, dan semua kanal.
- **Temuan (abstrak):** perbaikan sekitar 1% atas model CNN dan 11% atas GNN; seleksi task-adaptive memberi performa serupa dengan 20% kanal; seleksi acak lebih rendah.
- **Keterbatasan:** konektivitas dipakai sebagai **alat seleksi dan representasi**, bukan sebagai **objek yang dievaluasi** pada subset hasil seleksi lain. Tidak membandingkan subset terpilih-klasifikasi dengan subset terpilih-konektivitas. Model kompleks, lintas 109 subjek (bukan within-subject).
- **Relevansi:** bukti paling kuat bahwa "konektivitas + seleksi kanal + EEGMMI" sudah dikerjakan. Judul proyek kita **berbagi frasa awal** "Less Is More" (lihat Addendum §A.1).

### 3.2 Varsehi & Firoozabadi (2021) 
- **Dataset:** PhysioNet MI. **Seleksi:** analisis Granger causality, ditambah klasterisasi komponen ICA menjadi artefak dan EEG normal.
- **Klasifikasi:** CSP/RCSP dengan k-NN, SVM, LDA; kelas left vs right hand.
- **Konektivitas:** Granger causality (terarah).
- **Temuan (abstrak):** 8 kanal terpilih memberi akurasi 93,03% dengan RCSP dan "best classifier for each subject", naik 3,95 poin dari seleksi berbasis korelasi.
- **Keterbatasan:** frasa "best classifier for each subject" mengisyaratkan pemilihan klasifier memakai data uji, sehingga angka mungkin optimistis. **[CEK]** apakah seleksi kanal ada di dalam fold. Granger causality pada EEG sensor rentan volume conduction.
- **Relevansi:** jumlah kanal sama (8) dan dataset sama; tetapi tugas left/right, bukan hands/feet, dan konektivitas dipakai untuk seleksi, bukan dievaluasi silang.

### 3.3 Park & Chung (2020) 
- **Dataset:** BCI Competition III IVa dan BCI Competition IV Dataset I.
- **Seleksi kanal:** "distinctive channel" ditentukan dari jumlah kanal lain yang selisih koefisien korelasinya besar antar dua tugas MI; kelompok kanal terkorelasi kuat dibentuk; kelompok dengan Fisher score FBCSP tertinggi dipilih.
- **Klasifikasi:** FBCSP. **Konektivitas:** koefisien korelasi (Pearson, simetris, rentan zero-lag).
- **Temuan (abstrak):** memilih kanal paling sedikit rata-rata dan mengungguli metode pembanding.
- **Keterbatasan:** hanya dua dataset kompetisi; **[CEK]** detail validasi.
- **Relevansi:** **selisih konektivitas antar-kelas sebagai dasar seleksi sudah ada** (versi korelasi). CDS kita adalah kerabat dekatnya dengan estimator berbeda (debiased wPLI², dikoreksi noise floor).

### 3.4 Feng dkk. (2019) 
- **Dataset:** BCI Competition III IVa (5 subjek, 118 kanal, 140 trial/kelas, right hand vs right foot) dan BCI Competition IV Dataset 1 (4 subjek sehat, 59 kanal, 200 trial kalibrasi).
- **Seleksi:** CSP-rank per sub-band (7 band, 8–30 Hz); fitur CSP digabung, LASSO memilih fitur.
- **Klasifikasi:** LDA. **Konektivitas:** tidak ada.
- **Evaluasi:** 7-fold (dataset 1) dan 10-fold (dataset 2) cross-validation, akurasi rata-rata.
- **Temuan:** akurasi 82,48% vs 75,43% (CSP-rank) pada dataset 1; 77,75% vs 70,75% pada dataset 2.
- **Keterbatasan (dari teks):** penulis menyatakan statistik tingkat grup tidak dilaporkan karena subjek sedikit. Jumlah kanal optimal dibaca dari kurva akurasi validasi rata-rata semua subjek, dan teks tidak menyatakan bahwa pemilihan itu terisolasi dari fold uji. Itu **kemungkinan sumber optimisme**, bukan kesimpulan pasti.
- **Relevansi:** mewakili pendekatan "seleksi dioptimalkan untuk klasifikasi" yang kita beri nama S_clf; juga contoh bahwa kecilnya jumlah subjek lazim dan diakui.

### 3.5 Shiam dkk. (2024)
- **Dataset:** BCI Competition III-IVa dan IV-I. **Seleksi:** entropi rata-rata per kanal (entropi tinggi dipilih).
- **Klasifikasi:** CSP per sub-band + SVM, right hand vs right foot.
- **Konektivitas:** tidak ada. **[CEK]** protokol validasi.
- **Relevansi:** contoh seleksi tanpa label; tugas hand vs foot sejalan dengan kita.

### 3.6 Liang dkk. (2023) 
- **Dataset:** tiga dataset MI. **Seleksi:** kanal diperlakukan sebagai graf; seleksi sebagai klasifikasi node dengan GCN; asumsinya kanal berguna berbagi informasi yang terukur lewat konektivitas inter-kanal.
- **Keterbatasan:** **[CEK]** ukuran konektivitas, protokol, hasil.
- **Relevansi:** kerabat lain dari "konektivitas → seleksi kanal".

### 3.7 Abdullah dkk. (2022) — review 
- Mengulas algoritma seleksi kanal MI, tujuan, dan pasangan klasifikasinya pada berbagai dataset; menitikberatkan metode yang memilih kanal lebih sedikit dengan presisi tinggi. Reviewer peer-review mencatat bahwa analisis dalam review dinilai kurang mendalam oleh salah satu reviewer, jadi review ini lebih cocok sebagai peta, bukan bukti evaluatif.

### 3.8 Dasar metodologis
- **Vinck dkk. (2011) :** memperkenalkan wPLI; menunjukkan keunggulan atas PLI (kurang sensitif terhadap noise tak berkorelasi dan lebih berdaya mendeteksi perubahan), dan membahas **sample-size bias**. Ini mendukung keputusan C2 (debiased) di spec.
- **Nolte dkk. (2004) :** bagian imajiner coherency tidak dapat dinaikkan secara palsu oleh volume conduction sumber independen.

## 4. Sintesis: apa yang sudah dan belum dijawab

| Pertanyaan | Status di literatur yang saya periksa |
|---|---|
| Seleksi kanal MI memakai CSP/entropi/statistik dan dinilai dengan akurasi | **Sudah diteliti** [abdullah2022; feng2019; shiam2024] |
| Informasi hubungan antar-kanal dipakai untuk memilih kanal | **Sudah diteliti** [parkchung2020; varsehi2021; liang2023; lou2023] |
| Selisih konektivitas antar-kelas sebagai kriteria seleksi | **Sudah diteliti** versi korelasi [parkchung2020]; versi graf fungsional [lou2023] |
| Konektivitas debiased, dikoreksi null permutasi, sebagai metrik evaluasi pada subset | **Tidak ditemukan** |
| Apakah subset terbaik-klasifikasi dan terbaik-konektivitas **bertepatan**, dan apakah yang satu menangkap diskriminasi yang lain, diuji **silang pada data uji** | **Tidak ditemukan** |
| Desain leave-one-run-out bersarang, dengan label-shuffle control, pada beberapa subjek EEGMMI | **Tidak ditemukan** |

