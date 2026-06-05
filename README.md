# Báo cáo Tổng quan Dự án: Phân tích & Dự phóng Cổ phiếu Ngân hàng

## 1. Ngữ cảnh (Context)
Dự án được khởi xướng xuất phát từ **nhu cầu thực tế của một nhóm nhà đầu tư** quan tâm sâu sắc đến nhóm cổ phiếu ngân hàng (đặc biệt là các ngân hàng có trụ sở hoặc tầm ảnh hưởng lớn tại miền Bắc và các mã trụ cột như VCB, ACB, TCB, VPB, TPB). 

Thị trường chứng khoán luôn biến động và việc ra quyết định phụ thuộc vào quá nhiều biến số phân mảnh. Các nhà đầu tư cần một hệ thống có khả năng:
- Định lượng hóa sức mạnh Mua/Bán của cổ phiếu thay vì cảm tính.
- Cung cấp một cái nhìn dài hạn về tiềm năng giá trong tương lai (đặc biệt là mục tiêu năm 2026).
- Gom toàn bộ các chỉ số kỹ thuật và tài chính phức tạp thành các khuyến nghị hành động đơn giản, trực quan và dễ hiểu nhất.

## 2. Quy trình thực hiện (Chi tiết)
Quá trình xây dựng hệ thống được thực hiện qua một luồng dữ liệu (Data Pipeline) khép kín và tự động:

### Bước 2.1: Thu thập & Chuẩn hóa Dữ liệu (Data Ingestion & Processing)
- **Nguồn dữ liệu:** Dữ liệu giá đóng cửa hàng ngày (Daily Prices) và các báo cáo tài chính định kỳ từ file CSV.
- **Lưu trữ & Chuẩn hóa (PostgreSQL Database):** Dữ liệu thô được đổ vào schema Staging (`stg`), sau đó được Transform sang schema Data Warehouse (`dwh`) với các bảng Fact nâng cao như `fact_yearly_metrics_fin` (Tài chính) và đặc biệt là `fact_price_monthly_end` (Khung dữ liệu kỹ thuật theo Tháng).
- **Tiền xử lý (Python/SQLAlchemy):** Ngăn chặn hoàn toàn hiện tượng "Nhìn thấy tương lai" (Look-ahead bias) bằng cách thiết lập màng lọc dữ liệu ngặt nghèo (Chỉ học dữ liệu lịch sử từ 2021 đến 2025). Dữ liệu sau đó được kết nối trực tiếp, xử lý nhiễu để sẵn sàng đưa vào các mô hình tính toán.

### Bước 2.2: Xây dựng Mô hình Chấm điểm (Scoring Model)
Sử dụng Python để xây dựng hệ thống chấm điểm tổng hợp dựa trên 13 chỉ báo cốt lõi (Khóa chặt dữ liệu đến cuối 2025):
- **8 Chỉ báo Kỹ thuật (Tầm nhìn Dài hạn theo Tháng):** RSI, MA20, EMA20, MACD, Bollinger Bands (BB), ATR, Volume, OBV. Các chỉ báo này được tính toán trên **Khung Tháng (Monthly Timeframe)**, hấp thụ hoàn toàn các nhiễu động ngày để cung cấp tín hiệu cực kỳ bền vững.
- **5 Chỉ báo Tài chính (Định giá & Sức khỏe):** Sử dụng thuật toán Đo lường độ dốc **Xu hướng (Trend) 5 năm (2021-2025)** của P/E, P/B, ROE, NPL và CAR. Cổ phiếu chỉ được đánh giá cao khi có xu hướng cơ bản cải thiện mạnh mẽ.
- **Đầu ra:** Trả về Tỷ lệ % MUA (Buy Ratio) và Tỷ lệ % BÁN (Sell Ratio) cho mỗi cổ phiếu.

### Bước 2.3: Mô hình Học máy Dự phóng Giá (Machine Learning Prediction)
- **Thuật toán:** Hồi quy tuyến tính (Linear Regression) thông qua thư viện `scikit-learn`.
- **Huấn luyện:** Mô hình học từ dữ liệu lịch sử giá (Trung bình, Cao nhất, Thấp nhất) trong giai đoạn 2021-2025.
- **Dự phóng 2026:** Xuất ra 3 kịch bản giá cho năm 2026: Kịch bản Xấu (Min), Cơ sở (Avg) và Tích cực (Max), từ đó xác định Xu hướng (Trend) là Tăng hay Giảm.

### Bước 2.4: Tổng hợp Chiến lược & Khuyến nghị (Evaluation Strategy)
Một module logic (Rule-based) được xây dựng để đối chiếu chéo giữa **(Tỷ lệ Buy/Sell)** và **(Xu hướng 2026)**, từ đó sinh ra kết luận tự động:
- VD: Tỷ lệ Buy > 50% + Xu hướng Tăng $\rightarrow$ **Đồng thuận TÍCH CỰC** (Cung cấp vùng giá Mua gom).
- VD: Tỷ lệ Buy > 50% + Xu hướng Giảm $\rightarrow$ **Phân kỳ** (Định giá tốt nhưng xu hướng xấu $\rightarrow$ Khuyên đứng ngoài).

