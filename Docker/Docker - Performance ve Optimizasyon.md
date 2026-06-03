---
tarih: 2025-01-01
konu: Docker Performance, Build Cache, Image Boyutu, BuildKit
etiket: [docker, performans, optimizasyon, build-cache, buildkit, slim]
kaynak:
zorluk: ⭐⭐⭐
---

## 📌 Özet

Docker performans optimizasyonu iki ana başlıkta incelenir: Build süresi (cache kullanımı, BuildKit, layer sırası) ve runtime performansı (kaynak sınırları, image boyutu).

---

## 🧠 Detay

### Build Cache Nasıl Çalışır?

```
Her katman SHA256 hash ile cache'lenir.
Değişen katmandan itibaren TÜM katmanlar yeniden build edilir.

# ❌ Kötü sıralama — kod her değişimde pip çalışır
COPY . .                          # Layer 3
RUN pip install -r requirements.txt  # Layer 4 — her seferinde!

# ✅ İyi sıralama — pip sadece requirements değişince çalışır
COPY requirements.txt .           # Layer 3
RUN pip install -r requirements.txt  # Layer 4 — cache'den gelir
COPY . .                          # Layer 5 — sadece bu yeniden
```

### Cache Geçersiz Kılma Durumları

```dockerfile
# Şunlar cache'i geçersiz kılar:
# 1. FROM image değişikliği (yeni digest)
# 2. RUN komutunun kendisi değişirse
# 3. COPY / ADD kaynak dosyaların içeriği değişirse
# 4. ARG değeri değişirse (ARG sonrası tüm layerlar)

# Cache'i zorla geçersiz kıl
docker build --no-cache -t myapp .

# Sadece belirli aşamadan geçersiz kıl
ARG CACHE_BUST=1
RUN apt-get update    # Bu satır her build'de çalışır (ARG değişince)
```

### BuildKit Aktifleştirme

```bash
# Ortam değişkeni ile
DOCKER_BUILDKIT=1 docker build -t myapp .

# Global aktifleştirme (/etc/docker/daemon.json)
{
  "features": { "buildkit": true }
}

# Docker Compose ile
COMPOSE_DOCKER_CLI_BUILD=1 docker compose build
```

