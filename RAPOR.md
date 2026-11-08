# Salary Prediction – Proje Raporu

## 1. Amaç
Derste yapılan Linear Regression modeli maaş tahmininde R2 = 0.67 skoru verdi.
Bu projede amaç, veriyi daha doğru hazırlayarak bu skoru yükseltmek ve
farklı bir model deneyip sonuçları karşılaştırmaktır.

## 2. Veri Seti
- `Salary_Data.csv`: 6704 satır, 6 sütun
- Sütunlar: Age, Gender, Education Level, Job Title, Years of Experience, Salary
- Hedef değişken (y): **Salary**

## 3. Exploratory Data Analysis (EDA)
| Gözlem | Sonuç |
|---|---|
| Missing values | En fazla 5 eksik değer var, çok az |
| Duplicate satırlar | 4912 satır kopya, sonuçları etkileyebilir |
| Education Level | 7 farklı yazım var ama aslında 4 seviye (Bachelor's / Bachelor's Degree gibi) |
| Box plot | Eğitim seviyesi arttıkça maaş artıyor |
| Correlation | Years of Experience (0.81) ve Age (0.73) maaşla güçlü ilişkili |
| Scatter plot | Experience – Salary ilişkisi düz değil, eğri |
| describe() | Minimum maaş 350, outlier olabilir |

## 4. Derste Yapılandan Farklı Olarak Ne Yaptım?

| Adım | Derste yapılan | Benim yaptığım | Neden |
|---|---|---|---|
| Train/Test Split | Tüm işlemlerden sonra | En başta (%20 test) | Data leakage olmaması için |
| Education Level | LabelEncoder (alfabetik, 7 değer) | Ordinal encoding (0–3) | Aynı seviyeleri birleştirdim ve sırayı korudum |
| Job Title | LabelEncoder (193 sayı) | Rare class → "Other" + one-hot | Ünvanların sırası yok, sayı vermek modeli yanıltıyor |
| Age | Silindi | Tutuldu | Maaşla 0.73 korelasyonu var |
| Experience | Olduğu gibi | Experience² eklendi | İlişki eğri olduğu için |
| Outlier / Encoding / Scaling | Tüm veriden | Sadece train'den öğrenildi, test'e uygulandı | Data leakage olmaması için |

## 5. Pipeline (Akış)
1. Salary'si boş satırları sil. Burada bir şey hesaplanmadığı için leakage yok.
2. Train / Test Split: 5359 train, 1340 test
3. Kalan eksik satırları train ve test'te ayrı ayrı sil
4. Outlier: IQR sınırları y_train'den hesaplandı (-65.000 / 295.000). Satır silinmedi.
5. Education Level → 0, 1, 2, 3
6. Gender → LabelEncoder (train'e fit, test'e transform)
7. Job Title → train'de 5'ten az geçenler "Other" + one-hot (64 ünvan)
8. Experience_sq eklendi
9. StandardScaler (train'e fit, test'e transform)
10. Linear Regression

## 6. Sonuçlar

### Linear Regression
| Metrik | Derste yapılan | Benim modelim |
|---|---|---|
| R2 | 0.67 | **0.88** |
| MAE | 24.448 | **13.484** |

Aynı model, sadece veri hazırlığı değişti. Ortalama hata yaklaşık yarıya indi.

### Deneme 1: Decision Tree
- max_depth 2'den 20'ye kadar denendi
- Depth 2'de 0.71 (underfitting), depth 20'de **0.97**
- Veride çok fazla kopya satır olduğu için ezber şüphesi doğdu

### Deneme 2: Duplicate Satırları Silmek
Train'deki kopyaları ve train'de de bulunan test satırlarını sildim
(Train: 1557, Test: 212 satır).

| Model | Kopyalı | Kopyasız |
|---|---|---|
| Linear Regression | 0.88 | 0.87 |
| Decision Tree (max_depth=20) | 0.97 | 0.86 |

Linear Regression neredeyse hiç değişmedi. Decision tree ise 0.97'den 0.86'ya düştü,
yani yüksek skor test'teki kopyaları ezberlemekten geliyordu.

## 7. Final Karar
**Final model: Linear Regression (R2 = 0.88, kopyasız testte 0.87)**

- Kopyasız (dürüst) testte decision tree'den daha iyi
- Daha stabil: iki durumda da yaklaşık 0.87–0.88
- Daha basit ve yorumlanabilir

## 8. Sınırlamalar
- IQR yöntemi 350 gibi mantıksız düşük maaşları yakalayamadı (alt sınır negatif çıktı)
- Kopyasız test seti küçük (212 satır), bu yüzden skorlar biraz oynayabilir
- Bachelor's ile Bachelor's Degree box plot'ta farklı maaşlar gösterdi ama aynı diploma oldukları için birleştirdim

## 9. Kullanılan Araçlar
Python, pandas, seaborn, matplotlib, scikit-learn
