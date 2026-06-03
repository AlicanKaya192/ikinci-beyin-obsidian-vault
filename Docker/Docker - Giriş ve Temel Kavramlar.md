---
tarih: 2025-01-01
konu: Docker'a Giriş, Temel Kavramlar, VM vs Container
etiket: [docker, konteyner, image, container, temel]
kaynak:
zorluk: ⭐
---

## 📌 Özet

Docker, uygulamaları bağımlılıklarıyla birlikte izole edilmiş "konteyner" içinde paketleyen platformdur. "Bende çalışıyor" sorununu ortadan kaldırır.

---

## 🧠 Detay

### VM vs Container

```
VM                          Container
┌─────────────────┐         ┌─────────────────┐
│   Uygulama A    │         │  App A │  App B  │
│   Guest OS      │         ├────────┴─────────┤
│   Hypervisor    │         │  Docker Engine   │
│   Host OS       │         │  Host OS         │
│   Donanım       │         │  Donanım         │
└─────────────────┘         └─────────────────┘
GB'larca, dakikalar         MB'larca, saniyeler
```

| Özellik | VM | Container |
|---|---|---|
| Boyut | GB | MB |
| Başlatma | Dakikalar | Saniyeler |
| İzolasyon | Tam (kendi OS) | Süreç düzeyinde |
| Kaynak kullanım | Yüksek | Düşük |

### Temel Kavramlar

| Kavram | Açıklama |
|---|---|
| **Image** | Salt okunur şablon. Uygulamanın mavi kopyası |
| **Container** | Image'ın çalışan örneği |
| **Dockerfile** | Image oluşturmak için talimat dosyası |
| **Registry** | Image deposu (Docker Hub, ECR, GCR) |
| **Volume** | Kalıcı veri depolama |
| **Network** | Konteynerler arası iletişim |
| **Layer** | Her Dockerfile komutu bir katman oluşturur |

### Docker Mimarisi

```
┌──────────────────────────────────────────┐
│  Docker Client (CLI)                     │
│  docker build / run / pull / push        │
└──────────────┬───────────────────────────┘
               │ REST API
┌──────────────▼───────────────────────────┐
│  Docker Daemon (dockerd)                  │
│  ┌──────────┐  ┌──────────┐  ┌────────┐ │
│  │ Images   │  │Containers│  │Volumes │ │
│  └──────────┘  └──────────┘  └────────┘ │
└──────────────────────────────────────────┘
               │
┌──────────────▼───────────────────────────┐
│  Docker Registry (Docker Hub)            │
└──────────────────────────────────────────┘
```

### Kurulum Kontrolü

```bash
docker --version
docker info
docker run hello-world
```

### Image Katmanları

```
FROM python:3.11          # Layer 1: Base image
RUN apt-get update        # Layer 2: OS paketleri
COPY requirements.txt .   # Layer 3: Requirements
RUN pip install -r req..  # Layer 4: Python paketleri
COPY . .                  # Layer 5: Uygulama kodu
```

Her katman cache'lenir. Değişen katmandan sonrakiler yeniden build edilir.

---

## 💡 Bağlantılar
- [[Docker - Temel Komutlar]]
- [[Docker - Dockerfile Yazımı]]
- [[Docker - Docker Compose]]

## ❓ Sorular / Anlamadıklarım

## 🔗 Kaynaklar
- docs.docker.com/get-started
- Play with Docker (labs.play-with-docker.com)
