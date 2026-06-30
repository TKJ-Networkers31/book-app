# Gunakan image dasar Python
FROM python:3.11-slim

# Instal dependensi sistem yang diperlukan (build dependencies)
RUN apt-get update && apt-get install -y \
  libpq-dev \
  python3-dev \
  default-libmysqlclient-dev \
  build-essential \
  pkg-config \
  cron \
  && apt-get clean && rm -rf /var/lib/apt/lists/*

# Set environment variables
ENV TZ=Asia/Jakarta

# create workdir
WORKDIR /app

# Copy all to workdir
COPY ./requirements.txt ./

# Perbarui pip dan install dependencies
RUN pip install --no-cache-dir --upgrade pip && \
    pip install --no-cache-dir -r /app/requirements.txt --verbose

# Copy all files
COPY . .

# Expose Port
EXPOSE 5001

# Start app.py
CMD ["python", "app.py"]