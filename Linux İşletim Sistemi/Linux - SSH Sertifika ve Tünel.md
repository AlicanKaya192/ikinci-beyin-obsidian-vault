---
tarih: 2026-06-04
konu: Linux SSH Sertifikaları ve Tünelleme
etiket: [linux, ssh, güvenlik, tünel, anahtar]
kaynak: 
zorluk: İleri
---

## 📌 Özet
SSH (Secure Shell), Linux sunucularına uzaktan erişmek için kullanılan standart protokoldür. Parola ile giriş yapmak yerine, SSH Anahtarları (Public/Private Key pair) kullanmak çok daha güvenli ve pratik bir yöntemdir. Bu yöntem, asimetrik şifreleme mantığına dayanır; gizli anahtar (Private Key) istemcide (kullanıcının bilgisayarında) kalırken, açık anahtar (Public Key) bağlanılacak sunucuya (`~/.ssh/authorized_keys` dosyasına) yerleştirilir. Anahtar tabanlı kimlik doğrulama, Brute Force (kaba kuvvet) saldırılarını imkansız hale getirir. SSH'ın bir diğer güçlü özelliği ise "Tünelleme" (Port Forwarding) yapabilmesidir. SSH tüneli sayesinde, dışarıya kapalı olan (örneğin bir veritabanı veya iç ağdaki bir web uygulaması) servislere, şifrelenmiş SSH bağlantısı üzerinden güvenli bir şekilde erişilebilir.

## 🧠 Detay

```mermaid
graph TD
    A["İstemci (Sizin Bilgisayarınız)"] -->|1. Anahtar Üretimi| B["Private Key (id_rsa)"]
    A -->|1. Anahtar Üretimi| C["Public Key (id_rsa.pub)"]
    C -->|2. Kopyalama (ssh-copy-id)| D["Uzak Sunucu"]
    D --> E["~/.ssh/authorized_keys"]
    A -->|3. Parolasız Giriş| D
```

### SSH Anahtarı Oluşturma ve Kopyalama
1. **Anahtar Üretimi:** Kendi bilgisayarınızın terminalinde şu komutu çalıştırın:
   `ssh-keygen -t ed25519 -C "mail@adresiniz.com"`
   *(Eskiden RSA kullanılırdı, modern sistemlerde daha güvenli ve performanslı olan ED25519 algoritması tercih edilir).*
   Bu işlem `~/.ssh/` dizininde bir gizli (`id_ed25519`), bir de açık (`id_ed25519.pub`) anahtar oluşturur.
2. **Sunucuya Kopyalama:** Açık anahtarınızı bağlanmak istediğiniz sunucuya gönderin:
   `ssh-copy-id kullanici@sunucu_ip_adresi`
3. **Bağlantı Testi:** Artık parola sorulmadan giriş yapılabilmelidir:
   `ssh kullanici@sunucu_ip_adresi`

### Sunucu Tarafı Güvenlik Ayarları (/etc/ssh/sshd_config)
Sunucu güvenliğini maksimize etmek için SSH servisi yapılandırılmalıdır (değişiklik sonrası `sudo systemctl restart sshd` gerektirir):
- `PermitRootLogin no`: Root kullanıcısının direkt SSH girişi yapmasını engeller. (Normal kullanıcıyla girilip `sudo` olunmalıdır).
- `PasswordAuthentication no`: Sadece SSH anahtarı ile girişe izin verir, parola ile girişi tamamen kapatır (Bunu yapmadan önce anahtarınızın çalıştığından emin olun!).
- `Port 2222`: Varsayılan 22. portu değiştirmek, otomatik tarama botlarını (script kiddies) engellemeye yardımcı olur.

### SSH Tünelleme (Port Forwarding)
Sunucunun iç ağında çalışan ama dışarıdan erişilemeyen bir MySQL veritabanına (3306 portu) kendi bilgisayarımızdan bağlanmak için yerel tünel (Local Port Forwarding) açabiliriz:
```bash
ssh -L 8080:localhost:3306 kullanici@sunucu_ip_adresi
```
- Bu komut, sunucuya SSH bağlantısı kurar.
- Aynı zamanda sizin bilgisayarınızdaki `8080` portunu, sunucunun içindeki `localhost:3306` portuna bağlar.
- Artık kendi bilgisayarınızda bir veritabanı aracı açıp `localhost:8080`'e bağlandığınızda, aslında sunucudaki MySQL'e güvenli SSH tüneli içinden bağlanmış olursunuz.

## 💡 Bağlantılar
- [[Linux - Ağ Komutları (ip, netstat, curl, ssh)]]
- [[Linux - Firewall (iptables, ufw)]]

## ❓ Sorular / Anlamadıklarım

## 🔗 Kaynaklar
- [SSH Port Forwarding Example](https://www.ssh.com/academy/ssh/tunneling/example)