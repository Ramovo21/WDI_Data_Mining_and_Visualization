## Đặc tả tiền xử lý dữ liệu WDI cho đồ án Kuznets (2000–2025)

### 1. Giới thiệu & mục tiêu
- **Bối cảnh**: Đề tài phân tích *“Đánh đổi giữa Tăng trưởng Kinh tế và Môi trường: Phân tích đường cong Kuznets và Sự chuyển dịch Năng lượng toàn cầu (2000–2025)”* dựa trên dữ liệu World Development Indicators (WDI) cho toàn bộ các quốc gia và vùng lãnh thổ.
- **Mục tiêu tiền xử lý**:
  - Chuẩn bị một bộ dữ liệu dạng *tidy* dùng chung cho toàn bộ 12 mục tiêu phân tích và trực quan hóa trong notebook `01_dataset_overview_and_descriptive_analysis.ipynb`.
  - Đảm bảo dữ liệu sạch (loại bỏ dòng rác, chuẩn hóa missing value, giới hạn đúng khoảng thời gian 2000–2025) và có cấu trúc phù hợp cho các phép `groupby`, `pivot`, `corr`, cũng như các biểu đồ Line, Bar, Scatter, Bubble, Boxplot, Heatmap, Radar.
- **File đầu vào**:
  - `Data/Dataset.csv`: file WDI chính, chứa nhiều Series cho tất cả quốc gia, giai đoạn 2000 trở đi.
  - `Data/Series-Metadata.csv`: (cùng cấu trúc với Dataset, được dùng trong notebook để tra cứu mô tả chi tiết Series và kiểm tra encoding ký tự đặc biệt).

- **Đầu ra sau tiền xử lý**:
  - `df_long`: dữ liệu dài, 1 dòng = 1 quốc gia – 1 chỉ tiêu – 1 năm – 1 giá trị.
  - `df_tidy_country_year`: dữ liệu pivot, 1 dòng = 1 quốc gia – 1 năm với **10 cột metrics** đã chọn.
  - `df_tidy_full`: mở rộng `df_tidy_country_year` với thông tin `Region` và `IncomeGroup` (khi có bảng metadata quốc gia).
  - Một số bảng tổng hợp trung gian (`df_global_year`, `df_income_year`, `df_region_year`, v.v.) phục vụ trực tiếp cho 12 mục tiêu trực quan hóa.

---

### 2. Mô tả dữ liệu thô WDI

Trong các cell đầu của notebook, mô tả và kiểm tra cấu trúc dữ liệu như sau:

- **Cột định danh (id vars)**:
  - `Country Name`: tên quốc gia / vùng lãnh thổ (Afghanistan, Albania, Algeria, ...).
  - `Country Code`: mã 3 ký tự theo chuẩn World Bank (AFG, ALB, DZA, ...).
  - `Series Name`: tên đầy đủ của chỉ tiêu (ví dụ: *GDP per capita (constant 2015 US$)*).
  - `Series Code`: mã Series (ví dụ: `NY.GDP.PCAP.KD`), dùng làm khóa ổn định để lọc 10 chỉ tiêu và pivot.

- **Cột thời gian**:
  - Các cột năm có dạng `2000 [YR2000]`, `2001 [YR2001]`, …, kéo dài tới `2025 [YR2025]`.
  - Mỗi ô chứa giá trị số hoặc ký hiệu thiếu dữ liệu `..`.

- **Đặc điểm chất lượng dữ liệu**:
  - **Missing value**: World Bank sử dụng chuỗi `..` để biểu diễn thiếu dữ liệu; cần chuyển về `NaN`.
  - **Dòng metadata / ghi chú**: Cuối file có thể có các dòng không phải quốc gia (ví dụ các đoạn mô tả, chú thích). Ta cần định nghĩa tiêu chí để loại bỏ (xem Bước 2).

Mục tiêu phần này là giúp nhóm hiểu rõ **input schema** trước khi định nghĩa các bước xử lý.

---

### 3. Danh sách 10 chỉ tiêu, ý nghĩa và lý do chọn

#### 3.1. Bảng mapping Series Code
Mười chỉ tiêu sẽ được lọc theo `Series Code`:

