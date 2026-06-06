---
tarih: 2026-06-06
konu: OpenTelemetry
etiket: [monitoring, observability, otel, tracing, metrics]
kaynak: Gemini CLI
zorluk: İleri
---

## 📌 Ozet
OpenTelemetry (OTel), bulut tabanlı yazılımlardan log, metrik ve trace (iz sürme) toplamak için kullanılan açık kaynaklı bir standarttır. Vendor-neutral (tedarikçiden bağımsız) yapısı sayesinde veriyi bir kez toplayıp istediğiniz backend e (Prometheus, Jaeger, Datadog vb.) göndermenizi sağlar.

## 🧠 Detay

```mermaid
graph LR
    App["App (SDK)"] --> Col["OTel Collector"]
    Col --> Proc["Process / Batch"]
    Proc --> Prom["Prometheus (Metrics)"]
    Proc --> Jae["Jaeger (Traces)"]
    Proc --> ELK["ELK (Logs)"]
```

### 1. OTel Bileşenleri
- **SDK:** Uygulama içinden veri toplamak için kullanılan kütüphaneler.
- **Collector:** Veriyi alan, işleyen ve farklı yerlere aktaran merkezi servis.
- **Exporter:** Veriyi hedef sisteme uygun formata çevirip gönderen modül.

### 2. Neden Standart?
- Uygulama kodunu değiştirmeden izleme aracını (backend) değiştirebilirsiniz.
- Dağıtık sistemlerde (Microservices) uçtan uca görünürlük sağlar.

## 💡 Baglantilar
- [[MO - Distributed Tracing ve Jaeger]]
- [[MO - Prometheus ile Metrik Toplama]]
