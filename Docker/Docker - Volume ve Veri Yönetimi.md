---
tarih: 2025-01-01
konu: Docker Volume, Bind Mount, tmpfs, Veri Kalıcılığı
etiket: [docker, volume, bind-mount, veri, kalıcılık, tmpfs]
kaynak:
zorluk: ⭐⭐
---

## 📌 Özet

Container'lar ephemeral (geçici) dir — silinince veri gider. Volumes ile veri kalıcı hale getirilir. Üç tür mount vardır: Volume, Bind Mount, tmpfs.

---

## 🧠 Detay

### Mount Türleri

```
Host Filesystem          Docker Area
┌────────────────┐       ┌────────────────────────┐
│                │       │  Container              │
│  /home/user/   │◄──────│  Bind Mount             │
│  my-project/   │       │  (host path)            │
│                │       ├────────────────────────┤
│                │       │  /app/data              │
│  Docker Area   │◄──────│  Named Volume           │
│  /var/lib/     │       │  (docker managed)       │
│  docker/volumes│       ├────────────────────────┤
│                │       │  /tmp/cache             │
│  RAM (tmpfs)   │◄──────│  tmpfs Mount            │
└────────────────┘       └────────────────────────┘
```

| Tür | Veri Yeri | Kullanım |
|---|---|---|
| **Named Volume** | Docker yönetir | Prod DB, kalıcı veri |
| **Bind Mount** | Host dizini | Geliştirme, canlı kod |
| **tmpfs** | RAM | Geçici, hassas veri |

### Named Volume

```bash
# Volume oluştur
docker volume create mydata

# Volume listele
docker volume ls

# Volume detay
docker volume inspect mydata

# Volume sil
docker volume rm mydata
docker volume prune    # Kullanılmayanları sil

# Volume ile container çalıştır
docker run -v mydata:/app/data nginx
docker run --mount type=volume,source=mydata,target=/app/data nginx
```

### Bind Mount

```bash
# Geliştirmede: host kodu → container
docker run -v $(pwd):/app python:3.11 python app.py
docker run -v /absolute/path:/container/path nginx

# Salt okunur
docker run -v $(pwd)/config:/app/config:ro nginx

# --mount sözdizimi (daha açık)
docker run --mount type=bind,source=$(pwd),target=/app python:3.11 python app.py
```

### tmpfs Mount

```bash
# Sadece Linux
docker run --tmpfs /tmp:rw,size=64m nginx
docker run --mount type=tmpfs,destination=/tmp,tmpfs-size=64m nginx
```

Hassas veriler (şifreler, token'lar) için disk yazımını önler.

### Docker Compose'da Volume

```yaml
services:
  db:
    image: postgres:15
    volumes:
      # Named volume
      - postgres_data:/var/lib/postgresql/data

      # Bind mount (geliştirme)
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql:ro

      # tmpfs
      - type: tmpfs
        target: /tmp

  app:
    volumes:
      # Canlı kod (geliştirme)
      - .:/app
      # Named volume
      - static_files:/app/static
      # Salt okunur config
      - ./config.yml:/app/config.yml:ro

volumes:
  postgres_data:
    driver: local
  static_files:
    driver: local
  
  # Harici volume (zaten var)
  existing_volume:
    external: true
```

### Volume Yedekleme

```bash
# Volume yedekle
docker run --rm \
  -v mydata:/source:ro \
  -v $(pwd):/backup \
  alpine tar czf /backup/mydata_backup.tar.gz -C /source .

# Volume geri yükle
docker run --rm \
  -v mydata:/target \
  -v $(pwd):/backup:ro \
  alpine tar xzf /backup/mydata_backup.tar.gz -C /target

# Container'dan veri kopyala
docker cp mycontainer:/app/data ./local_backup/
```

### Volume Driver'ları

```yaml
volumes:
  # Local (varsayılan)
  local_vol:
    driver: local

  # NFS
  nfs_vol:
    driver: local
    driver_opts:
      type: nfs
      o: addr=192.168.1.100,rw
      device: ":/path/on/nfs"

  # Tmpfs
  tmp_vol:
    driver: local
    driver_opts:
      type: tmpfs
      device: tmpfs
      o: size=100m
```

### Veri Akışı Diyagramı

```
Geliştirme:
┌─────────────┐  Bind Mount   ┌─────────────┐
│  Host Code  │◄─────────────►│  Container  │
│  ./app/     │               │  /app/      │
└─────────────┘               └─────────────┘
  Değişiklik anında yansır

Production:
┌─────────────┐               ┌─────────────┐
│Docker Volume│◄─────────────►│  Container  │
│ postgres_   │               │/var/lib/pg/ │
│    data/    │               └─────────────┘
└─────────────┘
  Container silinse de veri kalır
```

### Best Practices

```bash
# ✅ Named volume: production DB
docker run -v postgres_data:/var/lib/postgresql/data postgres

# ✅ Bind mount: geliştirme
docker run -v $(pwd):/app -p 8000:8000 myapp

# ✅ Salt okunur config
docker run -v ./config.yml:/app/config.yml:ro myapp

# ❌ Root volume bağlama — güvenlik riski
docker run -v /:/host ubuntu  # Asla yapma!

# ✅ Kullanılmayan volumeleri temizle
docker volume prune
```

---

## 💡 Bağlantılar
- [[Docker - Temel Komutlar]]
- [[Docker - Docker Compose]]
- [[Docker - Network Yönetimi]]

## ❓ Sorular / Anlamadıklarım

## 🔗 Kaynaklar
- docs.docker.com/storage/volumes/
- docs.docker.com/storage/bind-mounts/