- **GDP per capita (constant 2015 US$)** → `NY.GDP.PCAP.KD`
- **GDP growth (annual %)** → `NY.GDP.MKTP.KD.ZG`
- **Carbon dioxide (CO2) emissions excluding LULUCF per capita (t CO2e/capita)** → `EN.GHG.CO2.PC.CE.AR5`
- **Energy use (kg of oil equivalent per capita)** → `EG.USE.PCAP.KG.OE`
- **Access to electricity (% of population)** → `EG.ELC.ACCS.ZS`
- **Access to clean fuels and technologies for cooking (% of population)** → `EG.CFT.ACCS.ZS`
- **Population, total** → `SP.POP.TOTL`
- **Forest area (% of land area)** → `AG.LND.FRST.ZS`
- **Renewable energy consumption (% of total final energy consumption)** → `EG.FEC.RNEW.ZS`
- **Industry (including construction), value added (% of GDP)** → `NV.IND.TOTL.ZS`

#### 3.2. Ý nghĩa và lý do lựa chọn (phục vụ phần thuyết minh)
- **NY.GDP.PCAP.KD – GDP per capita (constant 2015 US$)**  
  - **Ý nghĩa**: Thu nhập bình quân đầu người đã loại bỏ lạm phát, đo lường mức độ giàu có thực tế của cư dân mỗi quốc gia theo thời gian.  
  - **Lý do chọn**: Đây là trục X kinh điển trong phân tích đường cong Kuznets môi trường: giả thuyết cho rằng khi GDP/người tăng đến một ngưỡng, ô nhiễm (CO2/người) sẽ đạt đỉnh rồi giảm.

- **NY.GDP.MKTP.KD.ZG – GDP growth (annual %)**  
  - **Ý nghĩa**: Tốc độ tăng trưởng tổng sản phẩm quốc nội theo giá so sánh, thể hiện nhịp độ phát triển kinh tế hàng năm.  
  - **Lý do chọn**: Dùng để xem **chất lượng tăng trưởng** – liệu tăng trưởng nhanh có đi kèm đánh đổi mạnh về môi trường (CO2, rừng) hay không, và để tìm các “điểm sáng” tăng trưởng bền vững.

- **EN.GHG.CO2.PC.CE.AR5 – CO2 emissions excluding LULUCF per capita**  
  - **Ý nghĩa**: Lượng phát thải CO2 bình quân đầu người (loại trừ thay đổi sử dụng đất & rừng), phản ánh trực tiếp tác động khí nhà kính từ công nghiệp và tiêu dùng năng lượng.  
  - **Lý do chọn**: Đây là biến môi trường trọng tâm, dùng làm trục Y trong scatter Kuznets, phân tích phân hóa giữa các nhóm thu nhập / region, và nhận diện các nước phát thải cao.

- **EG.USE.PCAP.KG.OE – Energy use (kg of oil equivalent per capita)**  
  - **Ý nghĩa**: Mức tiêu thụ năng lượng bình quân đầu người quy đổi về kg dầu tương đương, thể hiện cường độ sử dụng năng lượng.  
  - **Lý do chọn**: Dùng để giải thích động lực phía sau phát thải CO2, đặc biệt khi kết hợp với tỉ trọng công nghiệp (Industry value added) ở các nước đang phát triển.

- **EG.ELC.ACCS.ZS – Access to electricity (% of population)**  
  - **Ý nghĩa**: Tỉ lệ dân số có điện, phản ánh mức độ phát triển hạ tầng năng lượng và phúc lợi cơ bản.  
  - **Lý do chọn**: Cho phép đánh giá *trade-off* giữa mở rộng điện khí hóa (nhất là ở khu vực nghèo như Sub-Saharan Africa) và kiểm soát phát thải / chuyển dịch năng lượng sạch.

