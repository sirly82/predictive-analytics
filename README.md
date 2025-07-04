# Prediksi Risiko Diabetes

# Laporan Proyek Machine Learning - Sirly Ziadatul Mustafidah

## Domain Proyek

Diabetes merupakan salah satu penyakit kronis paling umum di dunia, yang memengaruhi jutaan orang dan menyebabkan komplikasi kesehatan serius jika tidak ditangani secara dini. Diagnosis dini dan prediksi risiko diabetes sangat penting untuk mencegah komplikasi lebih lanjut, mengingat biaya pengobatan yang mahal dan beban kesehatan global yang signifikan.

Dalam konteks ini, penerapan machine learning untuk prediksi risiko diabetes dapat menjadi solusi berbasis data yang efisien, akurat, dan skalabel. Dengan memanfaatkan dataset klinis, seperti *Pima Indians Diabetes Dataset*, kita dapat mengembangkan model prediktif yang membantu tenaga medis maupun sistem informasi kesehatan dalam mengambil keputusan yang lebih baik.

Referensi:

[1] C. Z. V. Junus, T. and P. Kartikasari, "Klasifikasi menggunakan metode Support Vector Machine dan Random Forest untuk deteksi awal risiko diabetes melitus," *Jurnal Gauss*, vol. 11, no. 3, pp. 386-396, 2023, doi: 10.14710/j.gauss.11.3.386-396.

[2] A. Pratama, A. C. Nurcahyo, and L. Firgia, "Penerapan machine learning dengan algoritma logistik regresi untuk memprediksi diabetes," in *Seminar Nasional CORISINDO*, Kubu Raya, Indonesia, 2022, pp. 116-122.

## Business Understanding

Pada bagian ini, kamu perlu menjelaskan proses klarifikasi masalah.

Bagian laporan ini mencakup:

### Problem Statements

- Bagaimana memprediksi apakah seseorang berisiko terkena diabetes berdasarkan data medis?
- Model machine learning apa yang paling optimal untuk klasifikasi risiko diabetes menggunakan dataset Pima Indians?

### Goals

- Membangun model klasifikasi yang mampu mengidentifikasi individu berisiko diabetes dengan performa tinggi.
- Membandingkan beberapa algoritma machine learning untuk menentukan model terbaik.

### Solution Statements

- Membangun dua model klasifikasi: Random Forest dan XGBoost.
- Melakukan hyperparameter tuning untuk masing-masing model guna meningkatkan performa prediksi.
- Menggunakan metrik F1-score, recall, dan accuracy untuk mengevaluasi performa model.

## Data Understanding

