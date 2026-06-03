---
tarih: 2026-05-28
konu: FastAPI
etiket: ["fastapi", "sqlalchemy", "veritabanı", "orm", "postgresql"]
kaynak: FastAPI / SQLAlchemy Dokümantasyon
zorluk: orta
---

## 📌 Özet
FastAPI'da SQLAlchemy ORM veya async için SQLModel kullanarak veritabanı entegrasyonu yapılır. Tahmin logları, kullanıcı yönetimi ve model metadata saklamak için gereklidir.

## 🧠 Detay

### Kurulum
```bash
pip install sqlalchemy psycopg2-binary alembic
# veya async için
pip install sqlalchemy[asyncio] asyncpg
```

### SQLAlchemy Modeli
```python
# database.py
from sqlalchemy import create_engine, Column, Integer, Float, String, DateTime, Boolean
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker
from datetime import datetime
import os

DATABASE_URL = os.getenv("DATABASE_URL", "sqlite:///./ml_api.db")

engine = create_engine(DATABASE_URL, connect_args={"check_same_thread": False})
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)
Base = declarative_base()

# Modeller
class TahminLog(Base):
    __tablename__ = "tahmin_loglar"

    id = Column(Integer, primary_key=True, index=True)
    yas = Column(Integer)
    gelir = Column(Float)
    kredi_skoru = Column(Integer)
    tahmin = Column(Integer)
    olasilik = Column(Float)
    model_versiyonu = Column(String, default="1.0.0")
    olusturma_tarih = Column(DateTime, default=datetime.utcnow)
    istek_ip = Column(String, nullable=True)

Base.metadata.create_all(bind=engine)
```

### Dependency Injection ile Session
```python
# FastAPI endpoint'lerinde kullan
from fastapi import Depends
from sqlalchemy.orm import Session

def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

# Endpoint'te kullan
@app.post("/tahmin")
def tahmin_yap(
    giris: TahminGiris,
    request: Request,
    db: Session = Depends(get_db)
):
    tahmin_val = int(ml_modeller["model"].predict([[giris.yas, giris.gelir]])[0])
    olasilik = float(ml_modeller["model"].predict_proba([[giris.yas, giris.gelir]])[0].max())

    # Log kaydet
    log = TahminLog(
        yas=giris.yas,
        gelir=giris.gelir,
        kredi_skoru=giris.kredi_skoru,
        tahmin=tahmin_val,
        olasilik=olasilik,
        istek_ip=request.client.host
    )
    db.add(log)
    db.commit()

    return {"tahmin": tahmin_val, "olasilik": olasilik}
```

### CRUD İşlemleri
```python
# Logları sorgula
@app.get("/loglar")
def log_listesi(
    limit: int = 100,
    min_olasilik: float = 0.0,
    db: Session = Depends(get_db)
):
    loglar = db.query(TahminLog)\
        .filter(TahminLog.olasilik >= min_olasilik)\
        .order_by(TahminLog.olusturma_tarih.desc())\
        .limit(limit)\
        .all()
    return loglar

# Özet istatistikler
@app.get("/istatistik")
def tahmin_istatistik(db: Session = Depends(get_db)):
    from sqlalchemy import func

    sonuc = db.query(
        func.count(TahminLog.id).label("toplam"),
        func.avg(TahminLog.olasilik).label("ort_olasilik"),
        func.sum(TahminLog.tahmin).label("onay_sayisi")
    ).one()

    return {
        "toplam_tahmin": sonuc.toplam,
        "ortalama_olasilik": round(float(sonuc.ort_olasilik or 0), 4),
        "onay_orani": round(sonuc.onay_sayisi / sonuc.toplam * 100, 2)
    }
```

### Alembic ile Migration
```bash
# Başlat
alembic init alembic

# Migration oluştur
alembic revision --autogenerate -m "tahmin_log tablosu"

# Uygula
alembic upgrade head

# Geri al
alembic downgrade -1
```

## 💡 Bağlantılar
- [[API - FastAPI Giriş]]
- [[API - ML Modeli Servis Etmek]]
- [[DS - SQL ve Veritabanı Kullanımı]]
- [[MSSQL - Python ile MSSQL Bağlantısı]]

## ❓ Sorular / Anlamadıklarım
- Async SQLAlchemy ne zaman gerekli?
- Connection pool nasıl ayarlanır?

## 🔗 Kaynaklar
- https://fastapi.tiangolo.com/tutorial/sql-databases/
