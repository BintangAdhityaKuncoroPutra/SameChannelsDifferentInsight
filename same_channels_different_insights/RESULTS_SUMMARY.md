# RESULTS_SUMMARY.md — Stage 2

**Proyek:** Same Channels, Different Insights? Comparing EEG Channel Selection for Motor Imagery Classification and Functional Connectivity
**Dibuat:** 2026-10-19 dari CSV di `outputs/` (angka di file ini disalin terprogram dari CSV, bukan diketik tangan).
**Notebook:** `Same_Channels_Different_Insights.ipynb` (87 sel; tersimpan **beserta output** eksekusi terverifikasi).

## 1. Status eksekusi

- **Selesai dan terverifikasi:** notebook dijalankan dari atas ke bawah di kernel bersih (`jupyter nbconvert --execute`) untuk S001, S002, S003, run 6/10/14, 100 permutasi per fold. Smoke test S001 dijalankan lebih dulu dan lulus.
- Fold: **9 dari 9 berhasil**, 0 gagal (`outputs/fold_log.csv`). Waktu eksekusi penuh ±3,5 menit (CPU 4 inti).
- Data: 9 file EDF asli PhysioNet, checksum SHA-256 cocok dengan `SHA256SUMS.txt` resmi. Di container ini physionet.org diblokir proxy, sehingga `mne.datasets.eegbci` membaca file yang diunduh dari **mirror resmi PhysioNet di AWS Open Data** (`physionet-open`); notebook beralih otomatis ke mirror itu bila unduhan MNE gagal.
- Uji kebocoran (canary): run uji diganti noise dan label diacak → skor kandidat dan pilihan subset identik. Lulus.
- Semua acceptance test yang dapat dicek dari notebook lulus (tabel di bagian 15 notebook). AT-15 (verifikasi bibliografi akhir) dan AT-18 (Stage 3) di luar notebook.

## 2. Konfigurasi

| Item | Nilai |
|---|---|
| Subjek / run | S001–S003 / 6, 10, 14 (T1 = both fists → hands, T2 = both feet → feet; T0 dibuang) |
| Referensi | referensi rekaman asli EDF, tanpa re-referencing |
| Filter | FIR band-pass 7–30 Hz, zero-phase, per run; tanpa notch |
| ICA | tidak dipakai |
| Epoch | 1,0–2,5 s setelah onset (241 sampel), `baseline=None` |
| Penolakan epoch | PTP > 400 µV di salah satu dari 64 kanal (lihat §8) |
| Klasifikasi | CSP (4 komponen, `component_order="alternate"`, log, tanpa regularisasi) + LDA |
| Konektivitas | `wpli2_debiased`, multitaper, 8–12 Hz, `faverage=True`; diagonal NaN; nilai negatif dipertahankan |
| CDS | rata-rata \|W_hands − W_feet\| pada 28 edge; epoch diseimbangkan per kelas (subsampel acak) |
| Seleksi | S_CLS = argmax BA validasi-dalam (run A↔B); S_CONN = argmax CDS latih; seri → ID terkecil; S01 tetap |
| Audit noise floor | CDSx = CDS − rata-rata CDS permutasi (100 permutasi label di dalam setiap run latih); **tidak** dipakai untuk seleksi utama |
| Seed | 20261019 |
| Perangkat lunak | Python 3.13.16, MNE 1.13.2, MNE-Connectivity 0.9.0, scikit-learn 1.9.1, NumPy 2.5.3, SciPy 1.18.1 |

## 3. Data yang diproses

| subject | run | n_cues | n_rejected | n_kept | n_hands | n_feet |
|---|---|---|---|---|---|---|
| S001 | 6 | 15 | 1 | 14 | 6 | 8 |
| S001 | 10 | 15 | 0 | 15 | 7 | 8 |
| S001 | 14 | 15 | 0 | 15 | 7 | 8 |
| S002 | 6 | 15 | 0 | 15 | 8 | 7 |
| S002 | 10 | 15 | 0 | 15 | 8 | 7 |
| S002 | 14 | 15 | 0 | 15 | 8 | 7 |
| S003 | 6 | 15 | 0 | 15 | 7 | 8 |
| S003 | 10 | 15 | 0 | 15 | 7 | 8 |
| S003 | 14 | 15 | 0 | 15 | 7 | 8 |

