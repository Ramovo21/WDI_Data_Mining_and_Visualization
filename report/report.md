# REPORT PLAN — Đồ án Data Mining & Visualization (WDI 2000–2025)

## 1) Mục đích tài liệu này

Tài liệu này là **đặc tả triển khai báo cáo cuối cùng trên 1 notebook duy nhất: `report.ipynb`**.

- Bám sát yêu cầu Lab 2 theo hướng nhiều giai đoạn.
- Hiện tại chỉ **tổng hợp đầy đủ Giai đoạn 1 (Phân tích cơ bản dữ liệu)**.
- Các giai đoạn tiếp theo được mô tả dưới dạng **khung triển khai chi tiết theo cell** để bạn tiếp tục phát triển đồng nhất.

---

## 2) Ràng buộc kỹ thuật xuyên suốt đồ án

## 2.1. Thư viện được phép dùng (chỉ 4 thư viện)

```python
import numpy as np
import pandas as pd
import seaborn as sns
import matplotlib.pyplot as plt
```

**Không dùng thêm thư viện ngoài danh sách trên** trong toàn bộ `report.ipynb`.

## 2.2. Quy ước notebook bắt buộc

- **Markdown cell**: mô tả bài toán, giải thích dữ liệu, nhận xét kết quả.
- **Code cell (Python)**: xử lý dữ liệu + trực quan hóa.
- **Kết quả/biểu đồ hiển thị trực tiếp** ngay dưới cell code.
- Tên biến, luồng xử lý và format cột giữ nhất quán với pipeline trong `02_data_preprocessing.ipynb`.

## 2.3. Nguồn dữ liệu và phạm vi

- File chính: `Data/Dataset.csv`
- File tham chiếu series: `Data/Series-Metadata.csv`
- Thời gian: **2000–2025** (26 cột năm)
- 10 chỉ tiêu mục tiêu (Series Code) theo `spec.md`:
  - `NY.GDP.PCAP.KD`
  - `NY.GDP.MKTP.KD.ZG`
  - `EN.GHG.CO2.PC.CE.AR5`
  - `EG.USE.PCAP.KG.OE`
  - `EG.ELC.ACCS.ZS`
  - `EG.CFT.ACCS.ZS`
  - `SP.POP.TOTL`
  - `AG.LND.FRST.ZS`
  - `EG.FEC.RNEW.ZS`
  - `NV.IND.TOTL.ZS`

---

## 3) Tổng quan cấu trúc `report.ipynb` (toàn đồ án)

## Giai đoạn 0 — Mở đầu và cấu hình
- Giới thiệu đề tài, câu hỏi nghiên cứu, phạm vi dữ liệu.
- Import 4 thư viện cho phép.
- Khai báo hằng số: đường dẫn dữ liệu, danh sách 10 series, khoảng năm.

## Giai đoạn 1 — Phân tích cơ bản về dữ liệu (**đã hoàn thành, tổng hợp chi tiết bên dưới**)
- Giới thiệu dataset.
- Mô tả cấu trúc, số bản ghi, số trường.
- Thống kê mô tả cơ bản, giá trị thiếu.

## Giai đoạn 2 — Tiền xử lý dữ liệu
- Làm sạch dữ liệu: bỏ dòng rác, chuẩn hóa thiếu dữ liệu `".." -> NaN`.
- Lọc 10 series, lọc năm 2000–2025.
- Quy tắc phạm vi năm: nếu tỷ lệ thiếu của 2024 và 2025 ở mức quá cao (thực tế hiện tại 2024 = 54.45%, 2025 = 100%) thì loại hẳn 2 năm này khỏi tập phân tích.
- Scope phân tích chính sau tiền xử lý được chốt lại: **2000–2023**.
- Chuyển `wide -> long -> tidy`.
- Kiểm tra schema và lưu file `processed_long.csv`, `processed_tidy.csv`.

## Giai đoạn 3 — Phân tích theo mục tiêu đồ án (Kuznets + chuyển dịch năng lượng)
- Phân tích quan hệ GDP/CO2, tăng trưởng, năng lượng, rừng theo thời gian.
- So sánh nhóm quốc gia / khu vực (khi có mapping Region, IncomeGroup).
- Trực quan bằng line, bar, scatter, bubble, boxplot, heatmap.

## Giai đoạn 4 — Tổng hợp insight & kết luận
- Tổng hợp phát hiện chính theo từng mục tiêu.
- Trả lời câu hỏi nghiên cứu ban đầu.
- Nêu giới hạn dữ liệu và hướng mở rộng.

---

