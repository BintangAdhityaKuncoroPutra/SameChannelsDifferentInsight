# THEORETICAL_FRAMEWORK.md — Stage 1

**Prinsip:** algoritma bukan teori. CSP, LDA, wPLI, CDS, Jaccard, dan balanced accuracy adalah **alat** yang diturunkan dari pernyataan teoretis di tingkat atasnya. Label "grand theory" di sini adalah kerangka kerja kenyamanan untuk prinsip biofisika yang mapan, bukan satu teori tunggal yang diakui bidang sebagai "grand theory".

## 1. Tiga tingkat

```
Level 1  GRAND (prinsip dasar)
         EEG = potensial ekstraseluler skala kulit kepala dari arus sinaptik
         populasi neuron, tercampur oleh konduksi volume.
         BCI = sistem yang menerjemahkan pola EEG menjadi perintah.
              |
              |  menurunkan: apa yang bisa diukur dan apa yang menyesatkan
              v
Level 2  MIDDLE-RANGE (teori domain MI)
         (a) Sensorimotor rhythm dan ERD/ERS: imajinasi gerak mengubah
             daya mu/beta secara lokal
         (b) Spatial filtering: ERD terdistribusi, sehingga filter spasial
             berbasis data memperbaiki SNR
         (c) Functional connectivity: sinkronisasi fase antar-area, dengan
             jebakan konduksi volume dan referensi bersama
              |
              |  menurunkan: operasionalisasi yang bisa diuji
              v
Level 3  APPLIED (alat yang dipakai proyek)
         CSP + LDA | debiased wPLI² | CDS dan CDSx | seleksi 8 kanal
         Balanced accuracy | Jaccard | leave-one-run-out bersarang
```

## 2. Level 1 — Grand: dasar neurofisiologis EEG dan BCI

**Pernyataan.**
1. Aktivitas saraf menghasilkan arus transmembran yang terukur di medium ekstraseluler; kontributor utama sinyal ekstraseluler adalah arus sinaptik, dengan sumber lain (spike Na⁺/Ca²⁺, fluks ion, osilasi membran intrinsik) turut membentuk medan [buzsaki2012].
2. Karena konduksi volume, rekaman EEG multikanal memberi gambaran otak yang kabur [blankertz2008]. Konsekuensi langsung untuk proyek ini: dua elektrode dapat berkorelasi karena menangkap sumber yang sama, bukan karena dua area saling berinteraksi [nolte2004; vinck2011].
3. BCI berbasis EEG pada dasarnya adalah rantai: akuisisi → pemrosesan sinyal → fitur → klasifikasi → perintah, dan kebanyakan BCI EEG saat ini bergantung pada pembelajaran mesin [lotte2018; schalk2004].

**Peran dalam proyek:** menjelaskan *mengapa* kita membatasi bahasa (konektivitas antar **sinyal sensor**, bukan antar region otak) dan *mengapa* kebocoran antar-run bisa terjadi (EEG non-stasioner antar sesi).

## 3. Level 2 — Middle-range: MI, ERD/ERS, spatial filtering, konektivitas

### 3.1 Sensorimotor rhythm dan ERD/ERS
Event-related desynchronization/synchronization (ERD/ERS) menggambarkan penurunan/kenaikan daya pita tertentu terkait peristiwa, dan prinsip dasarnya diuraikan oleh Pfurtscheller & Lopes da Silva [pfurtscheller1999]. Untuk MI, bagian yang dipakai proyek adalah pita mu dan beta di atas area sensorimotor. Feng dkk. [feng2019] merangkum hal yang sama: MI menghasilkan ERD/ERS di area sensorimotor dalam pita mu (8–13 Hz) dan beta (13–30 Hz), sehingga band-pass 8–30 Hz lazim dipakai.

**Implikasi uji:** jika kelas dibedakan terutama oleh **perubahan daya lokal**, kanal berguna untuk klasifikasi adalah kanal dengan ERD kuat dan spesifik-kelas.

### 3.2 Spatial filtering
Spatial filter berbasis data, termasuk CSP, dibenarkan karena konduksi volume menyebabkan setiap kanal mencampur banyak sumber; filter yang dioptimalkan per subjek memperbaiki SNR dibanding filter tetap berbasis geometri seperti Laplacian [blankertz2008]. CSP bekerja pada **varians** terproyeksi. Materi kuliah menambahkan bahwa itu membuang hubungan antar-kanal (elemen off-diagonal) [lihat PROJECT_SPEC §1.1].