Total epoch: 134 (hands 65, feet 69); ditolak 1.
Kanal bertanda QC (|robust z log-variance| > 5, dilaporkan, tidak dibuang): S003 run 6 Fp1 (z = 5.30); S003 run 6 Fp2 (z = 5.29); S003 run 6 AF7 (z = 5.20). Semua 10 kandidat × 8 kanal tersedia di EDF.

## 4. Hasil per pertanyaan penelitian

### RQ2 — klasifikasi (balanced accuracy pada run uji)

| Pendekatan | Rata-rata BA (9 fold) | SD antar-fold |
|---|---|---|
| ANATOMICAL | 0.769 | 0.183 |
| CLS | 0.833 | 0.185 |
| CONN | 0.686 | 0.203 |

Per subjek (rata-rata 3 fold): S001: ANATOMICAL 0.951, CLS 0.979, CONN 0.844; S002: ANATOMICAL 0.750, CLS 0.893, CONN 0.688; S003: ANATOMICAL 0.604, CLS 0.628, CONN 0.527.
CLS vs ANATOMICAL per fold: lebih tinggi 5, sama 4, lebih rendah 0 dari 9; rata-rata selisih +0.065. S_CLS = S01 pada 2 fold. Deskriptif; tidak ada uji signifikansi.

### RQ3 — diskriminasi konektivitas pada run uji (CDS)

| Pendekatan | Rata-rata CDS uji | SD antar-fold |
|---|---|---|
| ANATOMICAL | 0.3324 | 0.0731 |
| CLS | 0.3404 | 0.0781 |
| CONN | 0.3215 | 0.0668 |

CLS vs CONN per fold: CLS lebih tinggi 5, sama (subset identik) 2, lebih rendah 2; rata-rata selisih CLS − CONN +0.0190. Subset yang dipilih karena CDS latih tertinggi **tidak** memberi CDS uji tertinggi secara rata-rata.

**Audit noise floor (data latih, 100 permutasi/fold):** rata-rata CDS null 0.2000 vs rata-rata CDS teramati semua kandidat 0.2135; hanya 0.09 dari 90 pasangan fold × kandidat berada di atas persentil-95 null (≈ 0,05 diharapkan bila tidak ada efek). Pilihan versi CDSx sama dengan versi CDS pada 67% fold. Rata-rata untuk subset terpilih:

| approach | CDS_train | CDS_perm_mean | CDS_perm_p95 | CDSx_train |
|---|---|---|---|---|
| ANATOMICAL | 0.2262 | 0.2030 | 0.3021 | 0.0232 |
| CLS | 0.2120 | 0.1930 | 0.2778 | 0.0190 |
| CONN | 0.2902 | 0.2222 | 0.3254 | 0.0680 |

Interpretasi yang didukung: dengan ±13–15 epoch latih per kelas, CDS sebagian besar berada di tingkat noise floor. Perbedaan CDS antar-pendekatan sebaiknya tidak dibaca sebagai perbedaan konektivitas neural.

### RQ1 — kesamaan subset (Jaccard)

| subject | outer_test_run | S_CLS | S_CONN | S_CONNx_audit | n_tied_CLS | n_tied_CONN | Jaccard_CLS_CONN | Jaccard_ANAT_CLS | Jaccard_ANAT_CONN |
|---|---|---|---|---|---|---|---|---|---|
| S001 | 14 | S05 | S10 | S10 | 1 | 1 | 0.000 | 0.333 | 0.000 |
| S001 | 10 | S01 | S01 | S10 | 1 | 1 | 1.000 | 1.000 | 1.000 |
| S001 | 6 | S05 | S08 | S08 | 1 | 1 | 0.143 | 0.333 | 0.600 |
| S002 | 14 | S09 | S05 | S05 | 1 | 1 | 0.143 | 0.143 | 0.333 |
| S002 | 10 | S03 | S01 | S01 | 1 | 1 | 0.455 | 0.455 | 1.000 |
| S002 | 6 | S03 | S03 | S09 | 1 | 1 | 1.000 | 0.455 | 0.455 |
| S003 | 14 | S02 | S07 | S07 | 1 | 1 | 0.143 | 0.455 | 0.231 |
| S003 | 10 | S08 | S10 | S04 | 1 | 1 | 0.000 | 0.600 | 0.000 |
| S003 | 6 | S01 | S10 | S10 | 1 | 1 | 0.000 | 1.000 | 0.000 |

