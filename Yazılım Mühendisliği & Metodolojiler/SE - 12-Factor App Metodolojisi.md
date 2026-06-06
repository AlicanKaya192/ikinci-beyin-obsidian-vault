---
tarih: 2026-06-06
konu: 12-Factor App
etiket: [software-engineering, saas, cloud-native, best-practices]
kaynak: Gemini CLI
zorluk: İleri
---

## 📌 Ozet
12-Factor App, modern, ölçeklenebilir ve bulut tabanlı (cloud-native) yazılımlar geliştirmek için kullanılan 12 altın kuraldır. Heroku nun kurucuları tarafından oluşturulan bu metodoloji, uygulamanın taşınabilirliğini ve sürekli dağıtımını (CI/CD) kolaylaştırır.

## 🧠 Detay

```mermaid
graph TD
    Code["1. Codebase"] --> Config["3. Config"]
    Config --> Dependencies["2. Dependencies"]
    Dependencies --> Build["5. Build, Release, Run"]
    Build --> Stateless["6. Processes (Stateless)"]
    Stateless --> Port["7. Port Binding"]
    Port --> Scale["8. Concurrency"]
    Scale --> Logs["11. Logs"]
```

### Kritik Faktörlerden Bazıları
- **III. Config:** Ayarlar kodun içinde değil, ortam değişkenlerinde (Env Vars) tutulmalıdır.
- **VI. Processes:** Uygulama "stateless" (durumsuz) olmalı; veriler bir veritabanı veya cache de saklanmalıdır.
- **XI. Logs:** Uygulama log yazma yönetimiyle uğraşmamalı, logları "stdout" a bir akış olarak vermelidir.

## 💡 Baglantilar
- [[SE - Clean Code Prensipleri ve Best Practices]]
- [[API - Production Deployment (Gunicorn, Uvicorn, Nginx)]]