- **EG.CFT.ACCS.ZS – Access to clean fuels and technologies for cooking (% of population)**  
  - **Ý nghĩa**: Tỉ lệ dân số sử dụng nhiên liệu sạch cho nấu nướng, gắn trực tiếp với sức khỏe, chất lượng sống và quá trình “xanh hóa” tiêu dùng hộ gia đình.  
  - **Lý do chọn**: Kết hợp với Access to electricity để tạo nhóm biến “Hạ tầng & Phúc lợi”, dùng trong heatmap tương quan và phân tích Sub-Saharan Africa (mục tiêu 7, 8, 9).

- **SP.POP.TOTL – Population, total**  
  - **Ý nghĩa**: Quy mô dân số tuyệt đối của quốc gia.  
  - **Lý do chọn**: Dùng như **trọng số** cho các phép tính trung bình gia quyền (toàn cầu, theo income/region) và làm kích thước bong bóng trong bubble chart (mục tiêu 10).

- **AG.LND.FRST.ZS – Forest area (% of land area)**  
  - **Ý nghĩa**: Tỉ lệ diện tích đất được phủ rừng, là proxy cho sức khỏe hệ sinh thái và khả năng hấp thụ carbon.  
  - **Lý do chọn**: Cho phép kiểm tra liệu tăng trưởng kinh tế có đi kèm phá rừng (mục tiêu 11) và dùng làm tiêu chí đánh giá “điểm sáng” bền vững (giữ được rừng – mục tiêu 12).

- **EG.FEC.RNEW.ZS – Renewable energy consumption (% of total final energy consumption)**  
  - **Ý nghĩa**: Tỉ trọng năng lượng tái tạo trong tổng năng lượng tiêu thụ cuối cùng.  
  - **Lý do chọn**: Là chỉ số chính để đo **chuyển dịch năng lượng xanh** – so sánh High Income vs Low Income (mục tiêu 8) và tìm các quốc gia vừa tăng trưởng vừa tăng tỉ lệ năng lượng tái tạo (mục tiêu 12).

- **NV.IND.TOTL.ZS – Industry (including construction), value added (% of GDP)**  
  - **Ý nghĩa**: Tỉ trọng khu vực công nghiệp & xây dựng trong GDP.  
  - **Lý do chọn**: Giúp giải thích mức độ công nghiệp hóa và mối quan hệ với tiêu thụ năng lượng / phát thải (mục tiêu 4, 5, 6), đặc biệt tại các nền kinh tế mới nổi.

---

### 4. Quy trình tiền xử lý chi tiết trong notebook

#### 4.1. Bước 1 – Thiết lập môi trường và đọc dữ liệu
- **Import thư viện**:
  - `pandas as pd`, `numpy as np` cho xử lý dữ liệu.
  - `matplotlib.pyplot as plt`, `seaborn as sns` cho các biểu đồ mô tả nhanh (nếu cần).
- **Đọc file CSV chính**:
  - Dùng `pd.read_csv("Data/Dataset.csv", na_values=[".."])` để tự động chuyển `..` thành `NaN`.
  - Gán vào biến `df_raw`.
- **Khảo sát nhanh**:
  - `df_raw.head()`, `df_raw.tail()`, `df_raw.info()` để kiểm tra số dòng, số cột, kiểu dữ liệu.
  - Kiểm tra vài dòng cuối để nhận diện dòng metadata/ghi chú (nếu có).

#### 4.2. Bước 2 – Làm sạch giá trị và loại bỏ dòng rác
- **Chuẩn hóa missing value**:
  - Xác nhận lại rằng `..` đã được chuyển hết sang `NaN`. Nếu phát hiện `..` còn sót, dùng:
    - `df_raw.replace("..", np.nan, inplace=True)`.
- **Loại bỏ dòng metadata / không phải quốc gia**:
  - Tiêu chí đề xuất:
    - Chỉ giữ các dòng có `Country Code` **không rỗng** và **độ dài đúng 3 ký tự**.
    - Loại các dòng mà `Country Name` chứa các chuỗi đặc biệt không phải tên vùng/quốc gia (nếu quan sát thấy, có thể lập danh sách để loại).
  - Kết quả: lưu vào `df_clean`.