S_CLS = S_CONN pada 2/9 fold; Jaccard CLS–CONN rata-rata 0.320, median 0.143 (median Jaccard di antara 45 pasangan kandidat: 0,143, hanya sebagai konteks; tidak ada baseline "chance"). Seleksi tidak stabil antar-fold: S_CLS memakai 6 subset berbeda dan S_CONN 6 subset berbeda di 9 fold. S_CONN = S10 (noncentral: Fp1, Fp2, F7, F8, P7, P8, O1, O2) pada 3 fold (S001 uji 14, S003 uji 10, S003 uji 6); pada S003, Fp1/Fp2 bertanda QC di run 6.

### RQ4 (opsional) — asosiasi skor latih

ρ Spearman (BA validasi-dalam vs CDS latih, 10 kandidat) per fold: S001/14: -0.31, S001/10: +0.38, S001/6: +0.51, S002/14: +0.07, S002/10: +0.19, S002/6: +0.89, S003/14: -0.21, S003/10: -0.31, S003/6: +0.09. Median +0.09; positif pada 6/9. Deskriptif tanpa p-value (kandidat tumpang tindih, bukan sampel independen).

## 5. Hasil per fold

| subject | outer_test_run | selected_by | subset_id | inner_BA_train | BA_test | CDS_train | CDS_test | n_test | n_hands | n_feet |
|---|---|---|---|---|---|---|---|---|---|---|
| S001 | 14 | ANATOMICAL | S01 | 0.927 | 1.000 | 0.1021 | 0.2793 | 15 | 7 | 8 |
| S001 | 14 | CLS | S05 | 0.969 | 1.000 | 0.0493 | 0.3168 | 15 | 7 | 8 |
| S001 | 14 | CONN | S10 | 0.647 | 0.679 | 0.1798 | 0.2398 | 15 | 7 | 8 |
| S001 | 10 | ANATOMICAL | S01 | 1.000 | 0.938 | 0.2276 | 0.2590 | 15 | 7 | 8 |
| S001 | 10 | CLS | S01 | 1.000 | 0.938 | 0.2276 | 0.2590 | 15 | 7 | 8 |
| S001 | 10 | CONN | S01 | 1.000 | 0.938 | 0.2276 | 0.2590 | 15 | 7 | 8 |
| S001 | 6 | ANATOMICAL | S01 | 0.969 | 0.917 | 0.2318 | 0.3987 | 14 | 6 | 8 |
| S001 | 6 | CLS | S05 | 1.000 | 1.000 | 0.1955 | 0.4502 | 14 | 6 | 8 |
| S001 | 6 | CONN | S08 | 0.866 | 0.917 | 0.2722 | 0.2804 | 14 | 6 | 8 |
| S002 | 14 | ANATOMICAL | S01 | 0.607 | 0.732 | 0.3479 | 0.3693 | 15 | 8 | 7 |
| S002 | 14 | CLS | S09 | 0.871 | 0.875 | 0.3460 | 0.3937 | 15 | 8 | 7 |
| S002 | 14 | CONN | S05 | 0.299 | 0.393 | 0.3761 | 0.3747 | 15 | 8 | 7 |
| S002 | 10 | ANATOMICAL | S01 | 0.545 | 0.866 | 0.3443 | 0.3456 | 15 | 8 | 7 |
| S002 | 10 | CLS | S03 | 0.746 | 1.000 | 0.2753 | 0.3842 | 15 | 8 | 7 |
| S002 | 10 | CONN | S01 | 0.545 | 0.866 | 0.3443 | 0.3456 | 15 | 8 | 7 |
| S002 | 6 | ANATOMICAL | S01 | 0.799 | 0.652 | 0.2455 | 0.4582 | 15 | 8 | 7 |
| S002 | 6 | CLS | S03 | 1.000 | 0.804 | 0.2912 | 0.4089 | 15 | 8 | 7 |
| S002 | 6 | CONN | S03 | 1.000 | 0.804 | 0.2912 | 0.4089 | 15 | 8 | 7 |
| S003 | 14 | ANATOMICAL | S01 | 0.446 | 0.795 | 0.1654 | 0.3349 | 15 | 7 | 8 |
| S003 | 14 | CLS | S02 | 0.531 | 0.795 | 0.1391 | 0.2698 | 15 | 7 | 8 |
| S003 | 14 | CONN | S07 | 0.451 | 0.473 | 0.2727 | 0.4018 | 15 | 7 | 8 |
| S003 | 10 | ANATOMICAL | S01 | 0.478 | 0.545 | 0.1573 | 0.3269 | 15 | 7 | 8 |
| S003 | 10 | CLS | S08 | 0.580 | 0.616 | 0.1703 | 0.3619 | 15 | 7 | 8 |
| S003 | 10 | CONN | S10 | 0.500 | 0.527 | 0.2317 | 0.3368 | 15 | 7 | 8 |
| S003 | 6 | ANATOMICAL | S01 | 0.741 | 0.473 | 0.2139 | 0.2194 | 15 | 7 | 8 |
| S003 | 6 | CLS | S01 | 0.741 | 0.473 | 0.2139 | 0.2194 | 15 | 7 | 8 |
| S003 | 6 | CONN | S10 | 0.598 | 0.580 | 0.4159 | 0.2462 | 15 | 7 | 8 |