## 4) Giai đoạn 1 — Tổng hợp đã làm (Basic Data Analysis)

## 4.1. Mô tả dataset (theo `spec.md` + notebook 01)

- Nguồn: World Development Indicators (World Bank).
- Mỗi dòng thô: 1 cặp `(Country/Entity, Series)`.
- Cột định danh: `Country Name`, `Country Code`, `Series Name`, `Series Code`.
- Cột giá trị năm: từ `2000 [YR2000]` đến `2025 [YR2025]`.
- Missing value trong dữ liệu thô biểu diễn bằng chuỗi `".."`.

## 4.2. Các con số cấu trúc dữ liệu (đã kiểm tra lại từ file CSV)

- **Tổng số dòng trong file**: `2665`
- **Số bản ghi hợp lệ (Country Code dài 3 ký tự)**: `2660`
- **Số cột (trường dữ liệu)**: `30`
  - 4 cột định danh
  - 26 cột năm
- **Số country/entity code phân biệt**: `266`
- **Số series code phân biệt**: `10`
- **Khoảng năm**: `2000 [YR2000]` → `2025 [YR2025]`

## 4.3. Thống kê thiếu dữ liệu (điểm nổi bật)

- Giai đoạn đầu (2000–2004): tỷ lệ thiếu khoảng `6.72%` đến `8.63%`.
- 2022 tăng thiếu rõ rệt: `13.88%`.
- 2023 thiếu cao: `22.81%`.
- 2024 thiếu rất cao: `54.45%`.
- 2025 thiếu `100%` (chưa có dữ liệu thực).

**Nhận xét GĐ1:** dữ liệu đủ tốt cho phân tích lịch sử, nhưng cần xử lý thiếu dữ liệu cẩn thận ở các năm mới. Với mức thiếu của 2024 và 2025 quá lớn, báo cáo sẽ chuyển scope phân tích thực tế sang **2000–2023**.

---

## 5) “Shell cực kỳ chi tiết” cho `report.ipynb` — PHẦN GIAI ĐOẠN 1

> Mục tiêu phần này: bạn có thể copy đúng trình tự cell để viết lại GĐ1 trong `report.ipynb` mà không lệch cấu trúc.

### Cell 1 (Markdown) — Tiêu đề notebook
- Tên đồ án.
- Mục tiêu notebook.
- Danh sách phần sẽ thực hiện trong GĐ1.

### Cell 2 (Markdown) — Mục lục ngắn
- 1. Thiết lập môi trường
- 2. Giới thiệu dataset
- 3. Cấu trúc dữ liệu
- 4. Thống kê mô tả
- 5. Kết luận giai đoạn

### Cell 3 (Code) — Import thư viện được phép
```python
import numpy as np
import pandas as pd
import seaborn as sns
import matplotlib.pyplot as plt
```

### Cell 4 (Code) — Khai báo đường dẫn dữ liệu
```python
DATASET_PATH = "Data/Dataset.csv"
SERIES_METADATA_PATH = "Data/Series-Metadata.csv"
```

### Cell 5 (Code) — Đọc dữ liệu thô
```python
df = pd.read_csv(DATASET_PATH, encoding="utf-8")
# nếu lỗi encoding thì fallback lần lượt: utf-8-sig, cp1252, latin-1
```

### Cell 6 (Markdown) — Giới thiệu nguồn dữ liệu
- WDI, ý nghĩa đề tài, phạm vi 2000–2025.

### Cell 7 (Code) — In các Series có trong dữ liệu
```python
series_info = df[["Series Name", "Series Code"]].drop_duplicates().sort_values("Series Code")
series_info
```

### Cell 8 (Markdown) — Mô tả cấu trúc bảng
- wide format, ý nghĩa 4 cột id, 26 cột năm.

### Cell 9 (Code) — Đếm số cột, số dòng, xác định cột năm
```python
id_cols = ["Country Name", "Country Code", "Series Name", "Series Code"]
year_cols = [c for c in df.columns if c not in id_cols]

n_rows_raw = len(df)
n_cols = len(df.columns)

valid_mask = df["Country Code"].notna() & (df["Country Code"].astype(str).str.len() == 3)
df_valid = df[valid_mask].copy()
n_rows_valid = len(df_valid)
```

### Cell 10 (Code) — Hiển thị head/tail + dtype
```python
display(df.head())
print(df.dtypes)
display(df.tail(10))
```

### Cell 11 (Code) — Thống kê số country, số series
```python
n_countries = df_valid["Country Code"].nunique()
n_series = df_valid["Series Code"].nunique()
print(n_countries, n_series)
```

