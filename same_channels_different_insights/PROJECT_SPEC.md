# PROJECT_SPEC.md — Frozen Stage 1 (v1.0)

**Tanggal:** 19 Oktober 2026  
**Status:** siap dijadikan kontrak implementasi Stage 2; bukan laporan hasil eksperimen.  
**Catatan asal dokumen:** `PROJECT_SPEC.md` versi asli tidak tersedia di lampiran. Spesifikasi ini disusun mandiri dari keputusan eksplisit dalam percakapan, `PROJECT_SPEC_ADDENDUM.md`, tujuh materi PDF, dan audit notebook contoh. Item yang belum terverifikasi ditandai; tidak mengklaim sebagai salinan spesifikasi yang hilang.

## 1. Identitas dan cakupan

**Judul ilmiah (FINAL):** *Same Channels, Different Insights? Comparing EEG Channel Selection for Motor Imagery Classification and Functional Connectivity*.

**Judul Indonesia:** *Kanal yang Sama, Informasi Berbeda? Perbandingan Seleksi Kanal EEG untuk Klasifikasi Motor Imagery dan Functional Connectivity*.

**Nama folder informal:** `less_is_more_eeg` (boleh dipakai; bukan judul ilmiah).

**Pertanyaan inti:** Apakah delapan kanal yang dipilih karena terbaik untuk klasifikasi imagined both fists vs imagined both feet juga menyediakan diskriminasi functional connectivity terbesar pada run uji yang belum dilihat?

**Kontribusi:** evaluasi silang yang transparan atas **kriteria pemilihan subset berbeda**, bukan penemuan metode baru. Penelitian Lou et al. (2023), Varsehi & Firoozabadi (2021), Park & Chung (2020) menunjukkan pendekatan hubungan antarkanal untuk seleksi bukan ide baru. Literatur pembanding dan keterbatasan akses: `LITERATURE_REVIEW.md`.

**Batasan:** tiga subjek sebagai studi eksploratif (opsional tambah S004–S005 hanya setelah pipeline selesai), sepuluh kandidat predefined, semua delapan kanal, satu classifier CSP–LDA, satu estimator konektivitas utama. Tidak ada deep learning, TSM, Granger, atau uji generalisasi antarsubjek.

## 2. Landasan konseptual, paper acuan, research gap

- **Grand framework:** elektrofisiologi EEG (arus postsinaptik, volume conduction), BCI.
- **Middle level:** motor imagery, perubahan mu/beta ERD/ERS, spatial filtering, functional connectivity sensor.
- **Applied:** CSP–LDA, debiased squared wPLI, skor diskriminasi konektivitas, balanced accuracy, Jaccard, validasi antar-run.
- **Anchor utama:** Lou et al. (2023), DOI 10.1109/TNSRE.2023.3252610; metode detailnya tidak boleh diuraikan sebagai terverifikasi bila hanya berdasarkan abstrak.
- **Pendukung:** Park & Chung (2020), Varsehi & Firoozabadi (2021), Feng et al. (2019), Abdullah et al. (2022), Vinck et al. (2011), Nolte et al. (2004).
- **Research gap terbatas:** dalam sumber yang ditinjau belum terdeteksi penilaian eksplisit apakah subset terbaik menurut klasifikasi juga membawa diskriminasi konektivitas di *held-out run*, dibanding subset yang dipilih menurut konektivitas dan baseline anatomi. Ini **bukan** klaim penelitian pertama atau gap universal.
- **Kebaruan praktis:** desain evaluasi silang, batas training/test yang jelas, pengujian kecocokan dua objektif. Noise-floor permutation digunakan hanya bila cukup waktu, dengan hasil dinyatakan eksploratif.

## 3. Pertanyaan, hipotesis eksploratif, pemetaan output

| ID | Pertanyaan | Hipotesis kerja, tidak harus terbukti | Output |
|---|---|---|---|
| RQ1 | Seberapa sama kanal yang dipilih dari dua kriteria? | H1: subset tidak selalu identik | Jaccard dan peta kanal |
| RQ2 | Apakah seleksi berbasis klasifikasi memperbaiki BA uji dibanding baseline anatomi? | H2: BA CLS cenderung ≥ baseline | BA per fold/subjek; tanpa klaim superiority inferensial |
| RQ3 | Apakah subset CLS tetap menangkap perbedaan konektivitas held-out dibanding CONN? | H3: CDS CLS dapat berbeda dari CDS CONN | CDS pada run uji |
| RQ4 (opsional) | Apakah skor klasifikasi dan CDS berkaitan di antara kandidat? | H4: ada asosiasi, arah belum ditentukan | Spearman deskriptif pada skor **training** 10 subset |