- **Đảm bảo kiểu dữ liệu số cho các cột năm**:
  - Sau khi thay `..` bằng `NaN`, ép tất cả cột năm về kiểu số bằng `pd.to_numeric(..., errors="coerce")` để tránh các giá trị chuỗi lẫn trong cột số.
  - Việc ép kiểu này giúp các phép tính thống kê, tính trung bình gia quyền và tương quan hoạt động ổn định, đúng như các bước mô tả trong notebook EDA.

- **Khảo sát chất lượng dữ liệu theo năm và theo chỉ tiêu (từ notebook EDA)**:
  - Tận dụng kết quả trong `01_dataset_overview_and_descriptive_analysis.ipynb`:
    - Đã thống kê được số giá trị thiếu và tỷ lệ thiếu theo từng cột năm (2000–2025), cho thấy các năm rất mới (2023–2025) có tỷ lệ thiếu cao hơn.
    - Đã thống kê số lượng giá trị thiếu theo từng `Series Code` để nhận diện chỉ tiêu nào ít dữ liệu (ví dụ một số chỉ tiêu có chuỗi gần như đầy đủ, một số khác bị khuyết nhiều năm).
  - Trong tiền xử lý, vẫn **giữ đầy đủ các năm 2000–2025**, nhưng cần ghi chú rõ khi phân tích:
    - Các biểu đồ hoặc phân tích nhạy cảm với thiếu dữ liệu (như hồi quy Kuznets, heatmap tương quan) nên ưu tiên những năm/chỉ tiêu có tỷ lệ thiếu thấp, hoặc chủ động lọc bớt các quan sát có quá nhiều `NaN`.

- **Chọn cột theo khoảng năm 2000–2025**:
  - Xác định danh sách cột năm cần dùng bằng list comprehension:
    - Lấy các cột bắt đầu bằng `"2000 "`, `"2001 "`, … `"2025 "`.
  - Giữ lại:
    - 4 cột định danh: `Country Name`, `Country Code`, `Series Name`, `Series Code`.
    - Các cột năm từ 2000 đến 2025.
  - Bỏ qua các cột năm > 2025 nếu xuất hiện, nhằm đồng bộ với phạm vi đề tài.

#### 4.3. Bước 3 – Lọc 10 Series mục tiêu
- **Tạo danh sách `Series Code` cần giữ** từ Mục 3.
- **Lọc theo `Series Code`**:
  - `df_target = df_clean[df_clean["Series Code"].isin(list_codes_10_metrics)]`.
  - Kỳ vọng: mỗi `Country Code` có tối đa 10 dòng (mỗi dòng là 1 chỉ tiêu), nhưng có thể thiếu một số chỉ tiêu ở một số quốc gia (sẽ là `NaN` ở nhiều năm).
- (Tùy chọn) Dùng `Series-Metadata` để đối chiếu lại tên, đơn vị của các Series nếu cần ghi chú thêm.

#### 4.4. Bước 4 – Chuyển từ wide sang long bằng `pd.melt`
- **Mục tiêu**: chuẩn hóa dữ liệu về dạng:
  - Cột định danh: `Country Name`, `Country Code`, `Series Name`, `Series Code`.
  - Cột `Year` (năm) và `Value` (giá trị).
- **Thực hiện**:
  - Đặt `id_vars = ["Country Name", "Country Code", "Series Name", "Series Code"]`.
  - Đặt `value_vars` = danh sách cột năm từ 2000 đến 2025.
  - Dùng `pd.melt` tạo dataframe mới, tạm gọi `df_long_raw_year`, với:
    - Một cột chứa tên cột năm gốc (ví dụ `year_col` hoặc `Year_raw`).
    - Một cột chứa `Value`.
  - Loại bỏ các dòng mà `Value` là `NaN` (không có dữ liệu).

#### 4.5. Bước 5 – Chuẩn hóa cột năm và lọc khoảng thời gian
- **Chuẩn hóa `Year`**:
  - Từ giá trị `Year_raw` (ví dụ `"2000 [YR2000]"`) tách ra phần năm `"2000"` và convert sang kiểu `int`.
  - Tạo cột mới `Year` (kiểu `int`) và có thể loại bỏ `Year_raw` sau khi kiểm tra.