### Cell 12 (Markdown) — Chuyển dữ liệu năm sang numeric để thống kê
- Nêu rõ `".."` sẽ thành `NaN` bằng `pd.to_numeric(..., errors="coerce")`.

### Cell 13 (Code) — Describe các cột năm
```python
df_numeric = df_valid.copy()
for c in year_cols:
    df_numeric[c] = pd.to_numeric(df_numeric[c], errors="coerce")

display(df_numeric[year_cols].describe())
```

### Cell 14 (Code) — Missing theo năm
```python
missing_per_year = df_numeric[year_cols].isna().sum()
missing_rate = (missing_per_year / len(df_numeric) * 100).round(2)
missing_df = pd.DataFrame({
    "year": [c[:4] for c in year_cols],
    "missing_count": missing_per_year.values,
    "missing_pct": missing_rate.values
})
display(missing_df)
```

### Cell 15 (Code) — Missing/valid theo series
```python
stats_by_series = df_numeric.groupby("Series Code")[year_cols].agg(["count", "mean", "min", "max"])
stats_by_series["valid_count"] = df_numeric.groupby("Series Code")[year_cols].apply(lambda x: x.notna().sum().sum())
stats_by_series["missing_count"] = df_numeric.groupby("Series Code")[year_cols].apply(lambda x: x.isna().sum().sum())
display(stats_by_series)
```

### Cell 16 (Code) — Phân bố bản ghi theo series/quốc gia
```python
record_per_series = df_valid.groupby(["Series Code", "Series Name"]).size().reset_index(name="record_count")
display(record_per_series)

record_per_country = df_valid.groupby("Country Name").size().sort_values(ascending=False).head(15)
print(record_per_country)
```

### Cell 17 (Code) — Mean theo một số năm mẫu
```python
sample_years = [c for c in year_cols if c[:4] in ["2000", "2010", "2020"]]
summary_by_series = df_numeric.groupby("Series Name")[sample_years].mean()
display(summary_by_series.round(4))
```

### Cell 18 (Markdown) — Kết luận Giai đoạn 1
- Dataset wide, có missing tăng mạnh ở năm mới.
- Dữ liệu đủ cho phân tích sau khi tiền xử lý.
- Chuyển sang Giai đoạn 2: clean + reshape dữ liệu.

---

## 6) Khung “shell chi tiết” cho các giai đoạn tiếp theo (để mở rộng sau)

## 6.1. Giai đoạn 2 (Preprocessing)
- Cụm cell A: đọc `df_raw`, kiểm tra cuối file.
- Cụm cell B: xóa dòng rác, chuẩn hóa missing, ép kiểu số.
- Cụm cell C: lọc 10 series + lọc năm; nếu 2024 và 2025 thiếu vượt ngưỡng chấp nhận thì loại 2 năm này và chốt phân tích 2000–2023.
- Cụm cell D: `pd.melt` -> `df_long`.
- Cụm cell E: `pivot` -> `df_tidy` + kiểm tra schema contract.
- Cụm cell F: lưu `Data/processed_long.csv`, `Data/processed_tidy.csv`.

## 6.2. Giai đoạn 3 (Phân tích chính của đồ án)
- Cell nhóm GDP–CO2 theo thời gian.
- Cell nhóm Kuznets scatter.
- Cell nhóm năng lượng tái tạo theo nhóm quốc gia.
- Cell nhóm điện/nhiên liệu sạch và tăng trưởng.
- Cell nhóm rừng và tăng trưởng.

## 6.3. Viễn cảnh triển khai chi tiết cho giai đoạn phân tích mục tiêu (GĐ3)

Phần này cụ thể hóa 3 mục tiêu phân tích để triển khai trực tiếp trong `report.ipynb`, bảo đảm mỗi trực quan hóa đều trả lời một câu hỏi nghiên cứu cụ thể, có cơ sở chọn biến rõ ràng và có tiêu chí diễn giải nhất quán.

### 6.3.1. Mục tiêu 1 — Kiểm chứng “Đường cong Kuznets”

**Câu hỏi phân tích**
- Có phải quốc gia càng giàu thì phát thải càng tăng mãi, hay tồn tại một ngưỡng thu nhập mà sau đó phát thải bình quân đầu người giảm dần (dạng chữ U ngược)?

**Xác định mục tiêu và lựa chọn trường dữ liệu**
- Trục X: `NY.GDP.PCAP.KD` (GDP per capita, constant 2015 US$).
- Trục Y: `EN.GHG.CO2.PC.CE.AR5` (CO2 emissions per capita).
- Trọng số bong bóng: `SP.POP.TOTL` (Population, total).
- Trục thời gian: `Year` trong scope chính 2000–2023.
- Biểu đồ chính: Bubble Scatter Plot + Polynomial Regression Line (bậc 2).

