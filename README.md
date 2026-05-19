<<<<<<< HEAD
# Data Mining & Visualization — WDI (Kuznets Curve & Energy Transition)

## 1) Mô tả tổng quan

**Chủ đề phân tích:** **Đánh đổi giữa tăng trưởng kinh tế và môi trường** (góc nhìn đường cong Kuznets), đồng thời phân tích **xu hướng chuyển dịch năng lượng toàn cầu**.

Đồ án đi theo chuỗi giá trị:
- Thu thập dữ liệu từ nguồn mở WDI (World Bank).
- Tiền xử lý dữ liệu theo chuẩn pipeline reproducible.
- Chuẩn bị dữ liệu phân tích cho trực quan hóa đa dạng.
- Viết nhận xét/insight có ý nghĩa dựa trên bằng chứng dữ liệu.

## 2) Mục tiêu
- Thu thập dữ liệu thô từ WDI và làm sạch để có dataset phân tích chất lượng.
- Xây dựng pipeline xử lý dữ liệu có thể lặp lại, có kiểm tra chất lượng.
- Trực quan hóa dữ liệu để trả lời câu hỏi nghiên cứu thay vì chỉ mô tả biểu đồ.
- Tạo hệ thống nhận xét có cấu trúc: **Hiện tượng -> Bằng chứng -> Ý nghĩa -> Hạn chế**.

## 3) Thu thập dữ liệu và phạm vi phân tích

### Nguồn dữ liệu
- World Development Indicators (WDI) — World Bank.
- File dữ liệu chính: `Data/Dataset.csv`.
- File tham chiếu series: `Data/Series-Metadata.csv`.

### Phạm vi thời gian
- Dữ liệu thô: 2000–2025.
- Phạm vi phân tích hiệu lực sau kiểm tra chất lượng: **2000–2023**.

### 10 chỉ tiêu mục tiêu
- `NY.GDP.PCAP.KD` (GDP bình quân đầu người)
- `NY.GDP.MKTP.KD.ZG` (Tăng trưởng GDP)
- `EN.GHG.CO2.PC.CE.AR5` (CO2 bình quân đầu người)
- `EG.USE.PCAP.KG.OE` (Mức dùng năng lượng bình quân)
- `EG.ELC.ACCS.ZS` (Tiếp cận điện)
- `EG.CFT.ACCS.ZS` (Tiếp cận nhiên liệu sạch)
- `SP.POP.TOTL` (Dân số)
- `AG.LND.FRST.ZS` (Diện tích rừng)
- `EG.FEC.RNEW.ZS` (Năng lượng tái tạo)
- `NV.IND.TOTL.ZS` (Tỷ trọng công nghiệp)

## 4) Quy trình Data Mining -> Tiền xử lý

### Bước 1 — Khảo sát dữ liệu thô
- Kiểm tra schema, kích thước dữ liệu, số country/entity, số series.
- Kiểm tra dòng metadata/rác ở cuối file export WDI.

### Bước 2 — Làm sạch và chuẩn hóa
- Loại dòng không hợp lệ (Country Code không chuẩn 3 ký tự).
- Chuẩn hóa missing value: `".." -> NaN`.
- Chuyển cột năm về kiểu số để phục vụ thống kê và vẽ biểu đồ.

### Bước 3 — Kiểm soát chất lượng theo năm
- Tính tỷ lệ thiếu dữ liệu theo từng năm.
- Áp dụng rule chất lượng ở phần đuôi chuỗi thời gian:
	- 2024 thiếu rất cao.
	- 2025 thiếu 100%.
- Chốt phạm vi phân tích chính: **2000–2023**.

### Bước 4 — Tái cấu trúc dữ liệu
- `wide -> long`: lưu tại `Data/processed_long.csv`.
- `long -> tidy`: lưu tại `Data/processed_tidy.csv`.

### Bước 5 — Schema contract cho dữ liệu đầu ra
- Đủ cột bắt buộc (`Country Name`, `Country Code`, `Year`, 10 metrics).
- Kiểu dữ liệu hợp lệ (Year dạng số nguyên, metric dạng số).
- Miền năm đúng 2000–2023.
- Khóa logic `(Country Code, Year)` không trùng.

## 5) Kết quả trực quan hóa (đã triển khai trong `report/report.ipynb`)

Phần report đã triển khai đầy đủ theo 3 mục tiêu phân tích, sử dụng dữ liệu đã xử lý (`Data/processed_tidy.csv`, `Data/processed_long.csv`) và phạm vi năm hiệu lực 2000–2023.

### Mục tiêu 1 — Kinh tế và phát thải (Kuznets)
- Biểu đồ đã dùng: **hexbin + bubble scatter + polynomial trend bậc 2** cho GDP per capita vs CO2 per capita; **line chart median theo năm** cho GDP và CO2.
- Kết quả định lượng chính:
	- Trung vị GDP per capita = **5,134 USD**; trung vị CO2 per capita = **2.49**.
	- Tương quan Pearson giữa log10(GDP per capita) và CO2 per capita = **0.462**.
	- GDP per capita trung vị tăng **+82.1%**; CO2 per capita trung vị tăng **+26.4%** (2000 → 2023).