Untuk RQ4, korelasi 10 kandidat yang saling overlap bukan sampel independen; jangan gunakan p-value standar sebagai bukti ilmiah konfirmatori. Hindari interpretasi 'signifikan' dari n=3.

## 4. Dataset, unit analisis, dan event

**Dataset:** PhysioNet EEG Motor Movement/Imagery Database (`eegmmidb`, EDF+), 64 EEG, 160 Hz; dokumentasi resmi: https://physionet.org/content/eegmmidb/1.0.0/.

| Konfigurasi | Ketetapan |
|---|---|
| Subjek | S001, S002, S003 |
| Run | 6, 10, 14 (*imagined both fists / both feet*, bukan pelaksanaan gerak) |
| Kelas | T1 = both fists imagined; T2 = both feet imagined; T0 = rest (dikeluarkan) |
| Unit prediksi | Satu trial/epoch per subjek |
| Unit evaluasi | Run yang ditahan dalam subjek; skor per subjek dilaporkan |
| Sampling asli | 160 Hz |
| Pengecekan | MNE annotations, sample onset, banyaknya trial per kelas/run; assert setelah loading |

Perhatian: T1/T2 memiliki arti berbeda pada run unilateral. **Jangan gabungkan run di luar 6,10,14 tanpa remapping label.**

## 5. Preprocessing: keputusan yang operasional

1. Unduh EDF via `mne.datasets.eegbci.load_data` (atau read EDF setara), baca metadata masing-masing run dan **simpan identitas run sebelum concat**.
2. Standardisasi kanal `eegbci.standardize(raw)` dan pemasangan `standard_1005`/montage sesuai channel names. Pastikan daftar kandidat valid terhadap channel aktual; jangan memaksakan rename/alias diam-diam.
3. Inspeksi raw/PSD, kanal datar/saturasi, artefak dominan, laporan QC dengan alasan dan data per run. Identifikasi montage dan bad-channel sebelum memilih subset.
4. Untuk pembandingan 8 kanal vs 8 kanal, hindari preprocessing yang diam-diam memakai fitur kanal lain untuk subset terpilih. **Gunakan salinan data 64 kanal untuk QC dasar, lalu subset-specific processing konsisten**; dokumentasikan perbedaan jika memakai referensi rata-rata seluruh kanal. Sebagai baseline praktis, gunakan skema referensi konsisten yang diterapkan sebelum seleksi, tetapi jangan mengklaim simulasi perangkat fisik 8 kanal independen.
5. Filter umum 7–30 Hz (mengandung band konektivitas 8–12 Hz) dengan metode/filter sama untuk seluruh konfigurasi, `skip_by_annotation` sesuai MNE; jangan menambah notch jika tidak ada line-noise yang memerlukan.
6. Epoch pada **t=1.0 sampai 2.5 s relatif onset** untuk T1/T2; simpan y, group run, subject dan metadata; `baseline=None` kecuali ada alasan praspesifik. tmin/tmax dipilih sebelum melihat skor.
7. Gunakan QC/epoch rejection dengan ambang yang ditetapkan di awal dan log jumlah epoch yang hilang. Tidak menghapus komponen ICA bernomor tetap. ICA boleh dihilangkan demi konsistensi, atau diaplikasikan dengan dokumentasi komponen yang benar-benar diperiksa dan tanpa mengintip held-out untuk keputusan supervised.
8. Periksa kelas minimal dua kelas di masing-masing run. Jumlah trial connectivity sedikit—jangan jadikan heatmap per trial sebagai connectivity map yang stabil.

**Catatan tentang leakage:** filtering tetap yang tidak dipelajari dari label boleh diterapkan konsisten per run, tetapi keputusan QC/ICA dan transformasi adaptif harus sesuai skenario penggunaan; semua supervised fit wajib pada training-fold saja.

## 6. Kandidat kanal (10 set × 8)

Nama kandidat sementara dipilih berdasarkan montage standard 10–10; `assert set(subset).issubset(raw.ch_names)` wajib sebelum analisis. **Jika ada nama hilang, hentikan dan laporkan; jangan otomatis mengganti dengan kanal lain.**

