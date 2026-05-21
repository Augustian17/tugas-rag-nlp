# 📘 RAG Parameter Exploration Assignment

Pada tugas ini kamu akan bereksperimen dengan **6 parameter utama Retrieval-Augmented Generation (RAG)** menggunakan **RAG Playground**.

### 🎯 Tujuan Pembelajaran

Kamu akan:

✅ Memahami **apa fungsi** masing-masing parameter RAG

✅ Mengamati **pengaruh** tuning terhadap kualitas jawaban vs kecepatan

✅ Belajar **trade-off** antara retrieval & generation

✅ Menulis **insight praktis** dari eksperimen yang kamu lakukan

> **Tools sudah disediakan.** Kamu **tidak perlu ngoding**, hanya ubah nilai parameter di file `app.py` & restart server.

### ⚠️ Notes Penting

- **Ikuti video di LMS jika kamu kebingungan.**
- **Sediakan penyimpanan at least 2GB**, akan ada download model yang cukup besar ke laptopmu.
- **Hapus proyek ini kalau sudah selesai**, biar laptop-mu tidak tercekik.
- **Ketika proyek ini sudah diisi & dikerjakan di bawah, upload ke GitHub.**

---

## ⚙️ Parameter yang Diuji

Semua parameter berikut dapat ditemukan di file `app.py`:

```python
# Buka app.py, cari bagian KONFIGURASI INTI
CHUNK_SIZE = 300
TOP_K_INITIAL = 20
FINAL_TOP_M = 5
USE_RERANKER = True
TEMPERATURE = 0.2
GROQ_MODEL = "llama-3.1-8b-instant"
```

---

### 📏 1. CHUNK_SIZE

```python
CHUNK_SIZE = 300  # ubah angka ini
```

**Apa itu CHUNK_SIZE?**

Saat dokumen PDF diupload, teksnya dipotong-potong menjadi bagian kecil (chunk) agar bisa diproses dan dicari. `CHUNK_SIZE` menentukan berapa banyak **kata** per potongan.

**Fungsi:**

> Jawab: CHUNK_SIZE berfungsi untuk menentukan ukuran potongan teks (chunk) dari dokumen PDF yang diupload. Semakin kecil nilainya, semakin banyak potongan yang dihasilkan namun setiap potongan lebih pendek dan spesifik. Semakin besar nilainya, semakin sedikit potongan namun setiap potongan memuat lebih banyak konteks. Parameter ini sangat mempengaruhi seberapa detail dan relevan konteks yang bisa ditemukan saat pencarian.

**Pengaruhnya:**

| Range Nilai | Efek |
| --- | --- |
| **Kecil (100–200)** | Menghasilkan banyak chunk yang sangat spesifik. Presisi tinggi untuk pertanyaan detail, namun berisiko konteks terpotong dan kehilangan informasi antar kalimat. Pada eksperimen dengan CHUNK_SIZE=100, dokumen terpecah menjadi 11 chunk. |
| **Sedang (250–400)** ⭐ Recommended | Keseimbangan antara presisi dan kelengkapan konteks. Chunk cukup panjang untuk memuat informasi yang bermakna, namun tetap spesifik untuk pencarian vektor. Range ini paling stabil untuk berbagai jenis dokumen. |
| **Besar (500–1000)** | Menghasilkan sedikit chunk (pada eksperimen dengan CHUNK_SIZE=700, dokumen hanya jadi 1 chunk). Konteks lebih lengkap namun pencarian vektor kurang presisi karena setiap chunk memuat terlalu banyak topik sekaligus, sehingga relevansi menurun. |

**Pertanyaan Refleksi:**

* Chunk besar atau kecil yang lebih tepat untuk dokumen yang kamu tes? Mengapa?
  > Jawab: Untuk dokumen Bill of Lading (BL) yang digunakan, chunk sedang (300 kata) lebih tepat. Dokumen ini pendek namun padat informasi spesifik seperti nomor BL, nama kapal, dan pelabuhan. Chunk kecil (100) memang menghasilkan lebih banyak potongan (11 chunk), namun chunk besar (700) justru menjadikan seluruh dokumen hanya 1 chunk sehingga pencarian vektor tidak bisa membedakan bagian mana yang paling relevan.

