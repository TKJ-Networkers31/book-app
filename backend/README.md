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
n
## 🛠️ Arsitektur Docker & CI/CD
Proyek ini memisahkan proses pembangunan lingkungan pengembangan dan produksi demi menjaga server utama tetap ringan dan aman.

GitHub Actions (CI/CD Pipeline):
- Melakukan checkout kode secara otomatis saat terjadi push ke branch main.
- Memvalidasi Dockerfile menggunakan Hadolint dengan batas ambang toleransi kesalahan tingkat error.
- Memvalidasi struktur sintaksis file docker-compose.yml 

Multi-stage Dockerfile Optimization:
Stage 1 (Builder): Mengunduh perkakas kompilasi sistem.
Stage 2 (Runtime): Hanya menyalin paket python matang dari Stage 1 tanpa menyertakan sampah kompilasi. Ditambahkan tools curl untuk mekanisme healthcheck.

## 💻 Cara Menjalankan di Server (Deployment)
Prasyarat
1. Docker dan Docker Compose telah terpasang di server.
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

## 🔍 Pemeriksaan Kesehatan Kontainer (Healthcheck)
Kontainer backend dikonfigurasi untuk memeriksa kesehetannya sendiri setiap 30 detik ke endpoint internal Flask /api/test. Untuk memantau status kesehatan kontainer dari terminal server, jalankan perintah:
Bash
docker ps
Output Valid:
Status kontainer akan menunjukkan indikasi (healthy) yang menandakan aplikasi berjalan normal dan lulus uji konektivitas internal:
STATUS
Up 10 minutes (healthy)

## 💻 Panduan Menjalankan Secara Lokal (Docker CLI Manual)

Jika kamu ingin melakukan pengujian, *build*, atau menjalankan kontainer di komputer lokal/VPS tanpa menggunakan Docker Compose, kamu bisa menggunakan perintah manual Docker CLI berikut.

1. Build Image Secara Lokal
Masuk ke folder tempat `Dockerfile` berada, lalu jalankan perintah `docker build`. Jangan lupa berikan tag nama agar mudah dipanggil.

# Masuk ke folder backend
bash
cd backend

# Build image dengan nama 'book-backend' dan tag 'local'
docker build -t book-backend:local .

2. Jalankan Kontainer Secara Lokal
Agar aplikasi backend bisa terhubung dengan database PostgreSQL tanpa Docker Compose, kedua kontainer harus dimasukkan ke dalam satu jaringan (network) Docker yang sama.
- Langkah A: Buat Jaringan Baru (Docker Network)
Bash
docker network create book-network-local
- Langkah B: Jalankan Kontainer Database (PostgreSQL)
Nyalakan kontainer database dan hubungkan ke network yang baru dibuat:

Bash
docker run -d \
  --name postgres-local \
  --network book-network-local \
  -e POSTGRES_USER=admin_buku \
  -e POSTGRES_PASSWORD=RahasiaLokal123! \
  -e POSTGRES_DB=booktracker_local \
  -v postgres_local_data:/var/lib/postgresql/data \
  --restart unless-stopped \
  postgres:16
Langkah C: Jalankan Kontainer Aplikasi Backend (Flask)
Nyalakan aplikasi backend, hubungkan ke network yang sama, arahkan portnya, dan arahkan DATABASE_URL ke nama kontainer database (postgres-local):

Bash
docker run -d \
  --name backend-local \
  --network book-network-local \
  -p 5001:5001 \
  -e DATABASE_URL=postgresql://admin_buku:RahasiaLokal123!@postgres-local:5432/booktracker_local \
  -e FLASK_PORT=5001 \
  --restart unless-stopped \
  book-backend:local

## 🛑 Cara Mematikan dan Menghapus Kontainer Lokal
Jika proses pengujian lokal sudah selesai dan kamu ingin membersihkan kontainer yang berjalan:

Bash
# Menghentikan kontainer
docker stop backend-local postgres-local

# Menghapus kontainer
docker rm backend-local postgres-local

# Menghapus network (opsional)
docker network rm book-network-local
