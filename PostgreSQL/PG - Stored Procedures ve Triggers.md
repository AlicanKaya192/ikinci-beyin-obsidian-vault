---
tarih: 2026-08-17
konu: PostgreSQL
etiket: [postgresql, stored-procedure, trigger, plpgsql, ileri]
kaynak: PostgreSQL Docs
zorluk: ileri
---

## 📌 Özet

PL/pgSQL, PostgreSQL'in prosedürel dili. Stored procedure ve fonksiyonlar, karmaşık iş mantığını veritabanında çalıştırır. Trigger'lar ise veri değişikliklerini otomatik olarak yakalar. ML sistemlerinde audit log, otomatik hesaplama ve veri bütünlüğü için çok kullanılır.

---

## 🧠 Detay

### Fonksiyon vs Procedure

| | Fonksiyon | Procedure |
|---|-----------|-----------|
| **Return** | Değer döner | Döndürmez (void) |
| **Transaction** | Kendi transaction'ını yönetemez | COMMIT/ROLLBACK yapabilir |
| **Çağrı** | SELECT ile | CALL ile |
| **Ne zaman?** | Hesaplama, dönüşüm | İş süreci, batch |

### Temel Fonksiyon

```sql
-- Basit fonksiyon
CREATE OR REPLACE FUNCTION yas_hesapla(dogum_tarihi DATE)
RETURNS INT
LANGUAGE plpgsql
AS $$
BEGIN
    RETURN EXTRACT(YEAR FROM AGE(dogum_tarihi))::INT;
END;
$$;

-- Kullanım
SELECT ad, yas_hesapla(dogum_tarihi) AS yas FROM kullanicilar;

-- TABLE döndüren fonksiyon
CREATE OR REPLACE FUNCTION aktif_kullanicilar(gun_sayisi INT DEFAULT 30)
RETURNS TABLE(id INT, email TEXT, son_aktiflik DATE)
LANGUAGE plpgsql
AS $$
BEGIN
    RETURN QUERY
        SELECT k.id, k.email, MAX(i.tarih)::DATE
        FROM kullanicilar k
        JOIN islemler i ON k.id = i.kullanici_id
        WHERE i.tarih >= CURRENT_DATE - gun_sayisi
        GROUP BY k.id, k.email;
END;
$$;

SELECT * FROM aktif_kullanicilar(7);
```

### PL/pgSQL Kontrol Akışı

```sql
CREATE OR REPLACE FUNCTION musteri_segment(kullanici_id INT)
RETURNS TEXT
LANGUAGE plpgsql
AS $$
DECLARE
    toplam_harcama NUMERIC;
    aktif_gun INT;
    segment TEXT;
BEGIN
    -- Değişkene ata
    SELECT
        SUM(tutar),
        COUNT(DISTINCT DATE(tarih))
    INTO toplam_harcama, aktif_gun
    FROM siparisler
    WHERE kullanici_id = musteri_segment.kullanici_id
      AND tarih >= CURRENT_DATE - 90;

    -- NULL kontrolü
    IF toplam_harcama IS NULL THEN
        RETURN 'Pasif';
    END IF;

    -- Koşullu segmentasyon
    IF toplam_harcama >= 10000 AND aktif_gun >= 15 THEN
        segment := 'VIP';
    ELSIF toplam_harcama >= 3000 THEN
        segment := 'Premium';
    ELSIF aktif_gun >= 5 THEN
        segment := 'Aktif';
    ELSE
        segment := 'Standart';
    END IF;

    RETURN segment;
END;
$$;
```

### Loop ve Cursor

```sql
CREATE OR REPLACE PROCEDURE toplu_segment_guncelle()
LANGUAGE plpgsql
AS $$
DECLARE
    kayit RECORD;
    guncellenen INT := 0;
BEGIN
    -- FOR loop ile cursor
    FOR kayit IN
        SELECT id FROM kullanicilar WHERE aktif = true
    LOOP
        UPDATE kullanicilar
        SET segment = musteri_segment(kayit.id)
        WHERE id = kayit.id;

        guncellenen := guncellenen + 1;

        -- Her 1000 kayıtta commit
        IF guncellenen % 1000 = 0 THEN
            COMMIT;
            RAISE NOTICE '% kayıt güncellendi', guncellenen;
        END IF;
    END LOOP;

    COMMIT;
    RAISE NOTICE 'Toplam % kayıt güncellendi', guncellenen;
END;
$$;

-- Çağır
CALL toplu_segment_guncelle();
```

### Exception Handling