**Ý nghĩa metrics và lý do phù hợp với mục tiêu**
- GDP bình quân đầu người đại diện mức phát triển kinh tế theo đầu người, giảm nhiễu do quy mô dân số quốc gia.
- CO2 bình quân đầu người phản ánh cường độ phát thải bình quân, cho phép so sánh công bằng giữa nước lớn và nhỏ.
- Dân số tổng dùng làm trọng số để biểu đồ không “đánh đồng” tác động của quốc gia rất nhỏ với quốc gia rất lớn.
- Bộ biến này đúng trọng tâm Kuznets vì trực tiếp kiểm định mối quan hệ giữa mức phát triển và áp lực môi trường theo đầu người.

**Mối quan hệ giữa các trường dữ liệu và lựa chọn trực quan**
- Quan hệ kỳ vọng là phi tuyến bậc 2 (tăng rồi giảm), nên dùng Bubble Scatter + đường hồi quy đa thức để kiểm tra dạng cong.
- Cần kiểm tra ảnh hưởng điểm ngoại lai GDP rất cao hoặc CO2 rất cao để tránh bóp méo đường cong tổng thể.
- Khi cần, có thể kiểm tra độ nhạy bằng cách hiển thị thêm trục GDP dạng log để giảm hiện tượng dồn cụm ở vùng thu nhập thấp.

### 6.3.2. Mục tiêu 2 — Phân tích “Decoupling” tăng trưởng và phát thải

**Câu hỏi phân tích**
- Quốc gia/nhóm quốc gia có thể vừa duy trì tăng trưởng GDP, vừa tăng năng lượng tái tạo và giảm CO2 hay không?

**Xác định mục tiêu và lựa chọn trường dữ liệu**
- `NY.GDP.MKTP.KD.ZG` (GDP growth, annual %)
- `EG.FEC.RNEW.ZS` (Renewable energy consumption, % tổng tiêu thụ năng lượng cuối cùng)
- `EN.GHG.CO2.PC.CE.AR5` (CO2 emissions per capita)
- Nhóm so sánh chính: `IncomeGroup` gồm `High income` vs `Lower middle income` (để biểu đồ chính rõ và đúng câu chuyện).
- Nhóm kiểm chứng độ bền kết luận: dùng đầy đủ 4 nhóm `High`, `Upper middle`, `Lower middle`, `Low income` khi dữ liệu đủ phủ.
- Trục thời gian: `Year` (chuỗi 20 năm gần nhất trong scope hợp lệ)
- Biểu đồ chính: Dual-axis Line Chart.

**Ý nghĩa metrics và lý do phù hợp với mục tiêu**
- GDP growth cho biết động lực mở rộng kinh tế theo thời gian ngắn hạn.
- Renewable share phản ánh mức chuyển dịch khỏi năng lượng hóa thạch.
- CO2 per capita đo kết quả môi trường đầu ra của quá trình chuyển dịch.
- Bộ ba chỉ số cho phép đánh giá “decoupling”: kinh tế tăng nhưng phát thải không tăng tương ứng, thậm chí giảm.
- Biến `IncomeGroup` giúp kiểm soát khác biệt về mức phát triển, năng lực công nghệ và tốc độ chuyển dịch năng lượng giữa các nhóm nền kinh tế.

**Mối quan hệ giữa các trường dữ liệu và lựa chọn trực quan**
- Dùng Dual-axis Line Chart để quan sát đồng thời xu hướng Renewable (trục trái) và CO2 (trục phải) theo thời gian.
- Đường GDP growth dùng như đường tham chiếu bối cảnh (theo subplot riêng hoặc bảng tóm tắt) để tránh quá tải 3 đại lượng khác đơn vị trên cùng một trục.
- Tín hiệu decoupling mạnh khi Renewable tăng, CO2 giảm và GDP growth duy trì dương trong nhiều năm liên tiếp.
- Nếu kết luận từ 2 nhóm chính không thay đổi khi mở rộng sang 4 nhóm, có thể xem kết luận có độ tin cậy cao hơn.

### 6.3.3. Mục tiêu 3 — Cái giá của công nghiệp hóa với hệ sinh thái và chất lượng sống

**Câu hỏi phân tích**
- Tăng tỷ trọng công nghiệp có đi kèm suy giảm diện tích rừng và chất lượng tiếp cận năng lượng sạch hộ gia đình hay không?