### Bước 2.5: Trực quan hóa dữ liệu (Web Dashboard / PowerBI)
Đầu ra được chuyển thành một ứng dụng Web Application (Flask API) với thiết kế UI/UX xuất sắc:
- **Tone màu "Mệnh Thủy Sáng":** Giao diện Light Blue trong trẻo, sang trọng.
- **Phong cách "Vó ngựa phi nước đại":** Hiệu ứng hoạt ảnh (animation) cuộn trào tốc độ cao, các khối hình nghiêng chồm tạo cảm giác động lượng, kết hợp hiệu ứng Glassmorphism.
- **Hệ thống Lưới Biểu đồ Động:** Tách biệt rõ ràng 13 chỉ báo (8 biểu đồ Kỹ thuật + 5 biểu đồ Tài chính) thành 13 Chart riêng rẽ. Mỗi biểu đồ đi kèm một chẩn đoán Buy/Sell/Neutral tự động và mốc thời gian rõ ràng (VD: T12/2025) để người dùng không bị nhầm lẫn. Màu sắc (Vàng cam, Trắng, Slate) được tối ưu hóa cho hiển thị trên nền Sáng (Light Theme).

## 3. Kết luận
Dự án đã thành công trong việc chuyển hóa những dòng dữ liệu tài chính/kỹ thuật thô cứng thành một **"Cố vấn Đầu tư Ảo"** toàn diện. Hệ thống không chỉ dừng lại ở mức độ cung cấp số liệu (Dashboard thông thường), mà đã tiến lên một bậc cao hơn là **Hệ thống Khuyến nghị (Recommendation System)** tự động đưa ra các kịch bản hành động cụ thể cho từng cổ phiếu.

Kiến trúc phần mềm được phân tách rõ ràng (Data $\rightarrow$ Model $\rightarrow$ API $\rightarrow$ UI), biến môi trường (.env) linh hoạt, sẵn sàng để đóng gói (Deploy) lên các máy chủ Production và mở rộng thêm hàng chục mã cổ phiếu khác trong tương lai.

## 4. Gain Value (Giá trị mang lại)

1. **Cho Nhà đầu tư (End-User):**
   - **Tiết kiệm thời gian:** Không cần mở chục đồ thị hay đọc hàng đống báo cáo. Mọi chỉ số quan trọng và kết luận nằm gọn trong 1 trang màn hình duy nhất.
   - **Quyết định khách quan:** Loại bỏ hoàn toàn cảm tính (FOMO). Việc mua bán dựa trên các chỉ số được đo lường cụ thể và có backtest bằng Linear Regression.
   - **Giao diện truyền cảm hứng:** Một UI đẹp, mượt mà và trực quan (có sẵn Data Dictionary giải thích thuật ngữ) giúp nhà đầu tư F0 cũng có thể dễ dàng sử dụng và thấu hiểu.

2. **Cho Hệ thống & Quản trị:**
   - **Tự động hóa hoàn toàn (Automation):** Luồng dữ liệu chạy xuyên suốt từ CSV đến UI mà không cần can thiệp thủ công tính toán lại.
   - **Tính quy mô (Scalability):** Dễ dàng cắm thêm các luồng dữ liệu API Real-time hoặc thêm các mã ngân hàng mới chỉ bằng việc thả file CSV vào hệ thống.
   - **Triển khai linh hoạt:** Hoàn toàn có thể nhúng trực tiếp API này vào PowerBI, hoặc chạy độc lập trên Web Server như một nền tảng SaaS riêng biệt.

3. **Cho Người làm đề tài (Tác giả):**
   - **Về Technical (Kỹ thuật):** Làm chủ tư duy xây dựng luồng dữ liệu (Data Pipeline) hoàn chỉnh (End-to-End). Tích hợp thành công thư viện tính toán (Pandas, Numpy) với mô hình Machine Learning (Scikit-Learn). Rèn luyện kỹ năng xây dựng Web API bằng Flask và tối ưu hóa hệ thống bằng biến môi trường (.env) để sẵn sàng triển khai (Deploy) thực tế.
   - **Về Domain (Nghiệp vụ Tài chính):** Hiểu sâu sắc bộ khung đánh giá sức khỏe của một ngân hàng (NPL, CAR, ROE). Biết cách lượng hóa các chỉ báo kỹ thuật (RSI, MA20) và kết hợp nhịp nhàng với định giá cơ bản (P/E, P/B) để tạo ra một chiến lược đầu tư (Evaluation Strategy) thực chiến trên thị trường chứng khoán.
   - **Về UI/UX (Trải nghiệm người dùng):** Nâng tầm tư duy thiết kế web hiện đại với xu hướng Glassmorphism. Trực quan hóa dữ liệu phức tạp một cách gọn gàng qua Chart.js. Biết cách sử dụng CSS (Animation tốc độ, Skew góc nghiêng, Neon glow) để thổi hồn vào sản phẩm, tạo ra một giao diện mang tính "động lượng" (Vó ngựa phi nước đại) và truyền cảm hứng mạnh mẽ.