### Mục tiêu 2 — Decoupling tăng trưởng, năng lượng tái tạo và phát thải
- Biểu đồ đã dùng: **scatter** (renewable vs CO2), **scatter có đường tham chiếu GDP growth = 0%**, **line chart theo IncomeGroupProxy**, **boxplot theo nhóm thu nhập proxy**.
- Kết quả định lượng chính:
	- Tương quan Renewable–CO2 = **-0.373**.
	- Tỷ lệ quan sát có GDP growth dương = **85.4%**.
	- Nhóm có renewable trung vị cao nhất và CO2 trung vị thấp nhất đều là **Low** (theo proxy).
	- Năm gần nhất của chuỗi là **2023**, còn thiếu dữ liệu đồng thời cho một số nhóm ở mục 2B.

### Mục tiêu 3 — Công nghiệp hóa, hệ sinh thái và chất lượng sống
- Biểu đồ đã dùng: **hexbin + scatter kiểm tra ngoại lệ**, **line chart median theo năm**, **scatter industry vs clean fuels (giới hạn y 0–100)**, **heatmap tương quan đa biến**.
- Kết quả định lượng chính:
	- Tương quan industry share vs forest area = **-0.025** (rất yếu).
	- Tương quan industry share vs access clean fuels = **0.149**.
	- Cặp tương quan mạnh nhất theo |r|: **renewable_energy_percent vs access_clean_fuels** với **r = -0.786**.

## 6) Framework viết insight có ý nghĩa

Mỗi biểu đồ sẽ được nhận xét theo form thống nhất:

1. **Observation**: xu hướng chính nhìn thấy được từ biểu đồ.
2. **Evidence**: số liệu cụ thể (mốc năm, độ chênh, top/bottom).
3. **Interpretation**: ý nghĩa kinh tế - môi trường của hiện tượng.
4. **Implication**: tác động đối với định hướng chuyển dịch năng lượng.
5. **Limitation**: giới hạn dữ liệu (thiếu dữ liệu, độ trễ cập nhật, thiên lệch tổng hợp).

## 7) Cấu trúc thư mục dự án

```text
Data Mining and Visualization/
├─ Data/
│  ├─ Dataset.csv
│  ├─ Series-Metadata.csv
│  ├─ processed_long.csv
│  └─ processed_tidy.csv
├─ overview/
│  └─ dataset_overview.ipynb
├─ Preprocessing/
│  ├─ data_preprocessing.ipynb
│  └─ data_preprocessing.md
├─ report/
│  ├─ report.ipynb
│  └─ report.md
└─ README.md
```

## 8) Công nghệ sử dụng

### Thư viện phân tích (đúng ràng buộc Lab 2)
- NumPy
- pandas
- seaborn
- matplotlib

### Môi trường
- Sử dụng một Python virtual environment duy nhất: `.venv`.

## 9) Cách chạy nhanh

1. Kích hoạt môi trường:
	 - PowerShell: `.\.venv\Scripts\Activate.ps1`
2. Chạy notebook theo thứ tự:
	 - `overview/dataset_overview.ipynb`
	 - `Preprocessing/data_preprocessing.ipynb`
	 - `report/report.ipynb`
3. Kiểm tra đầu ra trong `Data/`:
	 - `processed_long.csv`
	 - `processed_tidy.csv`

## 10) Kỹ năng học được

- Tư duy end-to-end: từ thu thập dữ liệu đến phân tích trực quan và diễn giải insight.
- Kỹ năng xử lý dữ liệu thực tế: encoding, metadata rows, thiếu dữ liệu, kiểm định schema.
- Thiết kế pipeline có thể tái lập và mở rộng cho các bước modeling/BI tiếp theo.
- Khả năng kết nối dữ liệu với bài toán kinh tế - môi trường mang ý nghĩa thực tiễn.

## 11) Trạng thái hiện tại & bước tiếp theo

- Đã hoàn tất:
	- Khảo sát dữ liệu + tiền xử lý + tạo dataset phân tích.
	- Triển khai đầy đủ hệ thống trực quan hóa cho 3 mục tiêu trong `report/report.ipynb`.
	- Bổ sung insight định lượng dạng markdown dưới từng cụm biểu đồ.
- Bước tiếp theo đề xuất:
	- Rà soát narrative cuối cùng để đồng bộ văn phong báo cáo.
	- Chạy lại notebook theo thứ tự từ đầu để đảm bảo reproducibility trước khi nộp.
	- Xuất bản báo cáo (HTML/PDF) nếu cần nộp bản tĩnh.

