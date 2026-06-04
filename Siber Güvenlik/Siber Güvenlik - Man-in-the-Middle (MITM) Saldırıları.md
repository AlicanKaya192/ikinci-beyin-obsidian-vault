---
tarih: 2026-06-04
konu: Siber Güvenlik - Man-in-the-Middle (MITM) Saldırıları
etiket: [cybersecurity, mitm, networking, spoofing]
kaynak: Wireshark Network Analysis
zorluk: Orta
---

## 📌 Özet
Man-in-the-Middle (MITM) saldırısı, bir saldırganın iki taraf arasındaki iletişimi gizlice kesmesi, izlemesi veya değiştirmesi durumudur. Taraflar birbirleriyle doğrudan ve güvenli bir şekilde konuştuklarını sanırken, tüm veri trafiği saldırganın kontrolündeki bir cihaz üzerinden geçer. Bu saldırılar genellikle açık Wi-Fi ağlarında ve yerel ağlardaki (LAN) ARP veya DNS protokol zayıflıkları kullanılarak gerçekleştirilir. MITM saldırıları sonucunda oturum bilgileri, şifreler ve hassas kişisel veriler çalınabilir. Uçtan uca şifreleme ve güçlü sertifika denetimi, bu sinsi tehdide karşı en kritik savunma kalkanlarıdır.

## 🧠 Detay

```mermaid
graph LR
    A["Kullanıcı"] -- "Veri" --> B["Saldırgan (Hacker)"]
    B -- "Manipüle Veri" --> C["Sunucu"]
    C -- "Yanıt" --> B
    B -- "Yanıt" --> A
```

### 1. Yaygın Teknikler
- **ARP Spoofing:** Yerel ağda MAC-IP eşleşmesini bozarak trafiği üzerine çekme.
- **DNS Spoofing:** Kullanıcıyı sahte web sitelerine yönlendirme.
- **SSL Stripping:** HTTPS bağlantısını HTTP'ye düşürme.

### 2. Savunma
- **HTTPS ve HSTS:** Bağlantıların her zaman şifreli olması sağlanmalıdır.
- **VPN:** Tüm trafiği şifreli bir tünel üzerinden geçirir.

## 💡 Bağlantılar
- [[Siber Güvenlik - Ağ Güvenliği Temelleri (TCP-IP, Firewall, VPN)]]
- [[Siber Güvenlik - Kriptografi Temelleri (Simetrik, Asimetrik, Hash)]]

## ❓ Sorular / Anlamadıklarım

## 🔗 Kaynaklar
- OWASP MITM Attack Guide
- Wireshark 101 Training