* Apa yang terjadi kalau jawaban terasa "terpotong" atau kurang konteks? Parameter mana yang harus diubah?
  > Jawab: Jika jawaban terasa terpotong atau kurang konteks, parameter yang harus diubah adalah CHUNK_SIZE (dinaikkan agar setiap potongan memuat lebih banyak informasi) dan FINAL_TOP_M (dinaikkan agar lebih banyak chunk yang dikirim ke LLM sebagai konteks). CHUNK_OVERLAP juga bisa dinaikkan untuk mengurangi kehilangan informasi di batas antar chunk.

---

### 🔍 2. TOP_K_INITIAL

```python
TOP_K_INITIAL = 20
```

**Apa itu TOP_K_INITIAL?**

Setelah pertanyaan diubah jadi vektor, sistem mencari chunk yang paling mirip. `TOP_K_INITIAL` menentukan berapa banyak **kandidat awal** yang diambil dari hasil pencarian vektor sebelum di-filter/rerank.

**Fungsi:**

> Jawab: TOP_K_INITIAL berfungsi sebagai "jaring pertama" dalam proses retrieval. Sistem mengambil sejumlah K chunk yang paling mirip secara vektor (cosine similarity) dengan pertanyaan yang diajukan. Kandidat-kandidat ini kemudian akan difilter lebih lanjut oleh reranker (jika aktif) untuk dipilih yang paling relevan. Semakin besar K, semakin luas jangkauan pencarian, namun semakin berat proses reranking-nya.

**Pengaruhnya:**

| Range Nilai | Efek |
| --- | --- |
| **Kecil (5–10)** | Pencarian lebih cepat namun berisiko melewatkan chunk yang relevan (recall rendah). Cocok untuk dokumen kecil atau saat kecepatan lebih diprioritaskan daripada kelengkapan. |
| **Sedang (15–25)** ⭐ Recommended | Keseimbangan antara recall dan kecepatan. Memberikan kandidat yang cukup banyak untuk reranker memilih yang terbaik tanpa membebani sistem secara berlebihan. |
| **Besar (30–50)** | Recall tinggi, hampir semua chunk relevan pasti masuk ke kandidat. Namun waktu proses reranking meningkat signifikan karena reranker harus mengevaluasi lebih banyak pasangan query-chunk. |

> 💡 **Catatan:** Nilai ini hanya berpengaruh signifikan jika `USE_RERANKER = True`. Tanpa reranker, yang dipakai langsung hanya `FINAL_TOP_M` teratas.

**Pertanyaan Refleksi:**

* Apakah top-k kecil (5–10) cukup untuk dataset kamu? Jelaskan alasannya.
  > Jawab: Untuk dokumen BL yang digunakan (hanya 1-2 halaman), top-k kecil (5-10) sebenarnya sudah cukup karena total chunk yang dihasilkan tidak banyak. Namun untuk dokumen yang lebih panjang dan kompleks, top-k kecil berisiko melewatkan chunk yang relevan, terutama jika informasi yang dicari tersebar di beberapa bagian dokumen yang tidak berurutan.

* Apa hubungan antara `TOP_K_INITIAL` dan `USE_RERANKER`?
  > Jawab: TOP_K_INITIAL dan USE_RERANKER bekerja secara berpasangan. TOP_K_INITIAL menentukan berapa banyak kandidat awal yang dikumpulkan dari vector search, sedangkan USE_RERANKER menentukan apakah kandidat-kandidat tersebut akan dievaluasi ulang dengan model Cross-Encoder yang lebih akurat. Jika USE_RERANKER=False, TOP_K_INITIAL menjadi kurang relevan karena sistem langsung mengambil FINAL_TOP_M teratas dari hasil vector search tanpa proses reranking.

---

### 📋 3. FINAL_TOP_M

```python
FINAL_TOP_M = 5
```

**Apa itu FINAL_TOP_M?**

Dari sekian banyak kandidat yang ditemukan, hanya **M teratas** (setelah reranking) yang dikirim ke LLM sebagai konteks untuk menjawab pertanyaan.

**Fungsi:**

