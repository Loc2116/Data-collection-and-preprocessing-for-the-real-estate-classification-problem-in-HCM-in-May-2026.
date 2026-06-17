# README — Pipeline phân loại phân khúc giá bất động sản TP.HCM

Dự án này xây dựng pipeline đầu cuối để thu thập, làm sạch, phân tích, tăng cường đặc trưng và phân loại phân khúc giá bất động sản tại TP.HCM.

Pipeline gồm 4 notebook chính:

```text
1. Crawler_bds.ipynb
2. 02_data_cleaning_classification.ipynb
3. 03_EDA_FeatureEngineering.ipynb
4. 04_ClassifyModeling_-_V2_test.ipynb
```

---

## 1. Mục tiêu bài toán

Input của mô hình là các thuộc tính của tin rao bất động sản như diện tích, số tầng, độ rộng đường vào, tọa độ, pháp lý, nội thất, quận và các đặc trưng tiện ích trích từ mô tả bằng Qwen.

Output là nhãn phân khúc giá gồm 5 lớp có thứ tự:

```text
0. Bình dân
1. Trung cấp
2. Cao cấp
3. Hạng sang
4. Siêu sang
```

Trong notebook modeling hiện tại, nhãn được tạo bằng KMeans trên đơn giá/m², tức:

```text
gia_m2 = (Khoảng giá * 1000) / Diện tích
```

`Khoảng giá` có đơn vị tỷ VNĐ, còn `gia_m2` có đơn vị triệu VNĐ/m².

---

## 2. Cấu trúc thư mục khuyến nghị

Đặt các notebook và dữ liệu theo cấu trúc sau:

```text
Tien_xu_ly_du_lieu_BDS/
│
├── notebooks/
│   ├── Crawler_bds.ipynb
│   ├── 02_data_cleaning_classification.ipynb
│   ├── 03_EDA_FeatureEngineering.ipynb
│   └── 04_ClassifyModeling_-_V2_test.ipynb
│
├── data/
│   ├── Raw/
│   │   ├── data_bds.csv
│   │   └── data_bds_NaNFill.csv
│   │
│   └── Processed/
│       ├── Cleaned_data/
│       │   ├── Cleaned_data.csv
│       │   ├── X_train_cleaned.csv
│       │   ├── X_test_cleaned.csv
│       │   ├── y_train_cleaned.csv
│       │   └── y_test_cleaned.csv
│       │
│       ├── Cleaned_LLM/
│       │   └── Cleaned_data_qwen.csv
│       │
│       └── Processed_data/
│           ├── X_train_processed.csv
│           ├── X_test_processed.csv
│           ├── y_train_processed.csv
│           ├── y_test_processed.csv
│           └── ModelReady/
│               ├── best_pipeline_XGBoost.pkl
│               ├── cv_results.csv
│               ├── X_train_model.csv
│               ├── X_test_model.csv
│               ├── y_train_encoded.csv
│               ├── y_test_encoded.csv
│               ├── class_distribution_grouped.png
│               ├── cv_macro_f1_boxplot.png
│               ├── confusion_matrix.png
│               └── feature_importance.png
│
├── CODEBOOK.md
├── README.md
└── requirements.txt
```

Các notebook hiện có dùng `Path.cwd().parent.parent` để suy ra `PROJECT_ROOT`. Vì vậy, khi chạy trong môi trường mới, cần kiểm tra lại biến đường dẫn ở đầu mỗi notebook. Nếu notebook không nằm đúng cấp thư mục như ban đầu, hãy sửa trực tiếp:

```python
PROJECT_ROOT = Path("đường/dẫn/tới/Tien_xu_ly_du_lieu_BDS")
```

---

## 3. Cài đặt môi trường

### Bước 1 — Tạo môi trường ảo

Trên Windows PowerShell:

```bash
python -m venv .venv
.venv\Scripts\activate
```

Trên macOS/Linux:

```bash
python -m venv .venv
source .venv/bin/activate
```

### Bước 2 — Cài thư viện

```bash
pip install --upgrade pip
pip install -r requirements.txt
```

### Bước 3 — Mở Jupyter Notebook

```bash
jupyter notebook
```

hoặc:

```bash
jupyter lab
```

---

## 4. Chạy pipeline từ A-Z

## 4.1. NB1 — Crawl dữ liệu

Notebook:

```text
Crawler_bds.ipynb
```

Chức năng chính:

1. Tạo Chrome driver bằng `undetected_chromedriver`.
2. Truy cập trang listing của `batdongsan.com.vn`.
3. Lấy danh sách URL tin rao.
4. Lưu URL vào `hrefs.txt`.
5. Truy cập từng URL chi tiết.
6. Thu thập các cột thô:
   - `Link`
   - `Địa chỉ`
   - `Khoảng giá`
   - `Diện tích`
   - `Số phòng ngủ`
   - `Số phòng tắm, vệ sinh`
   - `Số tầng`
   - `Hướng nhà`
   - `Hướng ban công`
   - `Đường vào`
   - `Pháp lý`
   - `Nội thất`
   - `Latitude`
   - `Longitude`
   - `Mô tả`
7. Lưu ra `data/Raw/data_bds.csv`.
8. Trích thêm một số thuộc tính từ `Mô tả`, điền vào các ô thiếu nếu có, rồi lưu `data/Raw/data_bds_NaNFill.csv`.

Trước khi chạy, kiểm tra các biến quan trọng:

```python
BASE_URL = "https://batdongsan.com.vn/ban-nha-dat-tp-hcm"
TARGET_SAMPLES = 10
CSV_FILE = ".../data/Raw/data_bds.csv"
```

Khi cần crawl dữ liệu thật cho báo cáo, đổi `TARGET_SAMPLES` thành số mẫu mong muốn.

Lưu ý:
- Cần cài Google Chrome.
- Nếu Chrome version thay đổi, sửa `version_main` trong hàm `create_driver()`.
- Nếu website thay đổi HTML/CSS selector, các selector trong notebook cần được cập nhật.

---

## 4.2. NB2 — Làm sạch dữ liệu

Notebook:

```text
02_data_cleaning_classification.ipynb
```

Input chính:

```text
data/Raw/data_bds.csv
```

hoặc:

```text
data/Raw/data_bds_NaNFill.csv
```

Chức năng chính:

1. Tự động tìm file raw trong `data/Raw`.
2. Chuẩn hóa Unicode và khoảng trắng cho các cột chữ.
3. Xóa dấu chấm cuối câu nếu có.
4. Chuẩn hóa chữ thường với các cột danh mục.
5. Chuẩn hóa `Pháp lý`.
6. Chuẩn hóa `Nội thất`.
7. Chuyển `Khoảng giá` về đơn vị tỷ VNĐ.
8. Chuyển các cột số về dạng numeric:
   - `Diện tích`
   - `Số phòng ngủ`
   - `Số phòng tắm, vệ sinh`
   - `Số tầng`
   - `Đường vào`
9. Xử lý trùng lặp theo các cột chính:
   - `Địa chỉ`
   - `Khoảng giá`
   - `Diện tích`
   - `Số phòng ngủ`
   - `Số phòng tắm, vệ sinh`
   - `Số tầng`
10. Loại bản ghi rỗng.
11. Trích `Loại đường vào` từ `Mô tả`.
12. Phân rã `Địa chỉ` thành:
   - `Đường`
   - `Phường/Xã`
   - `Quận`
13. Loại `Hướng nhà` và `Hướng ban công` vì thiếu quá nhiều.
14. Chia train/test trước khi xử lý thiếu để tránh leakage.
15. Xử lý thiếu:
   - `Pháp lý`, `Nội thất`: điền nhóm phù hợp.
   - `Số phòng tắm, vệ sinh`: KNN Imputer.
   - `Số tầng`: mô hình kiểu MissForest/IterativeImputer + RandomForest.
   - `Đường vào`: giữ missing có chủ đích và tạo flag.
16. Xử lý ngoại lai:
   - loại outlier `Đường vào > 50m`.
   - kiểm tra outlier số tầng.
17. Lưu dữ liệu đã làm sạch.

Output:

```text
data/Processed/Cleaned_data/X_train_cleaned.csv
data/Processed/Cleaned_data/y_train_cleaned.csv
data/Processed/Cleaned_data/X_test_cleaned.csv
data/Processed/Cleaned_data/y_test_cleaned.csv
data/Processed/Cleaned_data/Cleaned_data.csv
```

Theo log notebook, file `Cleaned_data.csv` là file gộp lại để phục vụ EDA và có cột `Loại_Dữ_Liệu` để đánh dấu `Train`/`Test`.

---

## 4.3. Chuẩn bị file Qwen

Notebook NB3 hiện dùng file:

```text
data/Processed/Cleaned_LLM/Cleaned_data_qwen.csv
```

File này là bản đã tăng cường thêm các đặc trưng nhị phân từ trường `Mô tả`, gồm:

```text
hem_xe_hoi
gan_cho_sieu_thi
gan_truong_hoc
gan_benh_vien
gan_cong_vien_ho_nuoc
```