**BuildKit Avantajları:**
- Paralel katman build
- Daha akıllı cache
- Secret mount (image'a yazmadan)
- SSH agent forwarding
- Build progress görünümü

### BuildKit Özellikleri

```dockerfile
# syntax=docker/dockerfile:1

# Cache mount — pip cache'i build'ler arası koru
RUN --mount=type=cache,target=/root/.cache/pip \
    pip install -r requirements.txt

# npm cache
RUN --mount=type=cache,target=/root/.npm \
    npm ci --cache /root/.npm

# Secret mount — image'a yazılmaz
RUN --mount=type=secret,id=npmrc,dst=/root/.npmrc \
    npm install

# SSH agent forwarding (private repo)
RUN --mount=type=ssh \
    git clone git@github.com:company/private-repo.git

# Bind mount (COPY yapmadan okuma)
RUN --mount=type=bind,source=.,target=/src \
    cd /src && go build ./...
```

```bash
# BuildKit ile secret kullanım
docker build \
  --secret id=npmrc,src=$HOME/.npmrc \
  --ssh default \
  -t myapp .
```

### Image Boyutu Azaltma

```dockerfile
# 1. Minimal base image seç
FROM python:3.11-slim    # 130MB (python:3.11 = 1GB)
FROM python:3.11-alpine  # 52MB (musl libc — bazı paketlerde sorun)

# 2. Tek RUN'da kur ve temizle
RUN apt-get update && apt-get install -y \
    libpq-dev \
    && rm -rf /var/lib/apt/lists/*  # APT cache sil

# 3. pip cache kullanma
RUN pip install --no-cache-dir -r requirements.txt

# 4. .dockerignore
# (test/, docs/, .git/, *.md dışla)

# 5. Multi-stage build ile dev bağımlılıklarını çıkar
```

### Layer Sayısını Azaltma

```dockerfile
# ❌ Her biri ayrı layer
RUN apt-get update
RUN apt-get install -y curl
RUN apt-get install -y git
RUN rm -rf /var/lib/apt/lists/*

# ✅ Tek layer
RUN apt-get update \
    && apt-get install -y curl git \
    && rm -rf /var/lib/apt/lists/*
```

### Dive ile Layer Analizi

```bash
# Image katman analizi
docker run --rm -it \
  -v /var/run/docker.sock:/var/run/docker.sock \
  wagoodman/dive myapp:latest

# CLI çıktısı:
# Layer  Size   Efficiency
# 0      77MB   -
# 1      130MB  0.9 (iyi)
# 2      45MB   0.4 (kötü — gereksiz dosyalar var!)
```

### docker-slim ile Otomatik Küçültme

```bash
# Image'ı otomatik analiz et ve küçült
docker-slim build myapp:latest

# Sonuç: myapp.slim
# Orijinal: 500MB → Slim: 20MB (25x küçülme)

# Test ile
docker-slim build --http-probe myapp:latest
```

### Build Performansı Karşılaştırma

```bash
# Temel build süresi ölç
time docker build -t myapp .

# BuildKit ile paralel build
time DOCKER_BUILDKIT=1 docker build -t myapp .

# Cache kullanım raporu
docker build --progress=plain -t myapp . 2>&1 | grep "CACHED"
```

### Runtime Kaynak Yönetimi

```bash
# CPU ve bellek sınırla
docker run \
  --cpus="2.0" \                  # 2 CPU core
  --cpu-period=100000 \           # 100ms periyot
  --cpu-quota=50000 \             # 50ms = %50 CPU
  --memory="512m" \               # Max RAM
  --memory-swap="512m" \          # Swap yok (=memory → swap kapalı)
  --memory-reservation="256m" \   # Soft limit
  --oom-kill-disable \            # OOM killer devre dışı
  --pids-limit=100 \              # Max süreç
  myapp

# Mevcut kullanım
docker stats --no-stream
```

### Build Argümanları ile Ortam Farklılaştırma

```dockerfile
ARG ENVIRONMENT=production
ARG PYTHON_VERSION=3.11

FROM python:${PYTHON_VERSION}-slim AS base

ARG ENVIRONMENT
RUN if [ "$ENVIRONMENT" = "development" ]; then \
      pip install pytest pytest-cov; \
    fi

ENV APP_ENV=$ENVIRONMENT
```

```bash
docker build --build-arg ENVIRONMENT=development -t myapp:dev .
docker build --build-arg ENVIRONMENT=production  -t myapp:prod .
```

### Dockerfile Lint (Hadolint)

```bash
# Dockerfile'ı lint et
docker run --rm -i hadolint/hadolint < Dockerfile

# Yaygın uyarılar:
# DL3008: Pin versions in apt-get install
# DL3009: Delete apt-get cache
# DL3013: Pin pip packages
# DL4006: Set SHELL -o pipefail
```

### Optimizasyon Kontrol Listesi

- [ ] requirements.txt / package.json önce kopyalandı mı?
- [ ] APT/pip cache temizlendi mi?
- [ ] Multi-stage build kullanıldı mı?
- [ ] .dockerignore tanımlı mı?
- [ ] slim/alpine base image kullanıldı mı?
- [ ] BuildKit aktif mi?
- [ ] Gereksiz katmanlar birleştirildi mi?
- [ ] Dive ile katman analizi yapıldı mı?

---

## 💡 Bağlantılar
- [[Docker - Dockerfile Yazımı]]
- [[Docker - Registry ve Image Yönetimi]]
- [[Docker - Güvenlik Best Practices]]

## ❓ Sorular / Anlamadıklarım

## 🔗 Kaynaklar
- docs.docker.com/develop/develop-images/dockerfile_best-practices/
- github.com/wagoodman/dive
