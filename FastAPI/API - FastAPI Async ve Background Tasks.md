---
tarih: 2026-05-28
konu: FastAPI
etiket: ["fastapi", "async", "background-task", "celery", "performans"]
kaynak: FastAPI Dokümantasyon
zorluk: orta
---

## 📌 Özet
FastAPI async/await ile yüksek eşzamanlılık sağlar. Arka plan görevleri ise uzun işlemleri (model eğitimi, e-posta gönderimi) istek döngüsü dışına taşır.

## 🧠 Detay

### Async Endpoint
```python
import asyncio
import httpx
from fastapi import FastAPI

app = FastAPI()

# Sync endpoint (CPU-yoğun işlemler için)
@app.post("/tahmin")
def sync_tahmin(giris: TahminGiris):
    sonuc = model.predict(giris.veri)
    return {"tahmin": sonuc}

# Async endpoint (I/O işlemleri için: DB, HTTP, dosya)
@app.get("/harici-veri")
async def async_veri():
    async with httpx.AsyncClient() as client:
        yanit = await client.get("https://api.example.com/data")
    return yanit.json()

# Birden fazla async işlem paralel
@app.get("/coklu-istek")
async def coklu_istek():
    async with httpx.AsyncClient() as client:
        sonuclar = await asyncio.gather(
            client.get("https://api1.com/data"),
            client.get("https://api2.com/data"),
            client.get("https://api3.com/data")
        )
    return [s.json() for s in sonuclar]
```

### Background Tasks
```python
from fastapi import FastAPI, BackgroundTasks
import logging

app = FastAPI()

def log_kaydet(istek_id: str, giris: dict, sonuc: dict):
    """Arka planda çalışır, isteği bekletmez"""
    logging.info(f"ID: {istek_id} | Giriş: {giris} | Sonuç: {sonuc}")
    # DB'ye yaz, e-posta gönder vs.

def model_metrikleri_guncelle(tahmin: int, olasilik: float):
    """Model metriklerini güncelle"""
    # monitoring sistemine gönder
    pass

@app.post("/tahmin")
def tahmin_yap(
    giris: TahminGiris,
    background_tasks: BackgroundTasks
):
    sonuc = model.predict([[giris.yas, giris.gelir]])[0]
    olasilik = model.predict_proba([[giris.yas, giris.gelir]])[0].max()

    # Arka plan görevleri ekle (yanıt döndükten sonra çalışır)
    background_tasks.add_task(log_kaydet, "req-123", giris.dict(), {"tahmin": sonuc})
    background_tasks.add_task(model_metrikleri_guncelle, sonuc, olasilik)

    return {"tahmin": int(sonuc), "olasilik": float(olasilik)}
```

### Uzun Süren İşlemler (Async Pattern)
```python
import uuid
from typing import Dict

gorev_durumu: Dict[str, dict] = {}

@app.post("/model-egit")
async def model_egit_baslat(background_tasks: BackgroundTasks):
    gorev_id = str(uuid.uuid4())
    gorev_durumu[gorev_id] = {"durum": "bekliyor", "ilerleme": 0}

    background_tasks.add_task(model_egit_arka, gorev_id)

    return {"gorev_id": gorev_id, "durum_url": f"/gorev/{gorev_id}"}

async def model_egit_arka(gorev_id: str):
    gorev_durumu[gorev_id]["durum"] = "çalışıyor"
    # ... uzun işlem
    for i in range(100):
        await asyncio.sleep(0.1)
        gorev_durumu[gorev_id]["ilerleme"] = i + 1
    gorev_durumu[gorev_id]["durum"] = "tamamlandı"

@app.get("/gorev/{gorev_id}")
def gorev_durumu_sorgula(gorev_id: str):
    if gorev_id not in gorev_durumu:
        raise HTTPException(404, "Görev bulunamadı")
    return gorev_durumu[gorev_id]
```

### Celery ile Dağıtık Görevler
```bash
pip install celery redis
```

```python
from celery import Celery

celery_app = Celery("tasks", broker="redis://localhost:6379/0")

@celery_app.task
def agir_tahmin(veri: list):
    sonuc = buyuk_model.predict(veri)
    return sonuc.tolist()

# FastAPI'den tetikle
@app.post("/tahmin/agir")
def agir_tahmin_baslat(giris: TahminGiris):
    gorev = agir_tahmin.delay([[giris.yas, giris.gelir]])
    return {"task_id": gorev.id}

@app.get("/tahmin/sonuc/{task_id}")
def tahmin_sonucu_al(task_id: str):
    gorev = agir_tahmin.AsyncResult(task_id)
    return {"durum": gorev.status, "sonuc": gorev.result}
```

## 💡 Bağlantılar
- [[API - FastAPI Giriş]]
- [[API - ML Modeli Servis Etmek]]
- [[API - FastAPI Loglama ve İzleme]]

## ❓ Sorular / Anlamadıklarım
- CPU-yoğun işlemlerde async ne zaman yararlı değil?
- Background Tasks ile Celery arasındaki temel fark?

## 🔗 Kaynaklar
- https://fastapi.tiangolo.com/tutorial/background-tasks/