> Jawab: FINAL_TOP_M berfungsi sebagai filter akhir yang menentukan berapa banyak chunk terbaik yang benar-benar dikirimkan ke LLM sebagai konteks. Setelah TOP_K_INITIAL chunk dikumpulkan dan diurutkan ulang oleh reranker, hanya M chunk dengan skor tertinggi yang dimasukkan ke dalam prompt LLM. Parameter ini langsung mempengaruhi panjang konteks yang diterima LLM dan kualitas jawaban yang dihasilkan.

**Pengaruhnya:**

| Range Nilai | Efek |
| --- | --- |
| **Sedikit (2–3)** | LLM menerima konteks yang sedikit dan sangat terfokus. Jawaban cenderung ringkas dan cepat, namun berisiko melewatkan informasi penting jika jawaban tersebar di beberapa chunk berbeda. |
| **Sedang (4–6)** ⭐ Recommended | Memberikan konteks yang cukup tanpa membebani LLM dengan terlalu banyak informasi. Jawaban lebih lengkap dan akurat dengan latency yang tetap terjaga. |
| **Banyak (7–10)** | LLM mendapat lebih banyak konteks sehingga jawaban bisa lebih komprehensif. Namun latency meningkat karena lebih banyak token yang diproses, dan LLM bisa "kebingungan" jika konteks yang dikirim saling tumpang tindih atau tidak semua relevan. |

> 💡 **Catatan:** Semakin banyak konteks = semakin banyak token yang dikirim ke LLM = latency lebih tinggi. Jaga agar tetap efisien.

**Pertanyaan Refleksi:**

* Benarkah "semakin banyak konteks semakin bagus"? Jelaskan berdasarkan eksperimenmu.
  > Jawab: Tidak selalu. Pada eksperimen dengan FINAL_TOP_M=9 (banyak), generation time justru meningkat (3166ms–4019ms) dibandingkan dengan FINAL_TOP_M=2 yang lebih cepat (1156ms). Selain itu, terlalu banyak konteks bisa membuat LLM memproses informasi yang tidak relevan, berpotensi menghasilkan jawaban yang tidak fokus atau memakan lebih banyak token. Konteks yang sedikit namun sangat relevan lebih efektif daripada konteks yang banyak namun sebagian tidak relevan.

* Apa yang terjadi kalau konteks yang dikirim ke LLM saling bertentangan?
  > Jawab: Jika konteks yang dikirim ke LLM saling bertentangan, LLM bisa menghasilkan jawaban yang ambigu, tidak konsisten, atau bahkan salah karena mencoba menggabungkan informasi yang berlawanan. Dalam kasus RAG, hal ini bisa terjadi jika dokumen memiliki versi data yang berbeda di bagian yang berbeda. Solusinya adalah menurunkan FINAL_TOP_M agar hanya konteks paling relevan yang dikirim, atau meningkatkan SCORE_THRESHOLD agar hanya chunk dengan skor tinggi yang digunakan.

---

### 🎯 4. USE_RERANKER

```python
USE_RERANKER = True  # atau False
```

**Apa itu Reranker?**

Reranker adalah model tambahan (Cross-Encoder) yang mengevaluasi ulang pasangan `[pertanyaan, chunk]` untuk mengurutkan relevansi lebih akurat daripada hanya cosine similarity biasa.

**Fungsi:**

> Jawab: USE_RERANKER berfungsi untuk mengaktifkan atau menonaktifkan tahap reranking setelah vector search. Ketika True, model Cross-Encoder (ms-marco-MiniLM-L-6-v2) mengevaluasi setiap pasangan [pertanyaan, chunk] secara lebih mendalam dibandingkan cosine similarity biasa, sehingga chunk yang benar-benar relevan secara semantik diprioritaskan. Ketika False, sistem langsung mengambil FINAL_TOP_M chunk teratas berdasarkan cosine similarity saja.

**Pengaruhnya:**

