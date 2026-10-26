# Pima Indians Diabetes Veri Seti Üzerinde Çok Yöntemli Veri Madenciliği Analizi

## Proje Hakkında

Bu çalışma; **Pima Indians Diabetes** veri seti (`diabetes.csv`) ile denek kimlik bilgilerini içeren simülasyon tablosunu (`isim_listesi1.csv`) temel alarak diyabet hastalığını anlamak, klinik risk seviyelerini segmentlere ayırmak ve olasılıksal/istatistiksel modeller üzerinden tahminler üretmek amacıyla geliştirilmiş çok yöntemli bir veri madenciliği projesidir[cite: 11, 12].

Proje, yalnızca tekil bir sınıflandırma algoritmasıyla yetinmeyip **5 ana aşamadan** oluşan uçtan uca bir analitik iş akışı sunar[cite: 11]:
1. **Veri Ön İşleme (Preprocessing):** Eksik veri doldurma simülasyonu, birleştirme ve Min-Max normalizasyonu[cite: 11, 12, 13].
2. **Eğri Uydurma / Regresyon:** Klinik biyobelirteçlerle Beden Kitle İndeksi (BMI) tahmini ve eşik tabanlı risk sınıflandırması ($BMI \ge 25$)[cite: 11, 14].
3. **Kümeleme Analizi (K-Means):** Gözetimsiz segmentasyon ile 3 farklı risk kümesinin (Düşük, Orta, Yüksek Risk) çıkarılması ve Lojistik Regresyon ile doğrulanması[cite: 11, 18, 19, 20].
4. **İstisna / Aykırı Değer (Outlier) Analizi:** IQR ve Z-Score yöntemleriyle uç değerlerin elenmesi, kutu grafikler ve Lojistik Regresyon başarım analizi[cite: 11, 23, 24, 25].
5. **Bayesian Ağları (Bayesian Network):** TAN (Tree Augmented Naive Bayes) yaklaşımı, CPT öğrenimi ve Variable Elimination ile klinik senaryo çıkarımları[cite: 11, 28, 30, 31, 32].

---

## Veri Seti ve Değişkenler

Veri seti 768 denekten elde edilen 8 klinik/biyokimyasal öznitelik ve 1 hedef teşhis sonucundan oluşmaktadır[cite: 11, 24]:

* **Pregnancies:** Gebelik sayısı[cite: 11, 24]
* **Glucose:** 2 saatlik plazma glukoz konsantrasyonu[cite: 11, 24]
* **BloodPressure:** Diyastolik kan basıncı ($mm\ Hg$)[cite: 11, 24]
* **SkinThickness:** Triseps deri kıvrım kalınlığı ($mm$)[cite: 11, 24]
* **Insulin:** 2 saatlik serum insülin değeri ($mu\ U/ml$)[cite: 11, 24]
* **BMI:** Beden kitle indeksi ($kg/m^2$)[cite: 11, 24]
* **DiabetesPedigreeFunction:** Aile geçmişine dayalı diyabet genetik risk skoru[cite: 11, 24]
* **Age:** Yaş (yıl)[cite: 11, 24]
* **Outcome (Hedef Değişken):** Teşhis sonucu (0: Sağlıklı / Negatif, 1: Diyabetik / Pozitif)[cite: 11, 24]

---

## Metodoloji ve Bulgular

### 1. Veri Ön İşleme (Preprocessing)
* **Veri Bütünleştirme:** `diabetes.csv` ve `isim_listesi1.csv` indeks bazlı birleştirilerek tek bir çalışma tablosu oluşturulmuştur[cite: 12].
* **Eksik Veri Yönetimi:** Glucose, BloodPressure ve Insulin değişkenlerine rastgele %3 oranında (69 adet) yapay eksik değer yerleştirilmiş; ardından ortalama (Mean Imputation) ve deneysel mod doldurma yöntemleri uygulanmıştır[cite: 12].
* **Normalizasyon:** Uzaklık ve gradyan temelli algoritmaların dengeli çalışabilmesi için tüm sayısal nitelikler `MinMaxScaler` ile [0, 1] aralığına ölçeklendirilmiştir[cite: 13].