| ID | Delapan elektroda | Konsep |
|---|---|---|
| S01 | FC3, FC4, C3, Cz, C4, CP3, CPz, CP4 | anatomical baseline sensorimotor |
| S02 | F3, F4, FC3, FC4, C3, Cz, C4, Pz | frontal-central |
| S03 | C3, Cz, C4, CP3, CP4, P3, Pz, P4 | central-parietal |
| S04 | FC1, FC2, FC3, FC4, C1, C2, C3, C4 | frontocentral |
| S05 | C1, C2, C3, Cz, C4, C5, C6, CPz | broad central |
| S06 | F3, Fz, F4, FC1, FC2, FC3, FC4, Cz | frontal |
| S07 | CP1, CP2, CP3, CP4, P3, Pz, P4, Cz | posterior sensorimotor |
| S08 | FC3, C3, CP3, P3, FC4, C4, CP4, P4 | bilateral |
| S09 | F3, F4, C3, C4, P3, P4, O1, O2 | spatially distributed control |
| S10 | Fp1, Fp2, F7, F8, P7, P8, O1, O2 | noncentral comparator |

Tidak ada klaim bahwa S10 'tanpa' sinyal motorik karena volume conduction. Semua subset ukuran 8. Kandidat tidak diubah karena hasil klasifikasi. **Status:** desain nama belum diperiksa terhadap *file EDF aktual dalam runtime ini*; Stage 2 harus memverifikasi pada data konkret.

## 7. Metode klasifikasi CSP–LDA

Satu epoch \(X\in\mathbb{R}^{C\times T}\). Setelah sinyal dipusatkan, kovariansi sampel \(\Sigma_i = X_iX_i^\top/(T-1)\). CSP memilih proyeksi \(w\) untuk membedakan variansi dua kelas, misalnya:
\[
J(w)=\frac{w^\top C_1w}{w^\top(C_1+C_2)w},\qquad C_1w=\lambda(C_1+C_2)w.
\]
Fitur adalah log-variance proyeksi spatial, classifier LDA. Gunakan `Pipeline(CSP(n_components=4, ...), LinearDiscriminantAnalysis())`. Tetapkan `n_components=4` sebagai default dan ubah hanya jika pengujian unit teknis mengharuskan (bukan tuning pada test).

**Balanced accuracy:** \(BA = \frac12 [TP/(TP+FN)+TN/(TN+FP)]\). Lapor confusion matrix pada run uji, BA per run dan mean per subjek. Periksa class count (nilai default chance ~0.5 untuk dua kelas seimbang).

## 8. Konektivitas, CDS, dan noise floor

Gunakan `mne_connectivity.spectral_connectivity_epochs(..., method='wpli2_debiased', fmin=8, fmax=12, faverage=True, mode='multitaper')` dengan **sekumpulan epoch per kelas**, bukan nilai wPLI satu epoch. Estimator `wpli2_debiased` mengestimasi **kuadrat wPLI terkoreksi bias sampel**, **bukan wPLI biasa**; estimasi finite-sample dapat negatif dan harus tetap dilaporkan/ditangani konsisten (jangan otomatis clip ke [0,1]). Dokumentasi: https://mne.tools/mne-connectivity/stable/generated/mne_connectivity.spectral_connectivity_epochs.html

Untuk setiap kandidat subset S, matriks \(\widehat{W}^{(H)}\), \(\widehat{W}^{(F)}\) kelas hands dan feet dari data yang sama-sama menjadi train atau sama-sama test. Matriks elemen off-diagonal diindeks pasangan 28 edge tak berarah.

**Skor deskriptif dasar:**
\[
CDS(S)=\frac1{28}\sum_{i<j}\left|\widehat W^{(H)}_{ij}-\widehat W^{(F)}_{ij}\right|.
\]
Nilai lebih besar bukan ukuran validitas neurofisiologis; ia rawan noise-floor dan sample-size.

**Pilihan inti untuk UTS:** gunakan CDS di atas pada train **dengan jumlah epoch yang diseimbangkan per kelas** dan dokumentasikan estimator, jumlah epoch, parameter spectral. Untuk melindungi dari artefak ukuran sampel, lakukan permutation null sederhana jika komputasi memungkinkan: shuffle label **hanya dalam training run**, pertahankan jumlah epoch per kelas, hitung ulang CDS tiap shuffle (contoh 100 permutasi, seed tetap), lalu skor terkoreksi \(CDS_x=CDS_\mathrm{obs}-\mathrm{mean}(CDS_\mathrm{perm})\). Estimator negatif dipertahankan pada selisih sebelum nilai absolut; CDSx sendiri bisa negatif dan perlu ditafsirkan hati-hati. Jangan mengurangi null memakai permutasi data uji untuk **seleksi**, dan jangan klaim uji signifikansi yang kuat dari permutasi kecil.