Trong notebook, phần gọi LLM/Qwen được ghi chú là chạy ở file riêng trên Kaggle do giới hạn phần cứng. Vì vậy, trước khi chạy đầy đủ NB3/NB4, cần bảo đảm đã có:

```text
data/Processed/Cleaned_LLM/Cleaned_data_qwen.csv
```

Nếu chưa có file này, có 2 cách:

1. Chạy notebook Qwen riêng để tạo `Cleaned_data_qwen.csv`.
2. Tạm thời dùng `Cleaned_data.csv` thay thế, nhưng phải bỏ hoặc sửa các đoạn code yêu cầu 5 cột Qwen.

---

## 4.4. NB3 — EDA và Feature Engineering

Notebook:

```text
03_EDA_FeatureEngineering.ipynb
```

Input:

```text
data/Raw/data_bds.csv
data/Processed/Cleaned_data/Cleaned_data.csv
data/Processed/Cleaned_LLM/Cleaned_data_qwen.csv
```

Chức năng chính:

1. EDA dữ liệu raw:
   - phân tích missing value.
   - phân phối giá.
   - phân phối diện tích.
   - phân phối pháp lý/nội thất.
   - phân phối số phòng ngủ/số tầng.
   - phân bố tọa độ địa lý.
2. EDA dữ liệu đã làm sạch:
   - kiểm tra kiểu dữ liệu.
   - thống kê missing.
   - phân tích `Khoảng giá`.
   - tạo `Giá_trên_m2` để phân tích.
   - kiểm định ANOVA.
   - kiểm tra VIF.
   - phân tích heatmap `Quận × Loại đường vào`.
3. Tách lại train/test từ cột `Loại_Dữ_Liệu`.
4. Tạo `Tổng số phòng`:

```python
Tổng số phòng = Số phòng ngủ + Số phòng tắm, vệ sinh
```

5. One-hot encoding:
   - `Pháp lý`
   - `Nội thất`
6. Ordinal encoding:

```text
Chưa xác định -> 0
Hẻm xe máy    -> 1
Hẻm ô tô      -> 2
Mặt tiền      -> 3
unknown       -> -1
```

7. Smoothed target encoding cho `Quận`:

```text
Quận_encoded = (n_i * mean_i + m * global_mean) / (n_i + m)
```

với `m = 10`.

8. Lưu tập dữ liệu đã xử lý.

Output:

```text
data/Processed/Processed_data/X_train_processed.csv
data/Processed/Processed_data/y_train_processed.csv
data/Processed/Processed_data/X_test_processed.csv
data/Processed/Processed_data/y_test_processed.csv
```

Theo log notebook:

```text
X_train_processed.csv : (2901, 25)
y_train_processed.csv : (2901, 1)
X_test_processed.csv  : (727, 25)
y_test_processed.csv  : (727, 1)
```

---

## 4.5. NB4 — Huấn luyện và đánh giá mô hình

Notebook:

```text
04_ClassifyModeling_-_V2_test.ipynb
```

Input:

```text
data/Processed/Processed_data/X_train_processed.csv
data/Processed/Processed_data/y_train_processed.csv
data/Processed/Processed_data/X_test_processed.csv
data/Processed/Processed_data/y_test_processed.csv
```

Chức năng chính:

1. Load train/test đã xử lý.
2. Kiểm tra và loại các cột leakage nếu xuất hiện.
3. Chỉ giữ cột số để mô hình hóa.
4. Tạo đơn giá/m²:

```python
gia_m2 = (Khoảng giá * 1000) / Diện tích
```

5. Tạo nhãn 5 lớp bằng KMeans trên `gia_m2_train`.
6. Xây dựng preprocessing:
   - `SimpleImputer(strategy='median')`
   - `RobustScaler()`
7. Huấn luyện 5 mô hình:
   - Logistic Regression
   - Random Forest
   - Extra Trees
   - SVM RBF
   - XGBoost
8. Đánh giá bằng Stratified 5-Fold Cross-Validation.
9. Chọn mô hình tốt nhất theo Macro F1.
10. Đánh giá trên test set.
11. Lưu mô hình, bảng kết quả và hình ảnh.

Output:

```text
data/Processed/Processed_data/ModelReady/best_pipeline_XGBoost.pkl
data/Processed/Processed_data/ModelReady/cv_results.csv
data/Processed/Processed_data/ModelReady/X_train_model.csv
data/Processed/Processed_data/ModelReady/X_test_model.csv
data/Processed/Processed_data/ModelReady/y_train_encoded.csv
data/Processed/Processed_data/ModelReady/y_test_encoded.csv
data/Processed/Processed_data/ModelReady/class_distribution_grouped.png
data/Processed/Processed_data/ModelReady/cv_macro_f1_boxplot.png
data/Processed/Processed_data/ModelReady/confusion_matrix.png
data/Processed/Processed_data/ModelReady/feature_importance.png
```

