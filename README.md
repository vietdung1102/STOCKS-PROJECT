I. Ngữ cảnh

Dự án Final Project MD02 được thực hiện với mục tiêu phân tích dữ liệu chứng khoán VN30 nhằm cung cấp thông tin về:Xu hướng biến động giá cổ phiếu qua các năm, phân tích cơ bản doanh nghiệp (ROA, ROE, EPS, Nợ/VCSH, v.v.)và định giá cổ phiếu qua các chỉ số P/E, P/B
Báo cáo được xây dựng bằng Power BI với dữ liệu gốc là CSV Datasets.
Báo cáo nhằm mục tiêu đưa ra khuyến nghị các cổ phiếu nên đầu tư.

II. Flowchart

CSV Dataset->Power BI->Dashboard->Phân tích và trực quan hóa kết quả

III. Các bước thực hiện

3.1: Input data
-Input dữ liệu giá cổ phiếu VN30 và chỉ số VNINDEX (giai đoạn 2015–2024).
-Input các chỉ số tài chính của các công ty VN30 trong giai đoạn từ 2015 đến 2024

3.2: Xây dựng database schema

-Tạo các bảng sau trong power BI:
 +Bảng dim
 +Bảng dim B/S Account
 +Bảng dim Company
 +Bảng dim I/S Account
 +Bảng dim mã chứng khoán
 +Bảng dim ngày giao dịch
 +Bảng dim year
 +Bảng dữ liệu lịch sử VN30
 +Bảng fact B/S
 +Bảng fact I/S
 +Bảng VN30

-Thiết lập một vài measure=DAX để tính toán chỉ số
 +P_B_Ratio = 
VAR LastYear = CALCULATE(MAX('DIm_Ngày giao dịch'[Ngày]),ALL('DIm_Ngày giao dịch'[Day],'DIm_Ngày giao dịch'[Month]))
VAR LastClosingPrice =
    CALCULATE(
        SUM('VN30'[Lần cuối]),
        'VN30'[Ngày] = LastYear
    )
VAR _bookvale = [Book value]
RETURN
    DIVIDE(LastClosingPrice, _bookvale, BLANK())
    
+P_E_Ratio = 
VAR LastYear = CALCULATE(MAX('DIm_Ngày giao dịch'[Ngày]),ALL('DIm_Ngày giao dịch'[Day],'DIm_Ngày giao dịch'[Month]))
VAR LastClosingPrice =
    CALCULATE(
        SUM('VN30'[Lần cuối]),
        'VN30'[Ngày] = LastYear
    )
VAR EPS = [EPS]
RETURN
    DIVIDE(LastClosingPrice, EPS, BLANK())

+Top CP Giảm Giá = 
VAR _mack = 
    SELECTEDVALUE('DIm_Mã chứng khoán'[Mã CK])
VAR _topn =
    TOPN(
        SELECTEDVALUE('Parameter'[Parameter]),  
        ALL('DIm_Mã chứng khoán'[Mã CK]),                         
   CALCULATE(SUM(VN30[% Thay đổi])),         
        ASC                                       
    )
RETURN
    IF(
        _mack IN _topn,
        CALCULATE(SUM(VN30[% Thay đổi]))  
    )
    
3.3: Xây dựng dashboard

Dashboard gồm 3 trang chính:

1. Tổng quan VN30

So sánh chỉ số VN30 vs VNINDEX

Bảng giá VN30 (cao, thấp, đóng, biến động, KL)

Cơ cấu ngành VN30 (Biểu đồ tròn)

Biểu đồ nến & Bollinger Band phân tích kỹ thuật theo năm

2.Bộ lọc cổ phiếu