Dataset yang digunakan adalah **Pima Indians Diabetes Dataset** dari [UCI Machine Learning Repository](https://www.kaggle.com/datasets/uciml/pima-indians-diabetes-database).

Dataset ini terdiri dari 768 baris dan 9 kolom (8 fitur, 1 label `Outcome`).

### Variabel-variabel dalam dataset:
- `Pregnancies`: Jumlah kehamilan
- `Glucose`: Kadar glukosa plasma
- `BloodPressure`: Tekanan darah diastolik
- `SkinThickness`: Ketebalan lipatan kulit triceps
- `Insulin`: Kadar insulin serum
- `BMI`: Indeks massa tubuh
- `DiabetesPedigreeFunction`: Fungsi silsilah diabetes (genetik)
- `Age`: Usia pasien
- `Outcome`: Label target (1: diabetes, 0: tidak diabetes)

### Exploratory Data Analysis
- Beberapa fitur seperti Glucose, BloodPressure, dan Insulin memiliki nilai 0 yang tidak mungkin secara medis dan perlu ditangani.
- Korelasi tertinggi terhadap Outcome ditemukan pada `Glucose` dan `BMI`.

## Data Preparation

Pada tahap ini, dilakukan pembersihan data berupa:
- Mengganti nilai nol yang tidak valid pada beberapa fitur medis menjadi nilai yang lebih representatif. Nilai 0 pada kolom 'Glucose', 'BloodPressure', 'SkinThickness', 'Insulin', dan 'BMI' dianggap tidak masuk akal secara medis, sehingga perlu diganti

```python
# Mengganti nilai 0 menjadi NaN pada kolom yang tidak mungkin bernilai 0
cols_with_zero = ['Glucose', 'BloodPressure', 'SkinThickness', 'Insulin', 'BMI']
diabetes_df[cols_with_zero] = diabetes_df[cols_with_zero].replace(0, np.nan)
```

- Mengisi nilai kosong (NaN) yang sudah diubah tadi akan diisi dengan median dari masing-masing kolom.

```python
# Isi NaN dengan median
diabetes_df.fillna(diabetes_df.median(), inplace=True)
```

- Melakukan standardisasi data menggunakan `StandardScaler`.

```python
# Scaling data
from sklearn.preprocessing import StandardScaler
scaler = StandardScaler()
X_scaled = scaler.fit_transform(df.drop('Outcome', axis=1))
```

- Memisahkan data menjadi train-test split (80:20) menggunakan `train_test_split`.

```python
# Split data
from sklearn.model_selection import train_test_split
X_train, X_test, y_train, y_test = train_test_split(X_scaled, df['Outcome'], test_size=0.2, random_state=42) 
```

## Model Development (Modelling)

### Model yang digunakan:
1. **Logistic Regression**:
   - Digunakan sebagai baseline model.
   - Menggunakan parameter default (`solver='lbfgs'`, `max_iter=100`).
   - Model ini memprediksi probabilitas dengan fungsi sigmoid dari kombinasi linear fitur.
   - Akurasi cukup baik namun tidak cukup kuat untuk pola non-linear.

2. **Random Forest**:
   - Model ensemble berbasis pohon keputusan (decision trees).
   - Parameter sebelum tuning:  
     - `n_estimators=50` (jumlah pohon)  
     - `max_depth=16` (kedalaman maksimal pohon)  
   - Parameter setelah tuning:  
     - `n_estimators=50`  
     - `max_depth=10`  
     - `min_samples_split=10`  
     - `min_samples_leaf=1`
   - Random Forest membangun banyak pohon keputusan secara acak dan menggabungkan prediksi untuk meningkatkan akurasi dan mengurangi overfitting.
   - Hasil tuning menunjukkan peningkatan akurasi dan F1-score.

3. **XGBoost**:
   - Model boosting yang populer dan sangat efektif untuk data kompleks.
   - Parameter yang digunakan:  
     - `max_depth=4`  
     - `n_estimators=100`  
     - `learning_rate=0.1`  
     - `subsample=0.8`  
     - `colsample_bytree=0.8`
   - XGBoost membangun model secara bertahap dengan memperbaiki kesalahan model sebelumnya melalui weighted trees.

### Kelebihan dan Kekurangan
1. **Logistic Regression**
  - Kelebihan: Cepat, mudah diinterpretasi, baseline yang baik.
  - Kekurangan: Tidak bisa menangkap hubungan non-linear, sensitivitas terhadap outlier.

2. **Random Forest**
  - Kelebihan: Kuat terhadap overfitting, dapat menangani fitur non-linear, feature importance jelas.
  - Kekurangan: Kurang efisien untuk dataset besar, hasil kurang transparan.

3. **XGBoost**
  - Kelebihan: Akurasi tinggi, mendukung regularisasi, sangat efektif untuk data kompleks.
  - Kekurangan: Kompleks, butuh tuning parameter lebih banyak, waktu pelatihan lebih lama.

### Proses Improvement Model
- Pada modeling menggunakan **Random Forest**, dilakukan tuning hyperparameter dengan grid search pada parameter `n_estimators`, `max_depth`, `min_samples_split`, dan `min_samples_leaf`.  
- Tuning ini menghasilkan peningkatan performa model dari akurasi sekitar 75% menjadi sekitar 76%.  
- Parameter lain pada XGBoost juga disetel untuk mengoptimalkan keseimbangan antara bias dan varians.

### Pemilihan Model Terbaik
- Berdasarkan hasil evaluasi akurasi dan f1-score, **XGBoost** memberikan performa terbaik secara keseluruhan.  
- Model ini mampu menangkap pola kompleks dalam data dengan regularisasi yang membantu menghindari overfitting.  
- Meskipun waktu pelatihan lebih lama, peningkatan akurasi dan stabilitas prediksi membuat XGBoost menjadi pilihan utama untuk prediksi risiko diabetes pada dataset ini.

**Rubrik/Kriteria Tambahan (Opsional)**: 
- Menjelaskan kelebihan dan kekurangan dari setiap algoritma yang digunakan.
- Jika menggunakan satu algoritma pada solution statement, lakukan proses improvement terhadap model dengan hyperparameter tuning. **Jelaskan proses improvement yang dilakukan**.
- Jika menggunakan dua atau lebih algoritma pada solution statement, maka pilih model terbaik sebagai solusi. **Jelaskan mengapa memilih model tersebut sebagai model terbaik**.

## Evaluation

### Metrik Evaluasi:

Metrik evaluasi yang digunakan adalah:

- **Accuracy**: Proporsi prediksi benar.
- **Precision**: Akurasi prediksi positif.
- **Recall**: Kemampuan model mendeteksi kasus positif.
- **F1-score**: Harmonik rata-rata precision dan recall.

| Model             | Accuracy | Precision (kelas 1) | Recall (kelas 1) | F1-score (kelas 1) |
|-------------------|----------|---------------------|------------------|--------------------|
| Logistic Regression| 75.3%    | 0.67                | 0.62             | 0.64               |
| Random Forest     | 75.97%   | 0.66                | 0.67             | 0.67               |
| XGBoost           | 75.97%   | 0.65                | 0.71             | 0.68               |

### Formula Metrik Evaluasi
- Accuracy = (TP + TN) / (TP + TN + FP + FN)
- Precision = TP / (TP + FP)
- Recall = TP / (TP + FN)
- F1-score = 2 * (Precision * Recall) / (Precision + Recall)

dimana TP = True Positives, TN = True Negatives, FP = False Positives, FN = False Negatives.

### Hasil
- **XGBoost** memberikan performa terbaik khususnya pada metrik recall (0.71) dan F1-score (0.68), menunjukkan kemampuannya dalam mendeteksi risiko diabetes dengan lebih sensitif.
- **Random Forest dengan tuning** juga menunjukkan hasil yang sangat kompetitif dengan akurasi dan F1-score yang hampir sama dengan XGBoost, serta performa recall yang cukup baik (0.67).
- **Logistic Regression** berperan sebagai baseline yang sederhana dan cepat, dengan akurasi 75.3%, namun nilai recall dan F1-score-nya sedikit lebih rendah dibanding dua model ensemble.

**---Ini adalah bagian akhir laporan---**