```sql
CREATE OR REPLACE FUNCTION guvenli_insert(email TEXT, ad TEXT)
RETURNS BOOLEAN
LANGUAGE plpgsql
AS $$
BEGIN
    INSERT INTO kullanicilar (email, ad) VALUES (email, ad);
    RETURN TRUE;
EXCEPTION
    WHEN unique_violation THEN
        RAISE WARNING 'Email zaten var: %', email;
        RETURN FALSE;
    WHEN check_violation THEN
        RAISE WARNING 'Geçersiz veri: %', SQLERRM;
        RETURN FALSE;
    WHEN OTHERS THEN
        RAISE EXCEPTION 'Beklenmeyen hata: %', SQLERRM;
END;
$$;
```

---

### Trigger'lar

```sql
-- Trigger türleri:
-- BEFORE INSERT/UPDATE/DELETE → işlemden önce
-- AFTER INSERT/UPDATE/DELETE  → işlemden sonra
-- INSTEAD OF                  → view üzerinde
-- FOR EACH ROW                → her satır için
-- FOR EACH STATEMENT          → her SQL ifadesi için
```

### Audit Log Trigger

```sql
-- Audit tablosu
CREATE TABLE audit_log (
    id BIGSERIAL PRIMARY KEY,
    tablo_adi TEXT,
    islem TEXT,        -- INSERT, UPDATE, DELETE
    kayit_id BIGINT,
    eski_veri JSONB,
    yeni_veri JSONB,
    kullanici TEXT DEFAULT current_user,
    zaman TIMESTAMPTZ DEFAULT NOW()
);

-- Trigger fonksiyonu
CREATE OR REPLACE FUNCTION audit_trigger_fn()
RETURNS TRIGGER
LANGUAGE plpgsql
AS $$
BEGIN
    IF TG_OP = 'INSERT' THEN
        INSERT INTO audit_log (tablo_adi, islem, kayit_id, yeni_veri)
        VALUES (TG_TABLE_NAME, 'INSERT', NEW.id, to_jsonb(NEW));
        RETURN NEW;

    ELSIF TG_OP = 'UPDATE' THEN
        INSERT INTO audit_log (tablo_adi, islem, kayit_id, eski_veri, yeni_veri)
        VALUES (TG_TABLE_NAME, 'UPDATE', NEW.id, to_jsonb(OLD), to_jsonb(NEW));
        RETURN NEW;

    ELSIF TG_OP = 'DELETE' THEN
        INSERT INTO audit_log (tablo_adi, islem, kayit_id, eski_veri)
        VALUES (TG_TABLE_NAME, 'DELETE', OLD.id, to_jsonb(OLD));
        RETURN OLD;
    END IF;
END;
$$;

-- Trigger'ı tabloya bağla
CREATE TRIGGER trg_siparis_audit
    AFTER INSERT OR UPDATE OR DELETE ON siparisler
    FOR EACH ROW EXECUTE FUNCTION audit_trigger_fn();
```

### Otomatik Hesaplama Trigger

```sql
-- Sipariş eklenince stok güncelle
CREATE OR REPLACE FUNCTION stok_guncelle()
RETURNS TRIGGER
LANGUAGE plpgsql
AS $$
BEGIN
    IF TG_OP = 'INSERT' THEN
        UPDATE urunler
        SET stok = stok - NEW.miktar
        WHERE id = NEW.urun_id;

        -- Stok negatif olmasın
        IF (SELECT stok FROM urunler WHERE id = NEW.urun_id) < 0 THEN
            RAISE EXCEPTION 'Yetersiz stok: ürün %', NEW.urun_id;
        END IF;

    ELSIF TG_OP = 'DELETE' THEN
        UPDATE urunler
        SET stok = stok + OLD.miktar
        WHERE id = OLD.urun_id;
    END IF;

    RETURN NEW;
END;
$$;

CREATE TRIGGER trg_siparis_stok
    AFTER INSERT OR DELETE ON siparis_satirlari
    FOR EACH ROW EXECUTE FUNCTION stok_guncelle();
```

### Trigger Yönetimi

```sql
-- Trigger'ı geçici devre dışı bırak (bulk import için)
ALTER TABLE siparisler DISABLE TRIGGER trg_siparis_audit;
-- ... toplu import ...
ALTER TABLE siparisler ENABLE TRIGGER trg_siparis_audit;

-- Tüm trigger'ları devre dışı bırak
ALTER TABLE siparisler DISABLE TRIGGER ALL;

-- Trigger listesi
SELECT trigger_name, event_manipulation, action_timing
FROM information_schema.triggers
WHERE event_object_table = 'siparisler';
```

---

## 💡 Bağlantılar
- [[PG - PostgreSQL Güvenliği ve Row-Level Security]]
- [[PG - İleri SQL — Window Functions ve CTE]]
- [[MSSQL - Stored Procedure ve Fonksiyon Oluşturma]]

## ❓ Sorular / Anlamadıklarım

## 🔗 Kaynaklar
- [PostgreSQL PL/pgSQL](https://www.postgresql.org/docs/current/plpgsql.html)
- [PostgreSQL Triggers](https://www.postgresql.org/docs/current/triggers.html)