| Mode | Efek |
| --- | --- |
| `True` | Presisi dan relevansi konteks meningkat signifikan karena reranker memahami hubungan semantik antara pertanyaan dan chunk secara lebih dalam. Namun latency retrieval meningkat karena ada proses evaluasi tambahan. Pada eksperimen, retrieval dengan reranker aktif mencapai 21964ms untuk 11 chunk. |
| `False` | Proses retrieval lebih cepat (hanya mengandalkan cosine similarity). Pada eksperimen, retrieval tanpa reranker hanya 26ms. Namun relevansi konteks bisa lebih rendah, terutama untuk pertanyaan yang ambigu atau menggunakan kata yang berbeda dari teks di dokumen. |

> 💡 **Catatan:** Untuk dokumen teknis atau pertanyaan yang ambigu, reranker sangat membantu. Untuk kecepatan maksimal, matikan.

**Pertanyaan Refleksi:**

* Pada query jenis apa reranker paling terasa manfaatnya? Berikan contoh.
  > Jawab: Reranker paling terasa manfaatnya pada query yang ambigu atau menggunakan parafrase berbeda dari teks dokumen. Contoh: jika dokumen menyebut "Bill of Lading number: XWSU/SIN/306346-26" namun user bertanya "berapa nomor BL nya?" — kata "BL" tidak secara eksplisit sama dengan "Bill of Lading number" secara cosine similarity, namun reranker memahami bahwa keduanya memiliki hubungan semantik yang kuat. Reranker juga berguna untuk pertanyaan reasoning seperti "kapan kapal berangkat?" yang membutuhkan pemahaman konteks lebih dalam.

* Berapa kira-kira tambahan latency saat reranker dinyalakan di eksperimenmu?
  > Jawab: Berdasarkan eksperimen, perbedaan retrieval time antara USE_RERANKER=True (27.9ms) dan USE_RERANKER=False (26ms) relatif kecil untuk dokumen dengan hanya 1 chunk. Namun pada percobaan dengan 11 chunk (CHUNK_SIZE kecil), retrieval time dengan reranker aktif mencapai 21964ms karena reranker harus mengevaluasi semua pasangan query-chunk. Ini menunjukkan bahwa overhead reranker bergantung pada jumlah chunk yang harus dievaluasi.

---

### 🌡️ 5. TEMPERATURE

```python
TEMPERATURE = 0.2  # range: 0.0 - 1.0
```

**Apa itu Temperature?**

Temperature mengatur distribusi probabilitas saat LLM memilih kata berikutnya. Semakin rendah, semakin "pasti" dan konsisten jawabannya.

**Fungsi:**

> Jawab: TEMPERATURE berfungsi mengontrol seberapa "kreatif" atau "acak" jawaban yang dihasilkan LLM. Nilai rendah membuat LLM memilih kata dengan probabilitas tertinggi secara konsisten (deterministik), menghasilkan jawaban yang faktual dan terprediksi. Nilai tinggi mendistribusikan probabilitas lebih merata sehingga LLM bisa memilih kata yang lebih bervariasi, menghasilkan jawaban yang lebih kreatif namun berisiko tidak akurat.

**Pengaruhnya:**

| Range Nilai | Efek |
| --- | --- |
| **0.0–0.2** ⭐ Recommended untuk RAG | Jawaban sangat konsisten, faktual, dan terprediksi. LLM cenderung mengikuti konteks dokumen dengan ketat tanpa menambahkan informasi di luar konteks. Ideal untuk QA berbasis dokumen karena meminimalkan risiko halusinasi. |
| **0.3–0.5** | Jawaban masih cukup akurat namun mulai ada variasi dalam cara penyampaian. LLM bisa sedikit lebih "kreatif" dalam menyusun kalimat sambil tetap berpegang pada konteks. |
| **0.6–0.8** | Jawaban lebih bervariasi dan kreatif. LLM mulai menambahkan elaborasi yang tidak selalu ada di dokumen. Cocok untuk tugas summarization atau penjelasan konsep, namun kurang tepat untuk QA faktual. |
| **0.9–1.0** | Jawaban sangat acak dan berisiko tinggi menghasilkan halusinasi. LLM bisa mengabaikan konteks dokumen dan menghasilkan informasi yang tidak akurat. Pada eksperimen dengan TEMPERATURE=0.9, generation time juga meningkat karena distribusi probabilitas yang lebih kompleks. |

