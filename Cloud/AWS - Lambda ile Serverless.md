---
tarih: 2026-05-28
konu: Cloud
etiket: ["aws", "lambda", "serverless", "tetikleyici", "event"]
kaynak: AWS Dokümantasyon
zorluk: orta
---

## 📌 Özet
AWS Lambda, sunucu yönetimi gerektirmeden olay tabanlı kod çalıştıran serverless servistir. Azure Functions'a karşılık gelir. S3, API Gateway, SQS gibi servislerle tetiklenebilir.

## 🧠 Detay

### Lambda Tetikleyicileri
```
API Gateway    → HTTP REST/WebSocket
S3             → Dosya yükleme/silme
SQS            → Mesaj kuyruğu
SNS            → Bildirim
EventBridge    → Zamanlanmış/event tabanlı
DynamoDB       → Stream değişiklikleri
Kinesis        → Gerçek zamanlı veri akışı
```

### Temel Lambda Fonksiyonu
```python
# lambda_function.py
import json
import boto3
import joblib
import numpy as np
import os

# Global scope'da yükle (cold start sonrası cache'lenir)
s3 = boto3.client("s3")
MODEL_BUCKET = os.environ["MODEL_BUCKET"]
MODEL_KEY = os.environ["MODEL_KEY"]

def model_yukle():
    from io import BytesIO
    response = s3.get_object(Bucket=MODEL_BUCKET, Key=MODEL_KEY)
    return joblib.load(BytesIO(response["Body"].read()))

model = model_yukle()

def lambda_handler(event, context):
    """Lambda ana giriş noktası"""
    try:
        # API Gateway'den gelen istek
        body = json.loads(event.get("body", "{}"))

        veri = np.array([[
            body["yas"],
            body["gelir"],
            body["kredi_skoru"]
        ]])

        tahmin = int(model.predict(veri)[0])
        olasilik = float(model.predict_proba(veri)[0].max())

        return {
            "statusCode": 200,
            "headers": {
                "Content-Type": "application/json",
                "Access-Control-Allow-Origin": "*"
            },
            "body": json.dumps({
                "tahmin": tahmin,
                "olasilik": olasilik,
                "karar": "Onaylandı" if tahmin == 1 else "Reddedildi"
            })
        }

    except KeyError as e:
        return {"statusCode": 400, "body": json.dumps({"hata": f"Eksik alan: {e}"})}
    except Exception as e:
        return {"statusCode": 500, "body": json.dumps({"hata": str(e)})}
```

### S3 Trigger Lambda
```python
def lambda_handler(event, context):
    """S3'e yeni dosya gelince tetiklenir"""
    for record in event["Records"]:
        bucket = record["s3"]["bucket"]["name"]
        key = record["s3"]["object"]["key"]

        print(f"Yeni dosya: s3://{bucket}/{key}")

        # Dosyayı işle
        import pandas as pd
        from io import BytesIO

        response = s3.get_object(Bucket=bucket, Key=key)
        df = pd.read_parquet(BytesIO(response["Body"].read()))

        # Temizle
        temiz_df = df.dropna().drop_duplicates()

        # Silver'a yaz
        silver_key = key.replace("bronze/", "silver/")
        buffer = BytesIO()
        temiz_df.to_parquet(buffer, index=False)
        s3.put_object(
            Bucket=bucket,
            Key=silver_key,
            Body=buffer.getvalue()
        )

        print(f"Silver'a yazıldı: {silver_key}")
```

### EventBridge (Zamanlanmış)
```python
def lambda_handler(event, context):
    """Her gece çalışır"""
    import logging

    logging.info(f"Zamanlanmış iş başladı: {event}")

    # Model drift kontrolü
    from monitoring import drift_kontrol
    drift_var = drift_kontrol()

    if drift_var:
        # Retraining tetikle
        import boto3
        sagemaker = boto3.client("sagemaker")
        sagemaker.start_pipeline_execution(
            PipelineName="KrediOnayPipeline"
        )
        logging.info("Yeniden eğitim tetiklendi")
```

### Layer ile Büyük Kütüphaneler
```bash
# Layer oluştur (scikit-learn, pandas gibi büyük kütüphaneler)
mkdir python
pip install scikit-learn numpy pandas -t python/
zip -r sklearn-layer.zip python/

# AWS CLI ile yükle
aws lambda publish-layer-version \
  --layer-name sklearn-layer \
  --zip-file fileb://sklearn-layer.zip \
  --compatible-runtimes python3.11
```

### SAM ile Local Test
```bash
pip install aws-sam-cli

# template.yaml
# Resources:
#   TahminFunction:
#     Type: AWS::Serverless::Function
#     Properties:
#       Handler: lambda_function.lambda_handler
#       Runtime: python3.11
#       Timeout: 30
#       MemorySize: 512
#       Environment:
#         Variables:
#           MODEL_BUCKET: ml-veri-golum
#           MODEL_KEY: models/rf_model.pkl

# Lokal test
sam local invoke TahminFunction --event test_event.json
sam local start-api  # HTTP test
```

### Environment Variables
```python
import os

# Lambda'da environment variable kullanımı
MODEL_BUCKET = os.environ["MODEL_BUCKET"]
API_KEY = os.environ["API_KEY"]
LOG_LEVEL = os.environ.get("LOG_LEVEL", "INFO")
```

### Cold Start Optimizasyonu
```python
# ❌ Kötü - her çağrıda yükle
def lambda_handler(event, context):
    model = joblib.load(...)  # Yavaş!

# ✅ İyi - global scope'da yükle
model = joblib.load(...)  # Bir kez yüklenir

def lambda_handler(event, context):
    tahmin = model.predict(...)  # Hızlı!
```

## 💡 Bağlantılar
- [[Cloud - Bulut Bilişime Giriş]]
- [[AWS - S3 ile Veri Depolama]]
- [[AWS - SageMaker ile ML]]
- [[Azure - Azure Functions ve Event Grid]]

## ❓ Sorular / Anlamadıklarım
- Lambda timeout limiti (15 dk) aşılırsa ne yapmalıyım?
- Provisioned concurrency ne zaman gerekli?

## 🔗 Kaynaklar
- https://docs.aws.amazon.com/lambda/