- **Bảo đảm đúng khoảng 2000–2025**:
  - Lọc `df_long = df_long[df_long["Year"].between(2000, 2025)]`.
  - Ghi rõ trong notebook: nếu dataset có dữ liệu sau 2025 thì không đưa vào phân tích chính thức để nhất quán với đề tài.

- **Schema cuối của `df_long`**:
  - Cột: `Country Name`, `Country Code`, `Series Name`, `Series Code`, `Year`, `Value`.

#### 4.6. Bước 6 – Pivot thành bảng Country–Year với 10 metrics
- **Mục tiêu**: tạo bảng trung tâm cho các phân tích định lượng, 1 dòng/1 quốc gia/1 năm với đủ 10 chỉ tiêu.

- **Pivot**:
  - Dùng `pivot` hoặc `pivot_table`:
    - `index = ["Country Name", "Country Code", "Year"]`.
    - `columns = "Series Code"`.
    - `values = "Value"`.
  - Sau pivot:
    - Reset index để đưa các khóa trở lại thành cột thông thường.
    - Đổi tên cột từ `Series Code` sang tên biến thân thiện:
      - `NY.GDP.PCAP.KD` → `gdp_pc_const2015`
      - `NY.GDP.MKTP.KD.ZG` → `gdp_growth`
      - `EN.GHG.CO2.PC.CE.AR5` → `co2_pc`
      - `EG.USE.PCAP.KG.OE` → `energy_use_pc`
      - `EG.ELC.ACCS.ZS` → `access_electricity`
      - `EG.CFT.ACCS.ZS` → `access_clean_cooking`
      - `SP.POP.TOTL` → `population`
      - `AG.LND.FRST.ZS` → `forest_area_pct`
      - `EG.FEC.RNEW.ZS` → `renewable_energy_pct`
      - `NV.IND.TOTL.ZS` → `industry_value_added_pct`

- **Xử lý missing**:
  - Không nội suy cứng nhắc trong bước tiền xử lý.
  - Ghi rõ: các phép tổng hợp sau sẽ dùng hàm mặc định bỏ qua `NaN` (ví dụ `mean(skipna=True)`), và khi vẽ biểu đồ cần kiểm soát số lượng điểm dữ liệu thực tế.

- **Kết quả**:
  - DataFrame `df_tidy_country_year` là dataset trung tâm cho các phân tích tiếp theo.

#### 4.7. Bước 7 – Bổ sung `Region` và `IncomeGroup` (nếu có bảng metadata quốc gia)
- **Mục tiêu**: tạo điều kiện cho phân tích theo khu vực và nhóm thu nhập: High, Upper Middle, Lower Middle, Low Income; và các region như East Asia & Pacific, Europe & Central Asia, Sub-Saharan Africa, v.v.

- **Nguồn dữ liệu**:
  - Nếu có thêm file metadata quốc gia (ví dụ `Metadata_Country.csv` từ World Bank), chứa các cột:
    - `Country Code`, `Region`, `IncomeGroup`.

- **Merge**:
  - Đọc metadata vào `df_country_meta`.
  - Merge với `df_tidy_country_year` trên `Country Code` (left join).
  - Kết quả là `df_tidy_full` với thêm 2 cột `Region`, `IncomeGroup`.

- Nếu hiện tại chưa có metadata quốc gia:
  - Ghi rõ trong notebook đây là bước mở rộng sẽ thực hiện sau (hoặc tự tạo bảng mapping ngoài notebook).

---

### 5. Các bảng trung gian phục vụ 12 mục tiêu trực quan hóa

Từ `df_tidy_country_year` (và `df_tidy_full` khi có Region/Income), tạo các bảng sau. Phần này chỉ mô tả logic groupby/agg, không cần viết code cụ thể trong spec.

#### 5.1. Mục tiêu 1 – Bức tranh toàn cầu GDP pc vs CO2 pc theo thời gian
- **Bảng**: `df_global_year`
- **Cách xây dựng**:
  - Nhóm theo `Year`.
  - Tính:
    - `gdp_pc_const2015` trung bình gia quyền theo `population`.
    - `co2_pc` trung bình gia quyền theo `population`.
  - Kết quả: mỗi dòng = 1 năm với cặp (GDP pc toàn cầu, CO2 pc toàn cầu), dùng cho Line chart 2 trục Y.