Kết quả trong notebook hiện tại:

```text
Best model        : XGBoost
CV Macro F1       : 0.6107 ± 0.0265
Test Accuracy     : 0.6919
Test Macro F1     : 0.6322
Test Weighted F1  : 0.6942
```

Top 3 đặc trưng quan trọng nhất:

```text
1. Quận_encoded
2. hem_xe_hoi
3. Số tầng
```

---

## 5. Cách chạy nhanh bằng thứ tự thủ công

Mở Jupyter Notebook và chạy lần lượt:

```text
Bước 1: Crawler_bds.ipynb
Bước 2: 02_data_cleaning_classification.ipynb
Bước 3: Chuẩn bị Cleaned_data_qwen.csv
Bước 4: 03_EDA_FeatureEngineering.ipynb
Bước 5: 04_ClassifyModeling_-_V2_test.ipynb
```

Sau mỗi notebook, kiểm tra các file output đã xuất hiện đúng thư mục hay chưa.

---

## 6. Kiểm tra nhanh sau khi chạy

Sau NB2, cần có:

```text
data/Processed/Cleaned_data/Cleaned_data.csv
```

Sau bước Qwen, cần có:

```text
data/Processed/Cleaned_LLM/Cleaned_data_qwen.csv
```

Sau NB3, cần có:

```text
data/Processed/Processed_data/X_train_processed.csv
data/Processed/Processed_data/X_test_processed.csv
data/Processed/Processed_data/y_train_processed.csv
data/Processed/Processed_data/y_test_processed.csv
```

Sau NB4, cần có:

```text
data/Processed/Processed_data/ModelReady/best_pipeline_XGBoost.pkl
data/Processed/Processed_data/ModelReady/cv_results.csv
```

---

## 7. Một số lỗi thường gặp

### Lỗi 1 — Không tìm thấy file CSV

Nguyên nhân thường là sai `PROJECT_ROOT` hoặc notebook đang chạy từ thư mục khác.

Cách sửa:

```python
from pathlib import Path
PROJECT_ROOT = Path(r"C:\Users\ADMIN\Tien_xu_ly_du_lieu_BDS")
```

Sau đó kiểm tra lại:

```python
RAW_DIR = PROJECT_ROOT / "data" / "Raw"
PROCESSED_DIR = PROJECT_ROOT / "data" / "Processed" / "Processed_data"
```

### Lỗi 2 — Không chạy được Chrome driver

Kiểm tra:
- Đã cài Google Chrome chưa.
- `version_main` trong `create_driver()` có khớp Chrome version không.
- Đã cài `selenium` và `undetected-chromedriver` chưa.

Cài lại:

```bash
pip install selenium undetected-chromedriver "setuptools<70.0.0"
```

### Lỗi 3 — Không có `Cleaned_data_qwen.csv`

NB3 cần file này để lấy 5 đặc trưng Qwen. Cần chạy phần Qwen riêng hoặc chuẩn bị file trước khi chạy NB3 phần feature engineering.

### Lỗi 4 — Lỗi `OneHotEncoder` do khác version scikit-learn

Notebook đã có đoạn xử lý tương thích:

```python
try:
    ohe = OneHotEncoder(sparse_output=False, drop='first', handle_unknown='ignore')
except TypeError:
    ohe = OneHotEncoder(sparse=False, drop='first', handle_unknown='ignore')
```

Nếu vẫn lỗi, kiểm tra version:

```python
import sklearn
print(sklearn.__version__)
```

### Lỗi 5 — Kết quả nhãn thay đổi

Nhãn 5 lớp được tạo bằng KMeans trên train set. Nếu dữ liệu, random seed hoặc cách chia train/test thay đổi, ranh giới phân khúc có thể thay đổi. Để tái lập, giữ nguyên:
- dữ liệu đầu vào.
- `random_state=42`.
- cách chia train/test.
- đoạn code tạo `BREAKS`.

---

## 8. Gợi ý cải thiện để tái lập tốt hơn

Nên lưu thêm các file sau trong NB4:

```text
label_breaks.json
label_mapping.json
quan_target_encoding_mapping.csv
```

Lý do:
- `label_breaks.json` giúp tái lập ngưỡng nhãn 5 lớp.
- `label_mapping.json` giúp tránh nhầm thứ tự nhãn.
- `quan_target_encoding_mapping.csv` giúp áp dụng đúng `Quận_encoded` cho dữ liệu mới.