> ⚠️ **Catatan:** Untuk RAG (jawaban berdasarkan dokumen), gunakan temperature rendah (0.1–0.3) agar jawaban tetap *grounded* pada konteks yang diberikan.

**Pertanyaan Refleksi:**

* Untuk QA berbasis dokumen perusahaan, temperatur ideal berapa? Jelaskan alasannya.
  > Jawab: Temperatur ideal untuk QA berbasis dokumen perusahaan adalah 0.1–0.2. Pada range ini, LLM memberikan jawaban yang konsisten, faktual, dan langsung mengacu pada isi dokumen tanpa menambahkan informasi yang tidak ada. Ini sangat penting untuk dokumen bisnis seperti kontrak, invoice, atau Bill of Lading dimana akurasi data (nomor, tanggal, nama) adalah prioritas utama. Jawaban yang "kreatif" justru berbahaya karena bisa menghasilkan data yang salah.

* Apa yang terjadi pada jawaban saat kamu naikkan temperature ke 0.9?
  > Jawab: Saat temperature dinaikkan ke 0.9, jawaban LLM menjadi lebih bervariasi dalam formulasi kalimat namun berisiko menyimpang dari konteks dokumen. Generation time juga cenderung lebih lama (3166ms–4019ms pada eksperimen) dibandingkan temperature rendah (1156ms). Pada kasus dokumen BL, jawaban masih akurat karena informasinya sangat spesifik, namun untuk dokumen yang lebih kompleks, temperature tinggi bisa menyebabkan LLM "mengarang" detail yang tidak ada di dokumen.

---

### 🤖 6. GROQ_MODEL

```python
GROQ_MODEL = "llama-3.1-8b-instant"  # model default
```

**Model lain yang bisa dicoba:**

```python
# Ganti value GROQ_MODEL ke salah satu di bawah:
GROQ_MODEL = "openai/gpt-oss-120b"
GROQ_MODEL = "llama-3.3-70b-versatile"
```

**Apa bedanya tiap model?**

Setiap model punya ukuran parameter dan kemampuan yang berbeda. Isi tabel di bawah berdasarkan eksperimenmu:

| Model | Ukuran | Pengaruh terhadap Kecepatan & Kualitas |
| --- | --- | --- |
| `llama-3.1-8b-instant` (Default) | 8B parameter | Paling cepat dalam generation (1156ms pada eksperimen). Kualitas jawaban baik untuk pertanyaan sederhana dan faktual. Cocok dipakai saat development, testing cepat, atau saat resource terbatas. Jawaban ringkas dan langsung ke poin. |
| `openai/gpt-oss-120b` | 120B parameter | Generation lebih lambat (3166ms–4019ms pada eksperimen) namun kualitas jawaban lebih detail dan terstruktur. Model ini mampu memahami konteks yang lebih kompleks dan menghasilkan jawaban yang lebih komprehensif. Cocok untuk dokumen teknis atau pertanyaan yang membutuhkan reasoning mendalam. |
| `llama-3.3-70b-versatile` | 70B parameter | Berada di antara 8B dan 120B dalam hal kecepatan dan kualitas. Lebih akurat dari 8B namun lebih cepat dari 120B. Cocok sebagai pilihan production yang menyeimbangkan kualitas dan efisiensi, terutama untuk dokumen dengan kompleksitas sedang. |

> 💡 **Tips:** Mulai dengan `llama-3.1-8b-instant` untuk development. Saat butuh akurasi tinggi, switch ke `llama-3.3-70b-versatile` atau `openai/gpt-oss-120b`.

**Pertanyaan Refleksi:**

* Apakah peningkatan kualitas jawaban dari model besar sebanding dengan penambahan waktu? Jelaskan.
  > Jawab: Tidak selalu sebanding, tergantung kompleksitas pertanyaan. Untuk pertanyaan faktual sederhana seperti "berapa nomor BL?", model 8B (1156ms) sudah memberikan jawaban yang sama akuratnya dengan model 120B (3166ms). Peningkatan kualitas dari model besar lebih terasa pada pertanyaan yang membutuhkan reasoning, summarization, atau pemahaman konteks yang kompleks. Untuk use case RAG dengan dokumen bisnis sederhana, model 8B sudah sangat memadai dan jauh lebih efisien secara biaya dan waktu.