`n_hands`/`n_feet` = jumlah trial **uji**. CDS uji memakai 6–7 epoch per kelas; CDS latih 13–14. Karena jumlah epoch berbeda, nilai CDS latih dan uji **tidak dapat dibandingkan langsung** (CDS uji lebih tinggi terutama karena noise lebih besar).

## 6. File keluaran

| File | Isi |
|---|---|
| `outputs/results_per_fold.csv` | 27 baris (3 subjek × 3 fold × 3 pendekatan): subset, kanal, BA_test, CDS_test, CDS_train, inner_BA_train, jumlah trial, Jaccard_CLS_CONN |
| `outputs/candidate_training_scores.csv` | 90 baris (9 fold × 10 kandidat): BA validasi-dalam A→B, B→A, rata-rata; CDS latih gabungan dan per run; CDS null (rata-rata, p95), CDSx; peringkat |
| `outputs/subset_selection.csv` | 9 baris: S_CLS, S_CONN, S_CONNx_audit, jumlah seri, kanal, tiga Jaccard |
| `outputs/summary_results.csv` | rata-rata dan SD BA/CDS uji per subjek × pendekatan + baris ALL |
| `outputs/rq4_spearman_training.csv` | ρ Spearman per fold |
| `outputs/fold_log.csv` | status, error, durasi tiap fold |
| `outputs/qc_channels.csv`, `outputs/qc_epochs.csv` | QC kanal (SD, robust z) dan epoch per run |
| `outputs/run_metadata.json` | parameter, versi paket, checksum data |

Gambar (`outputs/figures/`, PNG 300 dpi):

| File | Isi |
|---|---|
| `01_raw_eeg_S001_R06.png` | EEG mentah 8 kanal, 16 s pertama S001 run 6, dengan cue T1/T2 |
| `01b_filtered_eeg_S001_R06.png` | Segmen yang sama setelah band-pass 7–30 Hz |
| `02_psd_raw.png` | PSD Welch mentah per subjek (rata-rata 64 kanal; C3/Cz/C4), puncak 60 Hz ditandai |
| `03_qc_epoch_ptp_distribution.png` | Distribusi PTP maksimum per epoch; ambang 150 µV vs 400 µV |
| `04_event_timeline_S001.png` | Garis waktu T0/T1/T2 dan jendela epoch 1,0–2,5 s, S001 |
| `05_representative_epochs_S001.png` | Satu trial hands dan feet pada C3/Cz/C4 relatif onset |
| `06_candidate_subsets_sensor_map.png` | Peta 10 kandidat subset pada kepala |
| `07_csp_demo_S001.png` | Demo CSP: kovarians kelas, selisih, fitur log-variance latih/uji (S001, uji run 14, S01) |
| `08_wpli_synthetic_demo.png` | Simulasi wPLI vs debiased wPLI² (campuran seketika, lag, noise) |
| `09_balanced_accuracy_by_approach.png` | BA held-out per pendekatan, per subjek dan gabungan (RQ2) |
| `10_confusion_matrices.png` | Confusion matrix held-out dijumlah 3 run uji, per subjek × pendekatan |
| `11_cds_heldout_by_approach.png` | CDS held-out per pendekatan (RQ3) |
| `12_cds_train_vs_test.png` | CDS latih vs uji subset terpilih (beda jumlah epoch; lihat catatan) |
| `13_connectivity_heatmaps_S001_test14.png` | Matriks debiased wPLI² hands/feet/selisih, run uji, skala sama |
| `14_connectivity_network_S001_test14.png` | Jaringan selisih 28 edge di posisi sensor, tanpa ambang |
| `15_jaccard_overlap.png` | Jaccard per fold + konteks Jaccard antar-kandidat (RQ1) |
| `16_selection_frequency_sensor_map.png` | Frekuensi kanal terpilih CLS vs CONN di 9 fold |
| `17_rq4_innerBA_vs_CDStrain.png` | Scatter BA validasi-dalam vs CDS latih 10 kandidat (RQ4) |