#### 5.2. Mục tiêu 2 – Phân hóa CO2 giữa 4 nhóm thu nhập
- **Bảng**: `df_income_year`
- **Cách xây dựng**:
  - Dùng `df_tidy_full`, lọc `Year == 2025` (hoặc năm gần nhất có đủ dữ liệu).
  - Nhóm theo `IncomeGroup`.
  - Lấy phân phối `co2_pc` của từng nhóm để vẽ boxplot (không cần tổng hợp, chỉ cần giữ từng quốc gia trong nhóm).

#### 5.3. Mục tiêu 3 – Kiểm chứng đường cong Kuznets (GDP pc vs CO2 pc)
- **Bảng**: có thể sử dụng trực tiếp `df_tidy_country_year` hoặc một subset:
  - Lọc 1–3 mốc thời gian đại diện (ví dụ 2000, 2010, 2020/2025).
  - Giữ cột: `gdp_pc_const2015`, `co2_pc`, kèm `Region` hoặc `IncomeGroup` để tô màu.
  - Dùng cho scatter với đường hồi quy (regression line).

#### 5.4. Mục tiêu 4, 5, 6 – Phân tích region & động lực công nghiệp
- **Mục tiêu 4 – So sánh East Asia & Pacific vs Europe & Central Asia**  
  - **Bảng**: `df_region_year`
  - **Logic**:
    - Từ `df_tidy_full`, lọc `Region` ∈ {`East Asia & Pacific`, `Europe & Central Asia`}.
    - Nhóm theo `Region`, `Year`.
    - Tính trung bình (hoặc trung bình gia quyền theo `population`) cho:
      - `gdp_growth`, `gdp_pc_const2015`, `co2_pc`.

- **Mục tiêu 5 – Mối quan hệ Industry vs Energy use ở nước đang phát triển**  
  - **Bảng**: `df_dev_countries`
  - **Logic**:
    - Từ `df_tidy_full`, lọc các nước có `IncomeGroup` ≠ `High income`.
    - Giữ cột: `Country Name`, `Year`, `industry_value_added_pct`, `energy_use_pc`, có thể thêm `Region` hoặc `IncomeGroup`.
    - Dùng cho scatter plot (Industry vs Energy use) để xem xu hướng công nghiệp hóa – cường độ năng lượng.

- **Mục tiêu 6 – Top 10 quốc gia CO2 pc cao nhất**  
  - **Bảng**: `df_top10_co2`
  - **Logic**:
    - Cho mỗi `Year` (hoặc các năm trọng tâm), sắp xếp các quốc gia theo `co2_pc` giảm dần.
    - Chọn top 10 quốc gia, giữ thêm `Region`, `IncomeGroup`, `population`.
    - Dùng cho bar chart (nằm ngang hoặc bar race).

#### 5.5. Mục tiêu 7, 8, 9 – Chuyển dịch năng lượng & hạ tầng
- **Mục tiêu 7 – Ma trận tương quan giữa hạ tầng, CO2, tăng trưởng**  
  - **Bảng**: `df_corr_features`
  - **Logic**:
    - Từ `df_tidy_country_year` (hoặc một năm đại diện), chọn các cột:
      - `access_electricity`, `access_clean_cooking`, `co2_pc`,
      - `gdp_pc_const2015`, `gdp_growth`, `renewable_energy_pct`, `industry_value_added_pct`.
    - Loại bỏ các dòng có quá nhiều `NaN`.
    - Tính ma trận tương quan (Pearson) giữa các biến trên, dùng cho heatmap.

- **Mục tiêu 8 – So sánh tốc độ chuyển dịch năng lượng tái tạo High vs Low Income**  
  - **Bảng**: `df_renewable_income_year`
  - **Logic**:
    - Từ `df_tidy_full`, lọc các nước có `IncomeGroup` ∈ {`High income`, `Low income`}.
    - Nhóm theo `IncomeGroup`, `Year`.
    - Tính `renewable_energy_pct` trung bình gia quyền theo `population`.
    - Dùng cho line chart 2 đường (High vs Low Income) qua thời gian.