### 2. Regresyon ve Eğri Uydurma (Linear Regression)
* **Hedef:** Biyoklinik değişkenler yardımıyla BMI değerinin kestirimi[cite: 14].
* **Sonuçlar:** $MSE = 52.23$, $R^2 = 0.27$[cite: 15]. Doğrusal modelin sınırlı kalması, boy ve kilo değişkenlerinin veri setinde bulunmayışından ve biyobelirteç ilişkilerinin doğrusal olmamasından kaynaklanmıştır[cite: 16].
* **Eşik Tabanlı Sınıflama ($BMI \ge 25$):** Genel doğruluk %84 seviyesine ulaşsa da, Riskli sınıfındaki yüksek recall (0.96) karşısında Normal sınıfında düşük recall (0.17) gözlenmiş; sınıf dengesizliğinin etkisi tespit edilmiştir[cite: 15, 16, 17].

### 3. Kümeleme Analizi (K-Means) ve Sınıflandırma
* **Segmentasyon:** Glucose ve BMI değişkenleri üzerinden veriler normalize edilerek $k=3$ küme oluşturulmuştur[cite: 18, 19]:
  * **Küme 0 (Kırmızı):** Yüksek Glucose & Yüksek BMI $\rightarrow$ **%68 Diyabet Oranı (En Yüksek Risk)**[cite: 20]
  * **Küme 1 (Mavi):** Düşük Glucose & Düşük BMI $\rightarrow$ **%13 Diyabet Oranı (Düşük Risk)**[cite: 20]
  * **Küme 2 (Yeşil):** Orta Glucose & Yüksek BMI $\rightarrow$ **%31 Diyabet Oranı (Orta Risk)**[cite: 20]
* **Lojistik Regresyon Entegrasyonu:** Kümeleme sonrası eğitilen model test setinde **%76.62 Accuracy** değerine ulaşmıştır[cite: 21, 22, 23].

### 4. Aykırı Değer (Outlier) Analizi
* **Yöntemler:** IQR (Interquartile Range) ve $Z$-Score ($\vert{}Z\vert{} > 3$) kuralları uygulanarak kutu grafikler (Box Plot) ile görselleştirilmiştir[cite: 23, 24, 25].
* **Bulgular:** Özellikle BloodPressure (49 adet), Insulin (35 adet) ve DiabetesPedigreeFunction (29 adet) sütunlarında yoğun aykırı değer saptanmıştır[cite: 25, 28].
* **Modelleme:** Aykırı değer analizinin ardından eğitilen Lojistik Regresyon modeli ($max\_iter=500$), %80/%20 train-test ayrımında **%72.08 Accuracy** sağlamış; sağlıklı bireyleri tanımada ($TN=84$) yüksek başarı göstermiştir[cite: 25, 26, 27].

### 5. Bayesian Ağları ile Olasılıksal Modelleme (TAN)
* **Ağ Mimarisi:** `pgmpy` kütüphanesi ve TreeSearch kullanılarak Ağaç Destekli Naive Bayes (Tree Augmented Naive Bayes - TAN) yapısı kurulmuştur[cite: 28, 30]. Kök düğüm `Glucose`, sınıf düğümü `Outcome` olarak seçilmiştir[cite: 30].
* **Parametre Öğrenimi ve Çıkarım:** Koşullu olasılık tabloları (CPT) `MaximumLikelihoodEstimator` ile hesaplanmış, çıkarımlar `Variable Elimination` tekniği ile gerçekleştirilmiştir[cite: 31, 32].
* **Klinik Senaryo Çıkarımları:**
  * **Senaryo 1:** $P(\text{Outcome} = 1 \mid \text{Glucose}_{\%80\text{ üstü}}) \approx \mathbf{0.5556}$[cite: 32]
  * **Senaryo 2:** $P(\text{Outcome} = 1 \mid \text{BMI}_{\%75\text{ üstü}}) = \mathbf{0.6000}$[cite: 33]
  * **Senaryo 3:** BMI yüksek olduğunda yaş dağılımının ağırlıklı olarak orta yaş grubunda ($31-50$) toplandığı modellenmiştir[cite: 33, 34].

---

## Kullanılan Kütüphaneler ve Teknolojiler

* **Temel Dil:** Python 3.8+[cite: 12]
* **Veri Manipülasyonu & İstatistik:** `pandas`, `numpy`, `scipy`
* **Makine Öğrenmesi & Modelleme:** `scikit-learn`, `joblib`
* **Bayesian Ağları & Olasılıksal Çıkarım:** `pgmpy`, `networkx`
* **Görselleştirme:** `matplotlib`, `seaborn`
* **Ortam:** Jupyter Notebook
---