**Keputusan pembatasan kompleksitas:** satu metrik primer wajib adalah CDS yang konsisten; CDSx adalah audit/analisis tambahan yang jelas pelaksanaannya. Jika spesifikasi lama menuntut CDSx primer, gunakan CDSx hanya setelah prosedur dapat dijalankan, diuji dan ditulis konsisten di seluruh artefak. Jangan menyebut noise-floor 'sudah dikoreksi' jika permutation belum dijalankan.

## 9. Pemilihan subset dan validasi

**Outer three folds setiap subjek:**
- train run 6+10 → held-out 14
- train run 6+14 → held-out 10
- train run 10+14 → held-out 6.

Dalam train berisi dua run:
- **CLS selection:** bagi dua training-run untuk validasi internal; run A latih CSP-LDA → run B validasi, lalu sebaliknya. Mean BA dua putaran menentukan `S_CLS`. Refit pada seluruh dua run train dan tes sekali pada held-out outer run.
- **CONN selection:** hitung CDS dari epoch train saja untuk setiap kandidat. Agar estimasi lebih stabil, gabungkan dua run train per kelas, tetapi untuk kontrol run imbalance simpan CDS per run dan lihat konsistensi. Pilih `S_CONN = argmax CDS_train` (atau CDSx_train jika *telah disepakati dan terimplementasi*). Tie-break tetap dengan ID S paling kecil.
- **ANATOMICAL:** `S01` ditentukan sebelum model belajar. Eval sama pada held-out.
- **Test:** *setelah semua subset terkunci*, hitung BA + CDS pada held-out test untuk S01, S_CLS, dan S_CONN; hasil test **tidak boleh menentukan pilihan subset**.
- Simpan candidate scores, channel names, run, kelas, dan seed untuk audit.
- Bila satu kandidat set tidak memiliki 8 kanal yang ada, fail early; tidak boleh mengurangi jumlah kanal.

**Jaccard:** \(J(A,B)=|A\cap B|/|A\cup B|\), untuk masing-masing pasangan delapan kanal. Nilai benchmark 'chance 0.222' dari addendum sebelumnya **tidak boleh dianggap universal**, karena kandidat saling overlap dan seleksi tidak uniform. Jika ingin benchmark, simulasi dua pilihan independen dari sepuluh kandidat menggunakan distribusi sesuai aturan seleksi dan laporkan asumsi; cukup laporkan Jaccard aktual untuk scope minimum.

**Optional RQ4:** Spearman antara BA inner-CV dan CDS train di 10 kandidat; exploratory tanpa p-value konfirmatori karena subset overlap kuat.

## 10. Validitas, risiko, dan aturan klaim

- **Risk 1:** konektivitas terpengaruh volume conduction, shared reference, artefak, ketimpangan epoch, window pendek dan heterogenitas. Tidak menyimpulkan coupling anatomis atau kausalitas.
- **Risk 2:** 3 subjek × 3 run dan sekitar puluhan epoch per orang: hasil tidak dapat digeneralisasi luas; skor per subjek lebih penting daripada grand mean.
- **Risk 3:** seleksi antara kandidat sangat berkorelasi, multiple comparisons dan winner's curse. Jangan klaim metode menang secara signifikan.
- **Risk 4:** baseline 8 kanal saja tidak menjawab '8 vs 64' dan tidak membuktikan perangkat EEG 8 elektroda. Judul baru sengaja menghindari klaim itu.
- **Risk 5:** mentransformasikan full-run (ICA) sebelum pemisahan berisiko membuat informasi run held-out memengaruhi model. Default aman: tidak melakukan ICA tanpa audit, dan melakukan supervised fit train-only.
- **Risk 6:** `S_CONN` dioptimalkan untuk CDS_train sehingga hasil CDS_train tinggi secara mekanis; laporkan skor held-out untuk evaluasi yang adil.
- **Risk 7:** hasil pemilihan dari dua run mungkin tidak stabil; laporkan variasi subset dan BA antar-fold.

## 11. Struktur notebook pedagogis

Susunan notebook `.ipynb` wajib kaya Markdown berbahasa Indonesia dan **narasi → persamaan/analogi → kode → output → interpretasi**, mengikuti format notebook Michaelis–Menten, bukan meniru konten biokimia:

00 Judul, tujuan, roadmap; 01 Dasar EEG dan motor imagery; 02 Data dan event PhysioNet; 03 QC raw dan PSD; 04 Filter/epoch; 05 Sepuluh subset; 06 Intuisi + rumus CSP-LDA; 07 Intuisi phase lag, debiased squared wPLI; 08 CDS + validasi; 09 Nested run validation; 10 Hasil per-subjek; 11 Gambar klasifikasi; 12 Heatmap/graf sensor; 13 Interpretasi RQ1–RQ4; 14 Limitasi; 15 Reproducibility & referensi.

