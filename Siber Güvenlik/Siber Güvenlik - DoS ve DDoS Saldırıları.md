---
tarih: 2026-06-04
konu: Siber Güvenlik - DoS ve DDoS Saldırıları
etiket: [cybersecurity, ddos, networking, botnet]
kaynak: Cloudflare, Akamai
zorluk: Orta
---

## 📌 Özet
DoS (Denial of Service) ve DDoS (Distributed Denial of Service) saldırıları, bir sistemin, ağın veya servisin kaynaklarını tüketerek meşru kullanıcıların erişimini engellemeyi amaçlar. DoS tek bir kaynaktan yapılırken, DDoS dünya çapında dağılmış binlerce zombi cihazdan (Botnet) oluşan bir ağ üzerinden gerçekleştirilir. Bu saldırılar hacimsel, protokol bazlı veya uygulama katmanı düzeyinde olabilir. Hedef sistemin bant genişliğini, CPU gücünü veya bellek kapasitesini aşırı yükleyerek servis dışı kalmasına neden olur. Bu doküman, DDoS türlerini, Botnet kavramını ve modern savunma teknolojilerini ele alır.

## 🧠 Detay

```mermaid
graph TD
    A["DDoS Saldırı Türleri"] --> B["Hacimsel (Volumetric)"]
    A --> C["Protokol Bazlı"]
    A --> D["Uygulama Katmanı (L7)"]
    
    B --> B1["UDP Flood"]
    B --> B2["DNS Amplification"]
    
    C --> C1["SYN Flood"]
    C --> C2["Ping of Death"]
    
    D --> D1["HTTP Flood"]
    D --> D2["Slowloris"]
```

### 1. Saldırı Mekanizmaları
- **Amplification:** Küçük bir sorgu gönderip kurbanın IP adresiyle çok büyük bir yanıtın dönmesini sağlama.
- **Botnet:** Zararlı yazılım bulaşmış binlerce IoT veya PC'den oluşan saldırı ordusu.

### 2. Savunma Stratejileri
- **Anycast Network:** Trafiği coğrafi olarak dağıtarak yükü hafifletme.
- **Scrubbing Centers:** Trafiği temizleyip sadece meşru olanları hedefe iletme.
- **Rate Limiting:** IP bazlı istek sınırlandırma.

## 💡 Bağlantılar
- [[Siber Güvenlik - Giriş ve Temel Kavramlar (CIA, Threat Model)]]
- [[Siber Güvenlik - Ağ Güvenliği Temelleri (TCP-IP, Firewall, VPN)]]

## ❓ Sorular / Anlamadıklarım

## 🔗 Kaynaklar
- Cloudflare DDoS Learning Center
- Radware Global Application & Network Security Report
