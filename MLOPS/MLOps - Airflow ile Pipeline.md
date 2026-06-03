---
tarih: 2026-05-28
konu: MLOps
etiket: ["mlops", "airflow", "pipeline", "orkestrasyon", "dag"]
kaynak: Apache Airflow Dokümantasyon
zorluk: orta
---

## 📌 Özet
Apache Airflow, karmaşık ML pipeline'larını DAG (Directed Acyclic Graph) olarak tanımlayan ve zamanlamanı yöneten açık kaynak orkestrasyon aracıdır.

## 🧠 Detay

### Kurulum
```bash
pip install apache-airflow apache-airflow-providers-docker

# Başlat
airflow db init
airflow webserver --port 8080
airflow scheduler
# → http://localhost:8080
```

### Temel DAG
```python
# dags/veri_pipeline.py
from airflow import DAG
from airflow.operators.python import PythonOperator
from airflow.operators.bash import BashOperator
from airflow.utils.dates import days_ago
from datetime import timedelta

varsayilan_args = {
    "owner": "veri-ekibi",
    "depends_on_past": False,
    "retries": 1,
    "retry_delay": timedelta(minutes=5),
}

with DAG(
    dag_id="ml_veri_pipeline",
    default_args=varsayilan_args,
    description="ML veri işleme pipeline'ı",
    schedule_interval="@daily",
    start_date=days_ago(1),
    catchup=False,
    tags=["ml", "veri"]
) as dag:

    def veri_cek(**context):
        import pandas as pd
        df = pd.read_sql("SELECT * FROM satislar WHERE tarih = CURDATE()", conn)
        df.to_parquet(f"/tmp/ham_veri_{context['ds']}.parquet")
        return len(df)

    def on_isle(**context):
        import pandas as pd
        df = pd.read_parquet(f"/tmp/ham_veri_{context['ds']}.parquet")
        # Temizle, dönüştür
        df.to_parquet(f"/tmp/temiz_veri_{context['ds']}.parquet")

    def ozellik_uret(**context):
        import pandas as pd
        df = pd.read_parquet(f"/tmp/temiz_veri_{context['ds']}.parquet")
        # Feature engineering
        df.to_parquet(f"/tmp/ozellikler_{context['ds']}.parquet")

    def model_egit(**context):
        from src.egitim import egit_ve_kaydet
        egit_ve_kaydet(f"/tmp/ozellikler_{context['ds']}.parquet")

    # Tasklar
    t1 = PythonOperator(task_id="veri_cek", python_callable=veri_cek)
    t2 = PythonOperator(task_id="on_isle", python_callable=on_isle)
    t3 = PythonOperator(task_id="ozellik_uret", python_callable=ozellik_uret)
    t4 = PythonOperator(task_id="model_egit", python_callable=model_egit)
    t5 = BashOperator(
        task_id="temizlik",
        bash_command="rm /tmp/*_{{ ds }}.parquet"
    )

    # Bağımlılıklar
    t1 >> t2 >> t3 >> t4 >> t5
```

### XCom ile Task İletişimi
```python
def veri_cek(**context):
    kayit_sayisi = 5000
    # XCom'a yaz
    context["ti"].xcom_push(key="kayit_sayisi", value=kayit_sayisi)
    return kayit_sayisi

def kontrol_et(**context):
    # XCom'dan oku
    kayit_sayisi = context["ti"].xcom_pull(
        task_ids="veri_cek",
        key="kayit_sayisi"
    )
    if kayit_sayisi < 100:
        raise ValueError(f"Yetersiz veri: {kayit_sayisi}")
```

### Dinamik DAG Oluşturma
```python
# Birden fazla model için otomatik DAG
modeller = ["random_forest", "xgboost", "lightgbm"]

with DAG("coklu_model_egitim", schedule_interval="@weekly") as dag:
    for model_adi in modeller:
        egit = PythonOperator(
            task_id=f"egit_{model_adi}",
            python_callable=model_egit,
            op_kwargs={"model_adi": model_adi}
        )
        degerlendir = PythonOperator(
            task_id=f"degerlendir_{model_adi}",
            python_callable=model_degerlendir,
            op_kwargs={"model_adi": model_adi}
        )
        egit >> degerlendir
```

### Sensörler
```python
from airflow.sensors.filesystem import FileSensor
from airflow.sensors.python import PythonSensor

# Dosya gelince başla
dosya_bekle = FileSensor(
    task_id="dosya_bekle",
    filepath="/data/input/gunluk_veri.csv",
    poke_interval=300,     # 5 dakikada bir kontrol
    timeout=7200,          # 2 saat max bekle
    mode="poke"
)

# Özel koşul
def veri_hazir_mi():
    import os
    return os.path.exists("/data/input/veri.csv")

veri_sensoru = PythonSensor(
    task_id="veri_sensoru",
    python_callable=veri_hazir_mi,
    poke_interval=60
)
```

### TaskGroup ile Organizasyon
```python
from airflow.utils.task_group import TaskGroup

with DAG("organize_pipeline") as dag:
    with TaskGroup("veri_hazirlama") as veri_grup:
        cek = PythonOperator(task_id="cek", python_callable=veri_cek)
        isle = PythonOperator(task_id="isle", python_callable=on_isle)
        cek >> isle

    with TaskGroup("model_sureci") as model_grup:
        egit = PythonOperator(task_id="egit", python_callable=model_egit)
        degerlendir = PythonOperator(task_id="degerlendir", python_callable=degerlendir)
        egit >> degerlendir

    veri_grup >> model_grup
```

### Airflow Variables ve Connections
```python
from airflow.models import Variable

# UI'da tanımlanan değişkeni oku
model_versiyonu = Variable.get("model_versiyonu", default_var="1.0")
esik = float(Variable.get("dogruluk_esigi", default_var="0.85"))
```

## 💡 Bağlantılar
- [[MLOps - Otomatik Yeniden Eğitim]]
- [[MLOps - MLflow ile Deney Takibi]]
- [[MLOps - DVC ile Veri Versiyonlama]]

## ❓ Sorular / Anlamadıklarım
- Airflow vs Prefect vs ZenML ne zaman hangisi?
- Executor türleri (Sequential, Local, Celery) nasıl seçilir?

## 🔗 Kaynaklar
- https://airflow.apache.org/docs/apache-airflow/stable/
