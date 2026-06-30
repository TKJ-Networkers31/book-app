# 🚀 Fitur Utama

- **Production-Ready WSGI Server**: Menggunakan **Gunicorn** dengan arsitektur multi-worker untuk efisiensi performa tinggi.
- **Multi-stage Docker Build**: Memisahkan tahap kompilasi dependencies (*Builder*) dan runtime untuk meminimalkan ukuran Docker Image akhir.
- **Automated Linting**: Integrasi **Hadolint** di GitHub Actions untuk mendeteksi kesalahan penulisan Dockerfile sejak dini.
- **Healthcheck Built-in**: Kontainer backend dilengkapi instruksi `HEALTHCHECK` otomatis untuk memantau status keaktifan aplikasi secara real-time.
- **Database Resilience**: Konfigurasi `depends_on` dengan `service_healthy` memastikan backend hanya berjalan setelah PostgreSQL siap menerima koneksi.

---

## 📂 Struktur Direktori Proyek

```text
📂 book-app (Root / Repositori Utama)
 ┣ 📂 .github
 ┃ ┗ 📂 workflows
 ┃   ┗ 📄 ci.yml               # Pipeline GitHub Actions (Hadolint & Build Validation)
 ┣ 📂 backend
 ┃ ┣ 📄 app.py                 # File utama aplikasi Flask API
 ┃ ┣ 📄 Dockerfile             # Dockerfile Multi-stage
 ┃ ┣ 📄 docker-compose.yml     # Konfigurasi orkestrasi kontainer (Backend & Postgres)
 ┃ ┗ 📄 requirements.txt       # Daftar dependencies Python (Flask, Gunicorn, psycopg2, dll.)
 ┗ 📄 .gitignore               # Aturan pengabaian berkas Git (Mengamankan file .env)

🛠️ Arsitektur Docker & CI/CD
Proyek ini memisahkan proses pembangunan lingkungan pengembangan dan produksi demi menjaga server utama tetap ringan dan aman.

GitHub Actions (CI/CD Pipeline):
- Melakukan checkout kode secara otomatis saat terjadi push ke branch main.
- Memvalidasi Dockerfile menggunakan Hadolint dengan batas ambang toleransi kesalahan tingkat error.
- Memvalidasi struktur sintaksis berkas docker-compose.yml lewat instruksi config.

Multi-stage Dockerfile Optimization:
Stage 1 (Builder): Mengunduh perkakas kompilasi sistem.
Stage 2 (Runtime): Hanya menyalin paket python matang dari Stage 1 tanpa menyertakan sampah kompilasi. Ditambahkan tools curl untuk mekanisme healthcheck.

💻 Cara Menjalankan di Server (Deployment)
Prasyarat
1. Docker dan Docker Compose telah terpasang di server VPS.
2. Repositori telah di-clone ke server pada direktori /root/.

Langkah-Langkah Pemasangan:
Masuk ke Direktori Kerja Backend

Bash
cd /root/book-app/backend
Konfigurasi Environment Variables
Buat berkas .env di dalam folder backend untuk mengamankan data kredensial database (Berkas ini telah masuk ke dalam aturan .gitignore agar tidak bocor ke publik):

Bash
nano .env
Tambahkan variabel berikut:

Code snippet
POSTGRES_USER=username
POSTGRES_PASSWORD=password_rahasia
POSTGRES_DB=booktracker_prod
PORT_HOST=5001
PORT_APP=5001

Bash
docker compose up -d --build

Memperbarui Aplikasi Secara Aman (Seamless Update)
Jika Anda melakukan perubahan kode di kemudian hari, Anda dapat memperbarui kontainer backend tanpa mengganggu data atau mematikan database PostgreSQL:
Bash
docker compose up -d --build backend

🔍 Pemeriksaan Kesehatan Kontainer (Healthcheck)
Kontainer backend dikonfigurasi untuk memeriksa kesehetannya sendiri setiap 30 detik ke endpoint internal Flask /api/test. Untuk memantau status kesehatan kontainer dari terminal server, jalankan perintah:
Bash
docker ps
Output Valid:
Status kontainer akan menunjukkan indikasi (healthy) yang menandakan aplikasi berjalan normal dan lulus uji konektivitas internal:

Plaintext
STATUS
Up 10 minutes (healthy)

🛡️ Catatan Keamanan Produksi
Jangan Pernah Mengubah Aturan .gitignore: Pastikan berkas .env tidak dipaksa masuk ke dalam commit Git.

Gunicorn WSGI: Peringatan "WARNING: This is a development server" tidak akan muncul kembali karena aplikasi dijalankan di atas server Gunicorn berkekuatan penuh melalui perintah:

Plaintext
["gunicorn", "-w", "4", "-b", "0.0.0.0:5001", "app:app"]
Layanan Auto-Start: Seluruh kontainer dikonfigurasi dengan kebijakan restart: unless-stopped sehingga aplikasi otomatis menyala kembali jika server VPS mengalami restart atau crash tidak terduga.
