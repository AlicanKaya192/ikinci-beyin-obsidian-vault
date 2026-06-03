---
tarih: 2026-05-28
konu: MLOps
etiket: ["mlops", "retraining", "yeniden-eğitim", "otomatik", "airflow"]
kaynak: 
zorluk: orta
---

## 📌 Özet
Modeller zamanla bozulur; otomatik yeniden eğitim sistemi drift veya performans düşüşü tespit edildiğinde modeli yeniden eğitip deploy eder. Airflow veya Prefect ile orkestre edilir.

## 🧠 Detay

### Yeniden Eğitim Tetikleyicileri
```
Zamana Dayalı:
  → Her hafta/ay otomatik eğit
  → Düzenli ama verimsiz olabilir

Performans Bazlı:
  → Accuracy < 0.85 → eğit
  → F1 < 0.80 → eğit

Drift Bazlı:
  → Veri drifti tespit edildi → eğit
  → KS testi p < 0.05 → eğit

Veri Hacmi Bazlı:
  → 10.000 yeni kayıt geldi → eğit
```

### Basit Retraining Script
```python
# src/retraining.py
import mlflow
import joblib
import pandas as pd
import numpy as np
from sklearn.ensemble import RandomForestClassifier
from sklearn.model_selection import train_test_split
from sklearn.metrics import accuracy_score, f1_score
from datetime import datetime
import logging

logger = logging.getLogger(__name__)

METRIK_ESIKLERI = {
    "accuracy": 0.85,
    "f1_score": 0.80
}

def veri_yukle() -> pd.DataFrame:
    """En güncel veriyi yükle"""
    # DB'den veya S3'ten
    return pd.read_parquet("data/guncel_veri.parquet")

def model_egit(df: pd.DataFrame) -> tuple:
    X = df.drop(columns=["hedef"])
    y = df["hedef"]

    X_train, X_test, y_train, y_test = train_test_split(
        X, y, test_size=0.2, random_state=42, stratify=y
    )

    model = RandomForestClassifier(n_estimators=200, max_depth=10, n_jobs=-1)
    model.fit(X_train, y_train)

    metrikler = {
        "accuracy": accuracy_score(y_test, model.predict(X_test)),
        "f1_score": f1_score(y_test, model.predict(X_test)),
        "egitim_boyutu": len(X_train),
        "egitim_tarihi": datetime.now().isoformat()
    }

    return model, metrikler

def kalite_kontrol(metrikler: dict) -> bool:
    for metrik, esik in METRIK_ESIKLERI.items():
        if metrikler[metrik] < esik:
            logger.warning(f"❌ {metrik}: {metrikler[metrik]:.3f} < {esik}")
            return False
    return True

def modeli_kaydet(model, metrikler: dict) -> str:
    with mlflow.start_run():
        mlflow.log_metrics(metrikler)
        result = mlflow.sklearn.log_model(
            model, "model",
            registered_model_name="KrediOnayModeli"
        )
    return result.model_uri

def production_guncelle(model_uri: str):
    """Yeni modeli production'a al"""
    from mlflow.tracking import MlflowClient
    client = MlflowClient()

    # En son versiyonu production'a taşı
    model_versiyonlari = client.get_latest_versions("KrediOnayModeli")
    en_son = max(model_versiyonlari, key=lambda x: int(x.version))

    client.transition_model_version_stage(
        name="KrediOnayModeli",
        version=en_son.version,
        stage="Production",
        archive_existing_versions=True
    )
    logger.info(f"✅ v{en_son.version} Production'a alındı")

def main():
    logger.info("Yeniden eğitim başladı")

    df = veri_yukle()
    logger.info(f"Veri yüklendi: {len(df)} kayıt")

    model, metrikler = model_egit(df)
    logger.info(f"Metrikler: {metrikler}")

    if not kalite_kontrol(metrikler):
        logger.error("Kalite kontrolü başarısız, deploy edilmedi")
        return False

    model_uri = modeli_kaydet(model, metrikler)
    production_guncelle(model_uri)

    logger.info("✅ Yeniden eğitim tamamlandı")
    return True

if __name__ == "__main__":
    main()
```

### Airflow ile Zamanlanmış Eğitim
```python
# dags/retraining_dag.py
from airflow import DAG
from airflow.operators.python import PythonOperator, BranchPythonOperator
from airflow.operators.email import EmailOperator
from datetime import datetime, timedelta

varsayilan_args = {
    "owner": "veri-ekibi",
    "retries": 2,
    "retry_delay": timedelta(minutes=5),
    "email_on_failure": True,
    "email": ["team@sirket.com"]
}

with DAG(
    dag_id="ml_retraining_pipeline",
    default_args=varsayilan_args,
    schedule_interval="0 2 * * 1",   # Her Pazartesi 02:00
    start_date=datetime(2026, 1, 1),
    catchup=False,
    tags=["ml", "retraining"]
) as dag:

    def drift_kontrol(**context):
        from src.monitoring import drift_tespit
        drift_var = drift_tespit()
        return "egit" if drift_var else "atla"

    def model_egit_task(**context):
        from src.retraining import main
        basarili = main()
        if not basarili:
            raise ValueError("Model eğitimi başarısız")

    drift_karar = BranchPythonOperator(
        task_id="drift_kontrol",
        python_callable=drift_kontrol
    )

    egit = PythonOperator(
        task_id="egit",
        python_callable=model_egit_task
    )

    bildirim = EmailOperator(
        task_id="bildirim",
        to="team@sirket.com",
        subject="Model Yeniden Eğitildi",
        html_content="<p>Yeni model production'a alındı.</p>"
    )

    drift_karar >> [egit, "atla"]
    egit >> bildirim
```

### Prefect ile Pipeline
```python
from prefect import flow, task
from prefect.schedules import CronSchedule

@task(retries=3, retry_delay_seconds=60)
def veri_yukle_task():
    return pd.read_parquet("data/guncel_veri.parquet")

@task
def egit_task(df):
    return model_egit(df)

@task
def deploy_task(model, metrikler):
    if kalite_kontrol(metrikler):
        modeli_kaydet(model, metrikler)
        production_guncelle(None)
        return True
    return False

@flow(name="ML Retraining Pipeline")
def retraining_pipeline():
    df = veri_yukle_task()
    model, metrikler = egit_task(df)
    deploy_task(model, metrikler)

# Zamanla
retraining_pipeline.serve(
    name="haftalik-retraining",
    cron="0 2 * * 1"
)
```

## 💡 Bağlantılar
- [[MLOps - Model Monitoring ve Drift Tespiti]]
- [[MLOps - MLflow ile Deney Takibi]]
- [[MLOps - GitHub Actions ile CI-CD]]
- [[MLOps - Airflow ile Pipeline]]

## ❓ Sorular / Anlamadıklarım
- Online learning ile retraining arasındaki fark?
- Eğitim sırasında eski model hizmet vermeye devam edebilir mi?

## 🔗 Kaynaklar
- https://airflow.apache.org/docs/
- https://docs.prefect.io/