**Xác định mục tiêu và lựa chọn trường dữ liệu**
- `NV.IND.TOTL.ZS` (Industry value added, % of GDP)
- `AG.LND.FRST.ZS` (Forest area, % of land area)
- `EG.CFT.ACCS.ZS` (Access to clean fuels and technologies for cooking)
- Trục thời gian: `Year` (2000–2023)
- Biểu đồ chính: Correlation Heatmap kết hợp Hexbin Plot.

**Ý nghĩa metrics và lý do phù hợp với mục tiêu**
- Industry value added đại diện mức độ công nghiệp hóa trong cấu trúc kinh tế.
- Forest area là chỉ báo tài nguyên sinh thái dễ bị tổn thương trước mở rộng sản xuất.
- Access to clean cooking fuels phản ánh chất lượng sống cơ bản và rủi ro ô nhiễm không khí hộ gia đình.
- Kết hợp ba chỉ số cho phép đọc đồng thời “tăng trưởng cấu trúc” và “chi phí xã hội – môi trường”.

**Mối quan hệ giữa các trường dữ liệu và lựa chọn trực quan**
- Heatmap tương quan Pearson giúp định lượng chiều và độ mạnh liên hệ giữa các biến (đồng biến/nghịch biến).
- Hexbin Plot phù hợp khi điểm dữ liệu dày (nhiều country-year), giúp nhìn mật độ thay vì chỉ điểm rời rạc.
- Cần tách kết luận “tương quan” và “nhân quả”: kết quả chỉ cho thấy xu hướng liên hệ, không khẳng định quan hệ nhân quả trực tiếp.

### 6.3.4. Ma trận lựa chọn biến cho trực quan hóa (tóm tắt triển khai)

| Mục tiêu | Biến chính | Biến hỗ trợ | Dạng quan hệ kỳ vọng | Biểu đồ đề xuất |
|---|---|---|---|---|
| Kuznets | GDP per capita, CO2 per capita | Population (weight) | Phi tuyến chữ U ngược | Bubble Scatter + Polynomial Regression |
| Decoupling | Renewable share, CO2 per capita | GDP growth, IncomeGroup | Tách rời tăng trưởng–phát thải theo thời gian | Dual-axis Line (2 nhóm chính) + kiểm chứng 4 nhóm income |
| Công nghiệp hóa | Industry value added, Forest area | Clean cooking access | Tương quan nghịch/đồng biến đa chiều | Correlation Heatmap + Hexbin |

### 6.3.5. Tiêu chí ra quyết định trước khi vẽ biểu đồ

- Ưu tiên các trường có độ phủ đủ tốt trong 2000–2023; nếu thiếu dữ liệu cao theo nhóm thì ghi rõ mức tin cậy.
- Chuẩn hóa đơn vị và phạm vi so sánh theo đầu người hoặc theo tỷ lệ phần trăm để tránh sai lệch do quy mô quốc gia.
- Kiểm tra logic thời gian: không trộn năm ngoài scope và không so sánh chéo khi mẫu quốc gia giữa các năm thay đổi quá lớn.
- Với biểu đồ có hồi quy hoặc tương quan, luôn kèm nhận xét về ngoại lệ và giới hạn suy luận.
- Với Decoupling: ưu tiên 2 nhóm cho biểu đồ chính để giữ thông điệp; dùng 4 nhóm như bước kiểm chứng độ bền của kết luận khi dữ liệu cho phép.

## 6.4. Giai đoạn 4 (Kết luận)
- Tóm tắt phát hiện theo từng câu hỏi nghiên cứu.
- Nêu hạn chế dữ liệu (đặc biệt missing 2023–2025).
- Đề xuất hướng mở rộng.

---

## 7) Checklist hoàn thành cho bản báo cáo cuối

- [ ] Toàn bộ nội dung chạy trong **1 file** `report.ipynb`
- [ ] Chỉ dùng **NumPy, pandas, seaborn, matplotlib**
- [ ] Mỗi phần có đủ Markdown giải thích + Code + Output trực tiếp
- [ ] Có tách rõ “dữ liệu thô” và “dữ liệu đã tiền xử lý”
- [ ] Kết luận cuối cùng trả lời đúng bài toán Kuznets + chuyển dịch năng lượng

---

## 8) Trạng thái hiện tại

- ✅ Đã tổng hợp xong **Giai đoạn 1** vào tài liệu đặc tả này.
- 🔜 Bước kế tiếp: chuyển nội dung GĐ1 + GĐ2 vào `report.ipynb`, sau đó bổ sung GĐ3, GĐ4 theo đúng khung ở trên.