* Model mana yang paling cocok untuk use-case dokumenmu? Mengapa?
  > Jawab: Untuk dokumen Bill of Lading (BL) yang digunakan, `llama-3.1-8b-instant` paling cocok karena dokumen ini berisi data faktual spesifik (nomor, tanggal, nama kapal) yang tidak memerlukan reasoning kompleks. Model 8B sudah mampu mengekstrak dan menjawab pertanyaan faktual dengan akurat dalam waktu yang jauh lebih singkat. Model 120B baru diperlukan jika dokumen lebih kompleks seperti kontrak hukum multi-halaman atau laporan teknis yang membutuhkan analisis mendalam.

---

## ⚙️ Parameter Tambahan (Non-Tunable)

Parameter ini sudah di-set optimal. Ubah hanya jika paham dampaknya.

```python
CHUNK_OVERLAP = 60     # overlap antar chunk (kata)
SCORE_THRESHOLD = 0.30 # skor minimum cosine similarity
```

**Penjelasan:**

| Parameter | Nilai Default | Fungsi |
| --- | --- | --- |
| `CHUNK_OVERLAP` | 60 | Menentukan berapa banyak kata yang tumpang tindih antara chunk yang berurutan. Overlap ini penting untuk menjaga kontinuitas informasi di batas antar chunk sehingga kalimat yang terpotong di akhir satu chunk masih bisa ditemukan di awal chunk berikutnya. Jika terlalu kecil, informasi di batas chunk bisa hilang. Jika terlalu besar, terjadi redundansi data yang membebani storage dan proses embedding. |
| `SCORE_THRESHOLD` | 0.30 | Menentukan nilai minimum cosine similarity yang harus dipenuhi sebuah chunk agar masuk ke kandidat retrieval. Chunk dengan skor di bawah threshold ini dianggap tidak relevan dan dibuang. Jika dinaikkan (misal 0.5), sistem lebih selektif namun berisiko membuang chunk yang sebenarnya relevan. Jika diturunkan (misal 0.1), lebih banyak chunk masuk namun kualitas relevansi bisa turun. |

---

## 📊 Hasil Eksperimen

| Percobaan | CHUNK_SIZE | TOP_K | TOP_M | RERANKER | TEMP | MODEL | Chunks | Retrieval | Generation |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 1 (Kecil) | 100 | 5 | 2 | True | 0.1 | llama-3.1-8b-instant | 11 | 21964ms | 1156ms |
| 2 (Besar) | 700 | 40 | 9 | True | 0.9 | openai/gpt-oss-120b | 1 | 27.9ms | 3166ms |
| 3 (No Reranker) | 700 | 40 | 9 | False | 0.9 | openai/gpt-oss-120b | 1 | 26ms | 4019ms |

**Kesimpulan Eksperimen:**
- CHUNK_SIZE kecil menghasilkan lebih banyak chunk namun retrieval dengan reranker jauh lebih lambat
- CHUNK_SIZE besar mempercepat retrieval namun mengurangi presisi pencarian
- Menonaktifkan reranker mempercepat retrieval namun dapat menurunkan relevansi konteks
- Model besar (120B) menghasilkan jawaban lebih detail namun generation time lebih tinggi
- Temperature tinggi (0.9) meningkatkan variasi jawaban namun berisiko halusinasi

---

## 📝 Tips Eksperimen

* Pakai PDF minimal 1 halaman, maksimal 10 halaman, biar gak berat
* Coba berbagai jenis pertanyaan: **definisi**, **konteks**, dan **reasoning**
* Catat kapan sistem salah & mengapa
* Ubah **satu parameter pada satu waktu** agar bisa melihat pengaruhnya secara jelas
* Prioritaskan kualitas insight dalam laporan

---

## 🔗 Referensi

* Dokumentasi lengkap parameter: buka `docs.html` di browser atau akses via menu **📖 Dokumentasi** di RAG Playground
* Source code: `app.py` — semua parameter ada di bagian `KONFIGURASI INTI`
