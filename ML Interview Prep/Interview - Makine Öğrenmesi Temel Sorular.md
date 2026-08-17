---
tarih: 2026-06-08
konu: ML Interview Prep
etiket: [mülakat, interview, machine-learning, temel, sorular-cevaplar]
kaynak: Chip Huyen ML Interviews, Glassdoor
zorluk: başlangıç-orta
---

## 📌 Özet

ML mülakatlarında en sık karşılaşılan klasik makine öğrenmesi soruları ve model cevapları.

---

## 🧠 Detay

### Temel Kavramlar

**S: Bias-Variance tradeoff nedir? Örnek ver.**
> **Bias** — modelin gerçek ilişkiyi yakalamadaki yetersizliği (underfitting).
> **Variance** — modelin eğitim verisine aşırı bağlı olması (overfitting).
> Tradeoff: karmaşıklık artarsa bias ↓ ama variance ↑.
>
> Örnek: Lineer model → yüksek bias (karmaşık ilişkiyi yakalayamaz). 1000 dallı decision tree → yüksek variance (eğitim verisini ezberler).
> Çözüm: Regularization (L1/L2), cross-validation, ensemble yöntemler.

---

**S: L1 ve L2 regularization farkı nedir?**
> **L1 (Lasso):** `λΣ|wᵢ|` → ağırlıkları sıfıra iter → **özellik seçimi** yapar (sparse)
> **L2 (Ridge):** `λΣwᵢ²` → ağırlıkları küçük tutar ama sıfır yapmaz → **çok doğrusal bağlantıda** iyi
> **Elastic Net:** L1 + L2 kombinasyonu
>
> Ne zaman L1? Gereksiz özellikler çok, model sadeliği istiyorsun.
> Ne zaman L2? Çoğu özellik işe yarıyor, küçük bir iyileştirme istiyorsun.

---

**S: Precision ve Recall arasında nasıl seçim yaparsın?**
> **Precision** = TP / (TP + FP) → "Pozitif dediğimin kaçı gerçekten pozitif?"
> **Recall** = TP / (TP + FN) → "Gerçek pozitiflerin kaçını yakaladım?"
>
> - **Precision öncelikli:** Spam filtreleme (yanlış spam etiketi rahatsız eder)
> - **Recall öncelikli:** Kanser tespiti (kaçırılan hasta tehlikelidir)
> - **F1:** İkisini dengeler → genel amaçlı

```python
from sklearn.metrics import classification_report
print(classification_report(y_true, y_pred))
```

---

**S: Imbalanced dataset ile nasıl başa çıkarsın?**
> 1. **Oversampling:** SMOTE ile az sınıf sentetik örnekler üret
> 2. **Undersampling:** çoğunluk sınıftan örnekleri azalt
> 3. **class_weight="balanced":** sklearn'de otomatik ağırlıklandırma
> 4. **Farklı metrikler:** Accuracy yerine F1, AUC-ROC, PR-AUC kullan
> 5. **Threshold ayarı:** Default 0.5 yerine precision-recall eğrisinden optimum eşik bul

---

**S: Random Forest ile XGBoost arasındaki temel fark nedir?**
> **Random Forest:** Ağaçlar paralel, birbirinden bağımsız (bagging). Kolay tune edilir, overfitting'e karşı dayanıklı.
> **XGBoost:** Ağaçlar sıralı, önceki hataları düzeltir (boosting + gradient descent). Genellikle daha yüksek performans ama dikkatli tune gerektirir.
>
> Pratik kural: RF hızlı baseline için, XGBoost production ve Kaggle için.

---

**S: Cross-validation neden önemli? Türleri?**
> Modelin eğitim dışı veri üzerindeki gerçek performansını ölçer.
>
> - **K-Fold:** veriyi k parçaya böl, her parça bir kez test seti olsun
> - **Stratified K-Fold:** sınıf dağılımını koru (imbalanced için)
> - **Time Series Split:** temporal veri için — gelecek verisini test için kullan
> - **Leave-One-Out (LOO):** küçük veride, k=n

```python
from sklearn.model_selection import StratifiedKFold, cross_val_score
cv = StratifiedKFold(n_splits=5, shuffle=True, random_state=42)
scores = cross_val_score(model, X, y, cv=cv, scoring="f1")
```

---

**S: Özellik önemi (feature importance) nasıl hesaplanır?**
> - **Tree-based:** impurity azalmasının ağırlıklı ortalaması (MDI)
> - **Permutation importance:** özelliği karıştır, performans düşüşüne bak
> - **SHAP:** her örnek için katkıyı açıklar (en güvenilir)
>
> ⚠ MDI, yüksek kardinaliteli özellikleri abarttığı için SHAP tercih edilir.

---

**S: KNN'in dezavantajları?**
> 1. **Tahmin yavaş:** Her tahmin için tüm eğitim verisiyle mesafe hesaplar O(n)
> 2. **Yüksek boyutlarda çöker:** Curse of dimensionality
> 3. **Bellek:** Tüm eğitim verisini saklar
> 4. **Ölçeklendirme şart:** Mesafe bazlı → normalize etmezsen yanlış sonuç

---

**S: SVM'de kernel trick nedir?**
> Doğrusal ayrıştırılamayan veriyi yüksek boyutlu uzaya taşıyıp doğrusal ayırma yapmak. Fakat bu dönüşümü explicit yapmak pahalıdır.
> Kernel trick: Dönüşüm yapmadan, dönüşmüş uzaydaki iç çarpımı hesaplar.
> - RBF kernel: `K(x,y) = exp(-γ‖x-y‖²)` → sonsuz boyutlu uzay

---

### Hızlı Kavram Tablosu

| Kavram | 1 Cümle |
|---|---|
| Dropout | Eğitimde nöronları rastgele sıfırla → overfitting'i önler |
| Batch Norm | Mini-batch içi normaliz. → eğitimi hızlandırır |
| Gradient Descent | Loss fonksiyonunu minimize etmek için ağırlıkları güncelle |
| Early Stopping | Validation loss artmaya başlarsa eğitimi durdur |
| Data Augmentation | Eğitim verisi çeşitliliğini artır → augment et (flip, crop vb.) |

---

## 💡 Bağlantılar
- [[ML - Makine Öğrenmesine Giriş]]
- [[ML - Bias-Variance Dengesi ve Regularization]]
- [[ML - Model Değerlendirme Metrikleri]]
- [[Interview - İstatistik ve Olasılık Soruları]]

## ❓ Sorular / Anlamadıklarım

## 🔗 Kaynaklar
- [Chip Huyen - ML Interviews](https://huyenchip.com/ml-interviews-book/)