## 7. Komputasi gagal / keluaran hilang

Tidak ada fold gagal dan tidak ada keluaran wajib yang hilang. Yang **tidak** dikerjakan: kontrol "S5" (CDSx vs ERD/daya) dari `THEORETICAL_FRAMEWORK.md`, karena tidak tercantum di `PROJECT_SPEC.md` final; benchmark Jaccard berbasis simulasi (opsional di spec); subjek tambahan S004–S005.

## 8. Penyimpangan dan keputusan yang perlu disebut di laporan

1. **Ambang penolakan epoch 400 µV**, bukan ambang konvensional 100–150 µV: pada data 7–30 Hz, ambang 150 µV akan menolak 98% epoch S001, 13% S002, dan 100% S003. Ambang ditetapkan setelah melihat distribusi amplitudo, tanpa label atau skor; hanya 1 epoch ditolak.
2. **Tanpa re-referencing**, agar subset 8 kanal tidak memakai informasi 56 kanal lain lewat average reference.
3. `component_order="alternate"` pada CSP (2 filter dari tiap ujung spektrum eigen) agar sesuai uraian buku teks; spec hanya menyebut `n_components=4`.
4. Montage `colin27_1005` = nama baru `standard_1005` di MNE 1.13 (posisi sama).
5. Nama folder/notebook mengikuti prompt Stage 2 (`same_channels_different_insights/`), bukan `less_is_more_eeg/` di spec.
6. Kanal Fp1/Fp2/AF7 S003 run 6 bertanda QC tetapi tidak dibuang (spec melarang mengubah ukuran subset); S10 yang memuatnya terpilih oleh CONN pada dua fold S003.

## 9. Keterbatasan utama

- n = 3 subjek, ±15 trial per run; BA berbutir ±0,07 per trial; konektivitas uji dari ±7 epoch per kelas.
- CDS hampir setara noise floor; CDS bukan biomarker neural; dapat dipengaruhi SNR/daya dan artefak (frontal/oksipital).
- Volume conduction, source mixing, dan referensi bersama tidak sepenuhnya dihapus oleh wPLI²; semua hubungan level sensor, tanpa makna kausal/anatomis.
- Seleksi dari 10 kandidat tumpang tindih: winner's curse dan ketidakstabilan antar-fold.
- Literatur: banyak sumber hanya AB/AB+; jangan menyatakan metode Lou dkk. (2023) secara rinci.

## 10. Rekomendasi untuk diskusi Stage 3

- Laporkan hasil per subjek dan per fold, bukan hanya rata-rata ALL; S003 dekat tingkat acak untuk semua pendekatan.
- Pesan utama yang didukung data: kriteria klasifikasi dan konektivitas jarang memilih subset yang sama (2/9 fold), seleksi CLS tidak pernah lebih buruk dari S01 pada BA uji dalam data ini, sedangkan seleksi CONN memberi BA rata-rata lebih rendah dan tidak mempertahankan keunggulan CDS di run uji. Bingkai sebagai deskriptif.
- Tekankan audit noise floor: dengan jumlah trial ini, diskriminasi konektivitas sulit dibedakan dari noise, sehingga RQ3 lebih tepat dijawab "tidak dapat dibedakan secara andal" daripada "CLS menangkap konektivitas".
- Sebut pilihan S10 oleh CONN dan kanal frontal bertanda QC sebagai kemungkinan pengaruh artefak (hipotesis, belum diuji).
- Hindari kata "signifikan", "koneksi otak", dan "kausal".

## 11. Cara menjalankan ulang

```bash
pip install -r requirements.txt
jupyter nbconvert --to notebook --execute Same_Channels_Different_Insights.ipynb --output Same_Channels_Different_Insights.ipynb
# smoke test: SCDI_SUBJECTS=1 SCDI_N_PERM=0 jupyter nbconvert ...
```
Data (±23 MB) diunduh otomatis ke `~/mne_data` (atau `SCDI_DATA`).
