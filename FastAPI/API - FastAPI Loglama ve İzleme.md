---
tarih: 2026-05-28
konu: FastAPI
etiket: ["fastapi", "loglama", "monitoring", "prometheus", "logging"]
kaynak: FastAPI Dokümantasyon
zorluk: orta
---

## 📌 Özet
Production'daki ML API'larını izlemek için loglama ve metrik toplama kritiktir. Hata ayıklama, performans takibi ve model drift tespiti için kullanılır.

## 🧠 Detay

### Yapılandırılmış Loglama
```python
import logging
import json
from datetime import datetime
from fastapi import FastAPI, Request

# JSON formatlı log
class JSONFormatter(logging.Formatter):
    def format(self, record):
        log_data = {
            "zaman": datetime.utcnow().isoformat(),
            "seviye": record.levelname,
            "mesaj": record.getMessage(),
            "modul": record.module,
            "satir": record.lineno
        }
        if hasattr(record, "extra"):
            log_data.update(record.extra)
        return json.dumps(log_data, ensure_ascii=False)

# Logger kur
logger = logging.getLogger("ml_api")
logger.setLevel(logging.INFO)

handler = logging.StreamHandler()
handler.setFormatter(JSONFormatter())
logger.addHandler(handler)

# Dosyaya da yaz
file_handler = logging.FileHandler("logs/api.log")
file_handler.setFormatter(JSONFormatter())
logger.addHandler(file_handler)
```

### İstek/Yanıt Loglama
```python
import time, uuid

@app.middleware("http")
async def istek_yanit_logla(request: Request, call_next):
    istek_id = str(uuid.uuid4())[:8]
    baslangic = time.time()

    logger.info("İstek geldi", extra={
        "istek_id": istek_id,
        "method": request.method,
        "yol": request.url.path,
        "ip": request.client.host
    })

    response = await call_next(request)
    sure_ms = round((time.time() - baslangic) * 1000, 2)

    logger.info("Yanıt gönderildi", extra={
        "istek_id": istek_id,
        "durum_kodu": response.status_code,
        "sure_ms": sure_ms
    })

    return response
```

### ML Tahmin Loglama
```python
@app.post("/tahmin")
def tahmin_yap(giris: TahminGiris):
    baslangic = time.time()

    tahmin = ml_modeller["model"].predict([[giris.yas, giris.gelir]])[0]
    olasilik = ml_modeller["model"].predict_proba([[giris.yas, giris.gelir]])[0].max()

    sure_ms = round((time.time() - baslangic) * 1000, 2)

    # Tahmin logla (model monitoring için)
    logger.info("Tahmin yapıldı", extra={
        "giris": giris.dict(),
        "tahmin": int(tahmin),
        "olasilik": float(olasilik),
        "sure_ms": sure_ms,
        "model_versiyonu": "1.0.0"
    })

    return {"tahmin": int(tahmin), "olasilik": float(olasilik)}
```

### Prometheus Metrikleri
```bash
pip install prometheus-fastapi-instrumentator
```

```python
from prometheus_fastapi_instrumentator import Instrumentator
from prometheus_client import Counter, Histogram, Gauge

# Otomatik HTTP metrikleri
Instrumentator().instrument(app).expose(app)

# Özel ML metrikleri
tahmin_sayaci = Counter(
    "ml_tahmin_toplam",
    "Toplam tahmin sayısı",
    ["model", "karar"]
)

tahmin_suresi = Histogram(
    "ml_tahmin_suresi_saniye",
    "Tahmin süresi",
    buckets=[0.001, 0.005, 0.01, 0.05, 0.1, 0.5]
)

model_olasiligi = Histogram(
    "ml_tahmin_olasiligi",
    "Tahmin olasılık dağılımı",
    buckets=[0.5, 0.6, 0.7, 0.8, 0.9, 0.95, 0.99]
)

@app.post("/tahmin")
def tahmin_yap(giris: TahminGiris):
    with tahmin_suresi.time():
        tahmin = ml_modeller["model"].predict([[giris.yas]])[0]
        olasilik = ml_modeller["model"].predict_proba([[giris.yas]])[0].max()

    tahmin_sayaci.labels(model="rf_v1", karar=str(tahmin)).inc()
    model_olasiligi.observe(float(olasilik))

    return {"tahmin": int(tahmin), "olasilik": float(olasilik)}
```

### Sağlık Endpoint'leri
```python
from fastapi.responses import JSONResponse

@app.get("/saglik")
def saglik():
    return {"durum": "sağlıklı", "zaman": datetime.utcnow().isoformat()}

@app.get("/hazir")
def hazirlik():
    if "model" not in ml_modeller:
        return JSONResponse(status_code=503, content={"durum": "hazır değil"})
    return {"durum": "hazır", "model": "yüklü"}
```

## 💡 Bağlantılar
- [[API - FastAPI Middleware ve Güvenlik]]
- [[API - Docker ile Deploy]]
- [[API - FastAPI Test Yazma]]

## ❓ Sorular / Anlamadıklarım
- Model drift nasıl izlenir?
- Prometheus + Grafana nasıl kurulur?

## 🔗 Kaynaklar
- https://github.com/trallnag/prometheus-fastapi-instrumentator