- **Mục tiêu 9 – Tác động của điện & nhiên liệu sạch đến GDP ở Sub-Saharan Africa**  
  - **Bảng**: `df_ssa_energy_gdp`
  - **Logic**:
    - Từ `df_tidy_full`, lọc `Region == "Sub-Saharan Africa"`.
    - Giữ các cột: `Year`, `Country Name`, `gdp_pc_const2015`, `access_electricity`, `access_clean_cooking`.
    - Có thể tổng hợp theo `Year` (trung bình gia quyền) để vẽ stacked area/line, hoặc vẽ nhiều đường cho một vài quốc gia tiêu biểu.

#### 5.6. Mục tiêu 10, 11, 12 – Môi trường sinh thái, dân số và “điểm sáng”
- **Mục tiêu 10 – Bubble chart GDP–CO2–Population–Region**  
  - **Bảng**: sử dụng `df_tidy_full` hoặc một subset như `df_bubble_year`.
  - **Logic**:
    - Lọc 1–2 năm đại diện gần đây.
    - Giữ cột:
      - `gdp_pc_const2015` (trục X),
      - `co2_pc` (trục Y),
      - `population` (size),
      - `Region` (color).
    - Dùng cho bubble chart để nhìn bức tranh đa chiều toàn cầu.

- **Mục tiêu 11 – Tăng trưởng kinh tế vs diện tích rừng**  
  - **Bảng**: `df_forest_gdp_growth`
  - **Logic**:
    - Từ `df_tidy_country_year`, giữ các cột `gdp_growth`, `forest_area_pct`, `Year`, `Country Name`.
    - Có thể lọc 1 số năm hoặc tính trung bình giai đoạn để giảm nhiễu.
    - Dùng cho scatter plot kiểm tra xem tăng trưởng cao có đi kèm giảm tỉ lệ rừng hay không.

- **Mục tiêu 12 – Nhận diện các quốc gia “điểm sáng”**  
  - **Bảng**: `df_green_outliers`
  - **Logic**:
    - Xác định một giai đoạn phân tích (ví dụ trung bình 2015–2025).
    - Tính các thống kê trung bình theo quốc gia cho:
      - `gdp_growth` (tăng trưởng),
      - `renewable_energy_pct` (tỉ lệ năng lượng tái tạo),
      - `forest_area_pct` (diện tích rừng).
    - Đặt các ngưỡng percentile, ví dụ:
      - `gdp_growth` > percentile 60–70 (tăng trưởng tương đối cao),
      - `renewable_energy_pct` > percentile 60–70 (tỉ lệ năng lượng tái tạo cao),
      - `forest_area_pct` không giảm đáng kể (hoặc duy trì trên một ngưỡng nhất định so với năm 2000).
    - Lọc các quốc gia thỏa cả ba tiêu chí ⇒ tạo `df_green_outliers` dùng cho radar chart hoặc bar chart.

---

### 6. Lưu ý & giả định
- **Giới hạn thời gian**: Chỉ sử dụng dữ liệu giai đoạn 2000–2025; các năm ngoài khoảng này (nếu có) được loại bỏ để bám sát đề tài.
- **Thiếu dữ liệu**:
  - Không nội suy dữ liệu tại bước tiền xử lý trừ khi thực sự cần thiết cho một số phân tích cụ thể (sẽ ghi rõ sau trong notebook nếu áp dụng).
  - Các phép tổng hợp ưu tiên dùng hàm trung bình/median với tùy chọn bỏ qua `NaN`.
- **Trọng số dân số**:
  - Khi tính trung bình toàn cầu hoặc theo nhóm (Region/IncomeGroup), ưu tiên **trung bình gia quyền theo `population`** để phản ánh đúng đóng góp của các nước đông dân.
- **Tái sử dụng dataset**:
  - `df_tidy_country_year` và `df_tidy_full` là 2 bảng **cốt lõi**, nên được sử dụng nhất quán trong toàn bộ notebook để tránh phải lặp lại các bước melt/pivot.

