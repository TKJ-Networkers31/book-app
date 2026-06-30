# Stage 1 : Builder
FROM python:3.11-slim AS builder

# Install build dependencies
RUN apt-get update && apt-get install -y \
    build-essential \
    python3-dev \
    default-libmysqlclient-dev \
    libpq-dev \
    pkg-config \
 && rm -rf /var/lib/apt/lists/*

WORKDIR /app

# Install Python packages
COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

# Stage 2 : Runtime
FROM python:3.11-slim

ENV TZ=Asia/Jakarta

WORKDIR /app

# Copy Python packages dari builder
COPY --from=builder /usr/local/lib/python3.11 /usr/local/lib/python3.11
COPY --from=builder /usr/local/bin /usr/local/bin

# Copy source code
COPY . .

RUN apt-get update && apt-get install -y --no-install-recommends \
    curl \
    libpq-dev \
    && rm -rf /var/lib/apt/lists/*

EXPOSE 5001

HEALTHCHECK --timeout=20s --interval=30s --retries=3 CMD curl -f http://localhost:5001/api/test || exit 1

CMD ["gunicorn", "-w", "4", "-b", "0.0.0.0:5001", "app:app"]