Setiap hasil kuantitatif harus berasal dari kode yang **benar-benar dijalankan**; jika belum dijalankan, tulis template interpretasi/pertanyaan pembaca, bukan angka fiktif.

## 12. Output numerik, gambar, dan file Stage 2

Folder minimal:
```
less_is_more_eeg/
  PROJECT_SPEC.md
  LITERATURE_REVIEW.md
  THEORETICAL_FRAMEWORK.md
  references.bib
  Less_Is_More_EEG.ipynb  # nama berkas boleh dipertahankan; judul isi harus judul ilmiah baru
  requirements.txt
  RESULTS_SUMMARY.md
  outputs/
    results_per_fold.csv
    candidate_training_scores.csv
    subset_selection.csv
    summary_results.csv
    figures/
```

CSV fold: subject, outer_test_run, selected_by (ANATOMICAL/CLS/CONN), subset_id, channel_list, BA_test, CDS_test, n_train, n_test, n_hands, n_feet, Jaccard_CLS_CONN. Tambahkan train subset ranking dan fold log.

Gambar wajib: raw waveform+PSD, epoch/event, topomap/konfigurasi 8 kanal, BA per subset type dan per subjek, CDS held-out per subset type, overlap Jaccard dan peta sensor, pasangan wPLI hands/feet dengan skala identik, contoh network pada posisi sensor. Plot debiased wPLI2 tidak dipaksa [0,1] jika negatif.

## 13. Acceptance tests sebelum laporan Stage 3

AT-01 download EDF subjek 1 run 6/10/14 dan event ditampilkan.
AT-02 T1/T2 diverifikasi sebagai *both fists / both feet imagery* khusus run 6/10/14.
AT-03 160 Hz, 64 kanal raw (sebelum drop QC), dan konversi satuan konsisten.
AT-04 channel names standardize dan semua 10 kandidat 8 kanal tersedia serta unik.
AT-05 label + run group count benar, tidak ada trial dari test run di training.
AT-06 pipeline sklearn fit CSP/LDA hanya pada train inner dan outer.
AT-07 held-out tidak dipakai memilih S_CLS/S_CONN.
AT-08 wPLI2_debiased menghasilkan matriks 8×8 simetris dengan diagonal dinetralisasi (bukan dianggap edge).
AT-09 CDS memakai tepat 28 unique off-diagonal edges dan balanced count.
AT-10 permutation correction jika ditulis 'dilaksanakan', dibuktikan output dan reproducible seed.
AT-11 Jaccard 0..1, identik=1, disjoint=0; tidak gunakan baseline probabilitas 0.222 tanpa asumsi.
AT-12 semua fold tidak error pada tiga subjek; jika gagal log alasannya.
AT-13 output figures/CSV dihasilkan dari hasil nyata; angka di RESULTS_SUMMARY sama dengan CSV.
AT-14 notebook bisa run all dari kernel bersih setelah dependency terpasang.
AT-15 setiap sumber dalam teks mengacu ke BibTeX dan tidak ada detail bibliografis ditebak.
AT-16 hasil 'signifikan' tidak diklaim dari tiga subjek tanpa uji dan rancangan sesuai.
AT-17 bagian interpretasi setiap grafik menjawab RQ spesifik.
AT-18 Stage 3 menulis LaTeX hanya berdasarkan hasil Stage 2 terverifikasi.

## 14. Instruksi bagi Claude Stage 2

Baca keempat file Stage 1 ini, **anggap PROJECT_SPEC.md prioritas atas prompt lama saat bertentangan**, dan lakukan single-subject smoke test sebelum skala tiga subjek. Jangan mengubah desain hanya untuk mengejar akurasi. Jangan menciptakan temuan yang belum dihitung. Jika pilihan eksperimental tidak tertulis, pilih yang paling sederhana dan dokumentasikan.

## 15. Referensi dan verifikasi

Bibliografi BibTeX: `references.bib`. Sintesis klaim dan status akses: `LITERATURE_REVIEW.md`. Diagram konsep: `THEORETICAL_FRAMEWORK.md`. Dasar teknis lain berasal dari tujuh PDF materi kuliah dan notebook contoh yang tersedia dalam percakapan. **Tidak semua paper dikaji full text**; ungkapkan batas itu dalam laporan, jangan mengaku systematic literature review.
