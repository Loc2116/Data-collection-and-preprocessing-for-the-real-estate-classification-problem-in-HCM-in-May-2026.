# Code Book / Data Dictionary — Pipeline phân loại phân khúc giá BĐS TP.HCM

**Phiên bản:** 2.0 · **Cập nhật:** tháng 5/2026  
**Dataset:** [Property for Sale in HCMC on May 2026](https://www.kaggle.com/datasets/locdovan211/property-for-sale-in-hcmc-on-may-2026-dataset)  
**License:** CC BY 4.0

Tài liệu này mô tả dữ liệu được tạo và sử dụng trong pipeline 4 notebook:

1. `Crawler\_bds.ipynb` — thu thập dữ liệu từ batdongsan.com.vn.
2. `02\_data\_cleaning\_classification.ipynb` — làm sạch dữ liệu, chuẩn hóa kiểu dữ liệu, xử lý thiếu, xử lý ngoại lai.
3. `03\_EDA\_FeatureEngineering.ipynb` — EDA, tăng cường đặc trưng bằng Qwen, mã hóa đặc trưng, tạo tập train/test đã xử lý.
4. `04\_ClassifyModeling\_-\_V2\_test.ipynb` — tạo nhãn phân khúc, huấn luyện và đánh giá mô hình phân loại.

\---

## 0\. Thống kê tóm tắt theo giai đoạn

|Giai đoạn|Tập dữ liệu|Số mẫu|Số cột|Ghi chú|
|-|-|-:|-:|-|
|Sau NB1|`data\_bds\_NaNFill.csv`|4.719|15|Dữ liệu thô sau crawl|
|Sau dedup NB2|—|3.520|15|Loại 1.082 bản ghi trùng (\~22.93%) + 1 bản ghi rỗng|
|Sau cleaning NB2|`Cleaned\_data.csv`|3.512|18|Checkpoint trung gian|
|Sau Qwen NB3|`Cleaned\_data\_qwen.csv`|3.628|23|Thêm 5 đặc trưng POI từ Qwen|
|Model-ready train|`X\_train\_model.csv`|2.901|20|80% tập huấn luyện|
|Model-ready test|`X\_test\_model.csv`|727|20|20% tập kiểm thử|

\---

## 1\. Luồng dữ liệu chính

|Giai đoạn|File đầu vào chính|File đầu ra chính|Vai trò|
|-|-|-|-|
|NB1 — Crawl|`hrefs.txt` hoặc trang listing|`data\_bds.csv`, `data\_bds\_NaNFill.csv`|Thu thập thông tin rao bán và bổ sung một số thuộc tính từ trường `Mô tả`.|
|NB2 — Cleaning|`data\_bds\*.csv`|`Cleaned\_data.csv`, `X\_train\_cleaned.csv`, `X\_test\_cleaned.csv`, `y\_train\_cleaned.csv`, `y\_test\_cleaned.csv`|Chuẩn hóa dữ liệu, loại trùng, xử lý thiếu, xử lý ngoại lai.|
|NB3 — EDA + FE + Qwen|`Cleaned\_data.csv`, `Cleaned\_data\_qwen.csv`|`X\_train\_processed.csv`, `X\_test\_processed.csv`, `y\_train\_processed.csv`, `y\_test\_processed.csv`|Tạo thêm đặc trưng Qwen, gộp đặc trưng, OHE, ordinal encoding, target encoding.|
|NB4 — Modeling|`X\_train\_processed.csv`, `X\_test\_processed.csv`, `y\_train\_processed.csv`, `y\_test\_processed.csv`|`best\_pipeline\_XGBoost.pkl`, `cv\_results.csv`, `X\_train\_model.csv`, `X\_test\_model.csv`, `y\_train\_encoded.csv`, `y\_test\_encoded.csv`|Tạo nhãn 5 lớp, huấn luyện mô hình, đánh giá và lưu pipeline tốt nhất.|

\---

## 2\. Cột dữ liệu thô sau NB1

Tổng số mẫu: **4.719** bản ghi · **15** cột.

|Tên cột|Kiểu dữ liệu|Null (n)|Null (%)|Miền giá trị / ví dụ|Ý nghĩa|
|-|-|-:|-:|-|-|
|`Link`|string|0|0.00|URL bài đăng|Đường dẫn gốc của tin rao. Dùng để kiểm tra trùng hoặc truy vết dữ liệu.|
|`Địa chỉ`|string|183|3.88|`Đường Nguyễn Thượng Hiền, P.6, Q.Bình Thạnh`|Địa chỉ dạng văn bản lấy từ trang chi tiết.|
|`Khoảng giá`|string|183|3.88|`16,9 tỷ`, `950 triệu`, `thỏa thuận`|Giá rao bán dạng thô, chưa chuẩn hóa đơn vị.|
|`Diện tích`|string|183|3.88|`90 m²`, `48 m²`|Diện tích bất động sản dạng thô.|
|`Số phòng ngủ`|string|1.101|23.33|`3 phòng`, `11 phòng`, null|Số phòng ngủ dạng thô.|
|`Số phòng tắm, vệ sinh`|string|1.254|26.57|`2 phòng`, `12 phòng`, null|Số phòng tắm / vệ sinh dạng thô.|
|`Số tầng`|string|762|16.15|`2 tầng`, `3 tầng`, null|Số tầng dạng thô.|
|`Hướng nhà`|string|3.649|**77.33**|`Đông - Nam`, `Tây`, null|Hướng nhà. **Bị loại ở NB2** do tỷ lệ thiếu quá cao.|
|`Hướng ban công`|string|4.040|**85.61**|`Đông`, `Nam`, null|Hướng ban công. **Bị loại ở NB2** do tỷ lệ thiếu quá cao.|
|`Đường vào`|string|1.893|40.11|`6 m`, `8 m`, null|Độ rộng đường/hẻm vào nhà dạng thô.|
|`Pháp lý`|string|300|6.36|`Sổ đỏ/ Sổ hồng`, `Đang chờ sổ`, null|Tình trạng pháp lý dạng thô.|
|`Nội thất`|string|2.034|43.10|`Đầy đủ`, `Cơ bản`, null|Tình trạng nội thất dạng thô.|
|`Latitude`|float|183|3.88|`10.808340`|Vĩ độ lấy từ iframe bản đồ Google Maps nhúng trong trang chi tiết.|
|`Longitude`|float|183|3.88|`106.684205`|Kinh độ lấy từ iframe bản đồ Google Maps nhúng trong trang chi tiết.|
|`Mô tả`|string|183|3.88|đoạn văn bản tự do|Nội dung mô tả của tin rao; dùng để trích xuất đặc trưng bổ sung.|

> \*\*Ghi chú ngưỡng thiếu:\*\*
> - `< 5%` (xanh lá): thiếu thấp, an toàn sử dụng trực tiếp.
> - `5–30%` (xanh dương): thiếu trung bình, cần chiến lược điền khuyết phù hợp.
> - `> 30%` (cam/đỏ): thiếu cao, cân nhắc loại hoặc tạo cờ MNAR.

\---

## 3\. Cột sau làm sạch NB2

Tổng số mẫu sau dedup: **3.512** bản ghi · **18** cột (loại 2 cột, thêm 5 cột mới).

|Tên cột|Kiểu sau xử lý|Miền giá trị / quy ước|Xử lý thiếu|Ý nghĩa|
|-|-|-|-|-|
|`Link`|string|URL|—|Giữ để truy vết nguồn; không dùng cho mô hình.|
|`Địa chỉ`|string|địa chỉ đầy đủ|—|Dùng để phân rã thành `Đường`, `Phường/Xã`, `Quận`.|
|`Khoảng giá`|float|tỷ VNĐ; ví dụ `5.95`, `16.9`|Loại bản ghi thiếu|Giá rao bán sau chuẩn hóa đơn vị. Dùng tạo nhãn ở NB4; loại khỏi đặc trưng mô hình.|
|`Diện tích`|float|m²; ví dụ `48.0`, `90.0`|Loại bản ghi thiếu|Diện tích bất động sản.|
|`Số phòng ngủ`|integer|≥ 1; outlier lớn được kiểm tra|MICE + RandomForest|Số phòng ngủ sau chuẩn hóa và điền khuyết.|
|`Số phòng tắm, vệ sinh`|integer|≥ 1|KNN Imputer|Số phòng tắm / vệ sinh sau chuẩn hóa và điền khuyết.|
|`Số tầng`|integer|≥ 1; outlier quá lớn bị loại|MissForest / IterativeImputer + RF|Số tầng của bất động sản.|
|`Đường vào`|float|mét; outlier `> 50m` bị loại|**Giữ NaN có chủ đích** (MNAR)|Độ rộng đường/hẻm. Thiếu phản ánh thông tin nghiệp vụ.|
|`Pháp lý`|category|`Sổ hồng`, `Sổ đỏ`, `không có`|Gán `"không có"`|Pháp lý sau chuẩn hóa nhóm giá trị.|
|`Nội thất`|category|`Đầy đủ`, `Cơ bản`, `Trống / Nhà thô`, `không có`|Gán `"Trống / Nhà thô"`|Nội thất sau chuẩn hóa nhóm giá trị.|
|`Latitude`|float|\~10.4–10.9 (TP.HCM)|Loại bản ghi thiếu|Vĩ độ; có kiểm tra điểm bất thường.|
|`Longitude`|float|\~106.4–107.0 (TP.HCM)|Loại bản ghi thiếu|Kinh độ; có kiểm tra điểm bất thường.|
|`Mô tả`|string|văn bản tự do|—|Dùng cho EDA và trích xuất đặc trưng bằng Qwen.|
|`Loại đường vào`|category|`Mặt tiền`, `Hẻm ô tô`, `Hẻm xe máy`, `Chưa xác định`|—|Loại tiếp cận được trích từ mô tả và/hoặc độ rộng đường.|
|`Đường`|string|ví dụ `Nguyễn Thượng Hiền`, null|—|Tên đường phân rã từ `Địa chỉ`.|
|`Phường/Xã`|string|ví dụ `Tân Quy`, `Long Bình`|—|Phường/xã phân rã từ `Địa chỉ`.|
|`Quận`|string/category|ví dụ `Bình Thạnh`, `7`, `Thủ Đức`|—|Quận/huyện thuộc TP.HCM; dùng target encoding ở NB3.|
|`loai\_duong\_null\_flag`|integer|`0`, `1`|—|Cờ báo `Loại đường vào` ban đầu bị thiếu.|
|`duong\_vao\_null\_flag`|integer|`0`, `1`|—|Cờ báo `Đường vào` bị thiếu (MNAR — thiếu có thể phản ánh hành vi giấu thông tin của người đăng).|
|`Loại\_Dữ\_Liệu`|string|`Train`, `Test`|—|Cột đánh dấu phân hoạch train/test; dùng để khôi phục split ở NB3/NB4.|

**Cột bị loại trong NB2:**

|Cột|Null (%)|Lý do loại|
|-|-:|-|
|`Hướng nhà`|77.33|Tỷ lệ thiếu quá cao, không ổn định cho pipeline mô hình hóa|
|`Hướng ban công`|85.61|Tỷ lệ thiếu quá cao, không ổn định cho pipeline mô hình hóa|

\---

## 4\. Cột tăng cường bằng Qwen trong NB3

Các cột được trích xuất từ trường `Mô tả` bằng mô hình Qwen2.5-7B-Instruct chạy trên Kaggle T4 GPU. Đầu ra là JSON nhị phân; trường thiếu hoặc không parse được gán `0`.

|Tên cột|Kiểu|Miền giá trị|Ý nghĩa|
|-|-|:-:|-|
|`hem\_xe\_hoi`|integer/binary|`0`, `1`|Mô tả có dấu hiệu hẻm xe hơi / ô tô có thể vào nhà.|
|`gan\_cho\_sieu\_thi`|integer/binary|`0`, `1`|Bất động sản gần chợ, siêu thị hoặc tiện ích mua sắm.|
|`gan\_truong\_hoc`|integer/binary|`0`, `1`|Bất động sản gần trường học.|
|`gan\_benh\_vien`|integer/binary|`0`, `1`|Bất động sản gần bệnh viện / cơ sở y tế.|
|`gan\_cong\_vien\_ho\_nuoc`|integer/binary|`0`, `1`|Bất động sản gần công viên, hồ nước hoặc không gian xanh.|

> \*\*Reproducibility:\*\* Xem `llm-qwen-extract-bds.ipynb` để biết prompt template, schema JSON đầu ra và quy tắc hậu xử lý.

\---

## 5\. Cột sau Feature Engineering NB3

|Tên cột|Kiểu|Miền giá trị / quy ước|Ý nghĩa|
|-|-|-|-|
|`Tổng số phòng`|integer/float|`Số phòng ngủ + Số phòng tắm`|Đặc trưng gộp nhằm giảm đa cộng tuyến. Hai cột gốc bị loại sau bước này.|
|`Pháp lý\_Sổ đỏ`|float/binary|`0.0`, `1.0`|One-hot encoding `Pháp lý`; `drop='first'` → một nhóm làm mốc bị ẩn.|
|`Pháp lý\_không có`|float/binary|`0.0`, `1.0`|OHE nhóm pháp lý không có/không rõ.|
|`Nội thất\_Cơ bản`|float/binary|`0.0`, `1.0`|OHE nhóm nội thất cơ bản.|
|`Nội thất\_Trống / Nhà thô`|float/binary|`0.0`, `1.0`|OHE nhóm nhà thô/trống.|
|`Nội thất\_Đầy đủ`|float/binary|`0.0`, `1.0`|OHE nhóm nội thất đầy đủ.|
|`Loại đường vào`|float/ordinal|`0` = Chưa xác định; `1` = Hẻm xe máy; `2` = Hẻm ô tô; `3` = Mặt tiền; `-1` = unknown khi transform|Mã hóa thứ bậc theo mức độ tiếp cận.|
|`Quận\_encoded`|float|giá trị liên tục (smoothed mean)|Smoothed target encoding: `(n\_q × mean\_q + m × global\_mean) / (n\_q + m)`, `m = 10`. Chỉ fit trên tập train.|
|`Đường vào`|float|mét hoặc `NaN`|Giữ nguyên NaN; pipeline mô hình downstream xử lý bằng median imputer.|
|`Đường`|string|tên đường hoặc null|Còn trong `X\_train\_processed.csv`; bị loại ở NB4 khi chọn cột số.|
|`Phường/Xã`|string|tên phường/xã|Còn trong `X\_train\_processed.csv`; bị loại ở NB4 khi chọn cột số.|

\---

## 6\. Tập đặc trưng mô hình cuối trong NB4

NB4 chỉ giữ **20 đặc trưng số** sau khi loại leakage và cột phi số:

|#|Tên đặc trưng|Kiểu|Nhóm|Ý nghĩa|
|-:|-|-|-|-|
|1|`Diện tích`|float|Vật lý|Diện tích nhà/đất, đơn vị m².|
|2|`Số tầng`|float|Vật lý|Số tầng sau điền khuyết.|
|3|`Đường vào`|float|Hạ tầng|Độ rộng đường vào (m); NaN còn lại được impute median trong pipeline.|
|4|`Loại đường vào`|ordinal float|Hạ tầng|0–3 theo mức độ tiếp cận.|
|5|`Latitude`|float|Địa lý|Vĩ độ WGS84.|
|6|`Longitude`|float|Địa lý|Kinh độ WGS84.|
|7|`loai\_duong\_null\_flag`|binary|Null flag|Cờ thiếu loại đường vào.|
|8|`duong\_vao\_null\_flag`|binary|Null flag|Cờ thiếu độ rộng đường vào (MNAR).|
|9|`hem\_xe\_hoi`|binary|POI (Qwen)|Hẻm xe hơi / ô tô tiếp cận được.|
|10|`gan\_cho\_sieu\_thi`|binary|POI (Qwen)|Gần chợ/siêu thị.|
|11|`gan\_truong\_hoc`|binary|POI (Qwen)|Gần trường học.|
|12|`gan\_benh\_vien`|binary|POI (Qwen)|Gần bệnh viện.|
|13|`gan\_cong\_vien\_ho\_nuoc`|binary|POI (Qwen)|Gần công viên/hồ nước.|
|14|`Tổng số phòng`|float|Tổng hợp|Phòng ngủ + phòng tắm.|
|15|`Pháp lý\_Sổ đỏ`|binary|Pháp lý (OHE)|Có sổ đỏ/sổ hồng.|
|16|`Pháp lý\_không có`|binary|Pháp lý (OHE)|Không có giấy tờ pháp lý.|
|17|`Nội thất\_Cơ bản`|binary|Nội thất (OHE)|Nội thất cơ bản.|
|18|`Nội thất\_Trống / Nhà thô`|binary|Nội thất (OHE)|Nhà trống/thô.|
|19|`Nội thất\_Đầy đủ`|binary|Nội thất (OHE)|Nội thất đầy đủ.|
|20|`Quận\_encoded`|float|Vị trí|Smoothed target encoding của quận.|

**Danh sách cột leakage bị loại trước khi huấn luyện:**

```
Khoảng giá, Giá, Giá/m2, Giá/m², Gia\_m2, Giá trên m2,
price\_per\_m2, price\_per\_m², Khoảng giá\_log, Giá\_log,
Giá/m2\_log, Giá\_trên\_m2, Log\_Khoảng\_giá
```

\---

## 7\. Biến mục tiêu và nhãn phân loại

### 7.1. Biến mục tiêu gốc

|Tên biến|Kiểu|Đơn vị|Ý nghĩa|
|-|-|-|-|
|`Khoảng giá`|float|tỷ VNĐ|Giá rao bán sau chuẩn hóa. Dùng tạo nhãn; không đưa vào đặc trưng mô hình.|

### 7.2. Công thức đơn giá/m²

```
gia\_m2 = (Khoảng giá × 1000) / Diện tích
```

* `Khoảng giá`: đơn vị tỷ VNĐ → nhân 1000 để đổi sang triệu VNĐ.
* `Diện tích`: đơn vị m².
* Kết quả `gia\_m2`: đơn vị **triệu VNĐ/m²**.

### 7.3. Cách tạo nhãn 5 lớp (KMeans)

1. Tính `gia\_m2` trên tập train.
2. Loại 1% ngoại lai phía trên.
3. Chạy `KMeans(n\_clusters=5, random\_state=42, n\_init=10)` trên `gia\_m2` của train.
4. Sắp xếp các cụm theo tâm cụm tăng dần.
5. Áp dụng cụm lên tập test bằng `predict`.

|Mã nhãn|Tên nhãn|Số mẫu train|Số mẫu test|
|:-:|-|-:|-:|
|`0`|Bình dân|1.127 (38.8%)|299 (41.1%)|
|`1`|Trung cấp|997 (34.4%)|225 (30.9%)|
|`2`|Cao cấp|488 (16.8%)|142 (19.5%)|
|`3`|Hạng sang|201 (6.9%)|40 (5.5%)|
|`4`|Siêu sang|88 (3.0%)|21 (2.9%)|

> \*\*Tỷ lệ mất cân bằng\*\* ρ (Bình dân : Siêu sang) ≈ 12.8 : 1.

> ⚠️ \*\*Reproducibility:\*\* Ngưỡng phân khúc không cố định — được sinh lại từ KMeans mỗi lần chạy. Để tái lập chính xác, giữ nguyên `random\_state=42`, dữ liệu train và lưu biến `BREAKS` sau khi tạo nhãn.

\---

## 8\. Output mô hình

|File|Nội dung|
|-|-|
|`best\_pipeline\_XGBoost.pkl`|Pipeline mô hình tốt nhất theo Macro F1 CV.|
|`cv\_results.csv`|Bảng so sánh 5 mô hình theo Stratified 5-Fold CV.|
|`X\_train\_model.csv`|20 đặc trưng số dùng để huấn luyện.|
|`X\_test\_model.csv`|20 đặc trưng số dùng để kiểm thử.|
|`y\_train\_encoded.csv`|Nhãn train đã mã hóa 0–4.|
|`y\_test\_encoded.csv`|Nhãn test đã mã hóa 0–4.|
|`class\_distribution\_grouped.png`|Biểu đồ phân phối nhãn train/test.|
|`cv\_macro\_f1\_boxplot.png`|Boxplot Macro F1 qua 5 fold.|
|`confusion\_matrix.png`|Ma trận nhầm lẫn trên test set.|
|`feature\_importance.png`|Biểu đồ tầm quan trọng đặc trưng (top 15).|

**Kết quả tốt nhất (XGBoost):**

|Metric|Giá trị|
|-|-|
|CV Macro F1|0.6107 ± 0.0265|
|Test Macro F1|0.6322|
|Test Weighted F1|0.6942|
|Test Accuracy|0.6919|
|\|CV − Test\||0.0215|
|Top 3 features|`Quận\_encoded`, `hem\_xe\_hoi`, `Số tầng`|

\---

## 9\. Giới hạn và lưu ý sử dụng

1. **Giá rao bán, không phải giá giao dịch.** Dữ liệu phản ánh kỳ vọng của người bán; giá thực tế có thể thấp hơn và mức chênh lệch khác nhau giữa các phân khúc.
2. **Nhãn phân khúc theo dữ liệu thực nghiệm.** Nhãn được tạo bằng KMeans, không tương đương ngưỡng phân khúc cố định của thị trường (CBRE hoặc tương tự). Không nên dùng cho định giá pháp lý.
3. **Target encoding trước cross-validation.** `Quận\_encoded` được tính trên toàn bộ tập train trước khi chạy CV, không được đưa vào trong từng fold. Kết quả CV có thể lạc quan nhẹ.
4. **Phạm vi địa lý và thời gian.** Dataset chỉ phản ánh thị trường **TP.HCM tháng 5/2026** trên batdongsan.com.vn. Không suy rộng sang thành phố khác, giai đoạn khác, hoặc thị trường giao dịch thực tế mà không kiểm định lại.
5. **Tọa độ từ bản đồ nhúng.** Latitude/Longitude lấy từ iframe Google Maps trong trang chi tiết; có thể sai lệch so với vị trí thực tế.
6. **Qwen extraction không tất định.** Bước trích xuất đặc trưng bằng Qwen chạy trên Kaggle T4 GPU; kết quả có thể khác nhẹ nếu chạy lại trên môi trường khác. Xem `llm-qwen-extract-bds.ipynb` để biết chi tiết prompt và post-processing.

\---

*CODEBOOK v2.0 · Pipeline phân loại BĐS TP.HCM · DS108 · 2026*