**Implikasi uji:** CSP menjawab "kanal mana yang membawa perbedaan daya", **bukan** "kanal mana yang membawa perbedaan hubungan".

### 3.3 Functional connectivity
Sinkronisasi fase antar-area adalah salah satu cara menggambarkan interaksi. Masalahnya, konduksi volume dan referensi bersama dapat menaikkan indeks sinkronisasi secara palsu [vinck2011]. Rangkaian solusi:
- **Imaginary coherency** [nolte2004]: bagian imajiner tidak dapat dinaikkan palsu oleh konduksi volume dari sumber independen.
- **PLI** [stam2007]: menilai apakah fase memimpin/tertinggal secara tidak seimbang.
- **wPLI** [vinck2011]: memberi bobot sehingga kurang sensitif terhadap noise tak berkorelasi dan lebih berdaya; makalah yang sama membahas **sample-size bias** pada indeks fase.

**Implikasi uji:** perbedaan konektivitas antar-kelas hanya bermakna jika (i) bukan artefak sampel-kecil, (ii) bukan sekadar akibat SNR/ERD. Dua syarat itu menjadi alasan C2, C3, S1, dan S5 di spec.

### 3.4 Hubungan antar kedua cabang (inti hipotesis proyek)
Dua jalur ini memakai **statistik berbeda** dari sinyal yang sama:

| Cabang | Statistik | Dipengaruhi oleh | Bisa menunjuk kanal berbeda karena |
|---|---|---|---|
| ERD / CSP | daya (varians) lokal | amplitudo ERD, SNR | kanal dengan ERD kuat tapi tanpa kopling fase konsisten |
| Konektivitas / wPLI | konsistensi selisih fase non-zero-lag | SNR, kopling nyata, jumlah epoch | pasangan dengan kopling jelas tapi daya tak berubah antar-kelas |

Dua kemungkinan yang kita uji, tanpa memihak:
- **Konvergen:** keduanya dikendalikan faktor bersama (SNR alfa/mu di sensorimotor), jadi subset terbaik-klasifikasi juga membawa diskriminasi konektivitas.
- **Divergen:** keduanya menangkap aspek berbeda, sehingga subset yang sama tidak optimal untuk keduanya.

Konvergensi karena SNR **bukan bukti kopling saraf**. Karena itu S5 (CDSx vs ERD) wajib ada.

## 4. Level 3 — Applied framework

| Alat | Berasal dari | Status | Catatan kejujuran |
|---|---|---|---|
| CSP + LDA | §3.2 | Metode mapan | Diambil dari materi kuliah dan [blankertz2008]; bukan teori |
| Debiased wPLI² | §3.3 | Estimator dari [vinck2011], diimplementasikan seperti di MNE-Connectivity | Rumus dicek terhadap sumber MNE-Connectivity, bukan terhadap paper |
| **CDS / CDSx** | §3.4 | **Ukuran buatan proyek** | Bukan biomarker tervalidasi; kerabat konseptual selisih-korelasi [parkchung2020] |
| Seleksi 8 kanal (S_clf, S_conn, S01) | §3.4 | Prosedur desain | Hanya 10 dari C(64,8) subset ditelusuri |
| Balanced accuracy | evaluasi | Metrik standar | Kelas seimbang; chance 0,5 |
| Jaccard | evaluasi | Ukuran tumpang tindih | Tidak gunakan angka chance universal; peluang overlap bergantung pada 10 kandidat dan kebijakan pemilihan |
| Leave-one-run-out bersarang | evaluasi | Desain validasi | Menjaga seleksi dari data uji |

**Aturan penulisan untuk laporan:** CSP, wPLI, dan CDS disebut **metode/ukuran**, tidak pernah "teori". Teori adalah Level 1 dan 2.


## 5. Catatan sinkronisasi
Judul ilmiah final: **Same Channels, Different Insights? Comparing EEG Channel Selection for Motor Imagery Classification and Functional Connectivity**.

Istilah **connectivity** di sini adalah ketergantungan statistik *level sensor*, bukan pemetaan hubungan anatomis; nilai `wpli2_debiased` bukan probabilitas, tidak harus berada di rentang [0,1] pada estimasi finite-sample, dan tidak boleh diplot dengan asumsi wPLI biasa. Hierarki di atas bersifat konseptual (bukan satu teori grand yang baku).
