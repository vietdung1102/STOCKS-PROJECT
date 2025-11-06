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

-Tạo các bảng sau trong power BI:Bảng dim, Bảng dim B/S Account,Bảng dim Company,Bảng dim I/S Account,Bảng dim mã chứng khoán,Bảng dim ngày giao dịch, Bảng dim year, Bảng dữ liệu lịch sử VN30, Bảng fact B/S, Bảng fact I/S, Bảng VN30

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

1. Tổng quan VN30 bao gồm: So sánh chỉ số VN30 vs VNINDEX, Bảng giá VN30 (cao, thấp, đóng, biến động, KL), Cơ cấu ngành VN30 (Biểu đồ tròn), Biểu đồ nến & Bollinger Band phân tích kỹ thuật theo năm

2.Bộ lọc cổ phiếu bao gồm: Bộ lọc Top N cổ phiếu (Top tăng giá, giảm giá, KL giao dịch cao nhất), Biểu đồ P/E, P/B theo mã cổ phiếu và theo năm

P/E (Price to Earnings Ratio): là tỷ lệ giữa giá cổ phiếu và lợi nhuận trên mỗi cổ phiếu (EPS). Chỉ số này cho biết mức giá mà nhà đầu tư sẵn lòng trả cho mỗi đồng lợi nhuận mà công ty tạo ra.
So với ngành: Khi P/E cao hơn trung bình ngành, cổ phiếu có thể được định giá quá cao, trong khi một P/E thấp có thể cho thấy cổ phiếu đang bị định giá thấp.
P/B là tỷ lệ giữa giá cổ phiếu và giá trị sổ sách trên mỗi cổ phiếu (BVPS). Nó cho biết mức giá mà nhà đầu tư trả so với giá trị tài sản thực của công ty.Một P/B cao có thể cho thấy giá cổ phiếu quá cao so với giá trị thực của tài sản. Ngược lại, P/B thấp cho thấy tiềm năng định giá thấp.

3.Phân tích cơ bản doanh nghiệp

Doanh thu thuần: số tiền thu được từ hoạt động kinh doanh sau khi đã trừ đi các chi phí liên quan như các loại thuế và các khoản giảm trừ 
Thu nhuận trước thuế (EBIT – Earnings Before Interest and Taxes): phản ánh khả năng sinh lời của doanh nghiệp trước khi trừ các khoản chi phí lãi vay và thuế thu nhập doanh nghiệp.
Thu nhuận sau thuế (Profit After Tax) là khoản lợi nhuận còn lại sau khi đã trừ các chi phí, bao gồm cả thuế thu nhập doanh nghiệp.
ROA là tỷ số lợi nhuận ròng trên tài sản (Return on total assets), là chỉ tiêu đo lường khả năng sinh lợi trên mỗi đồng tài sản của công ty.
ROE là tỷ số lợi nhuận ròng trên vốn chủ sở hữu (Return on common equyty), là tỷ số đo lường khả năng sinh lợi trên mỗi đồng vốn của cổ đông thường.
EBIT là một khoản lợi nhuận mà một công ty thu được từ việc kinh doanh, chưa trừ đi các khoản trả lãi vay và thuế thu nhập doanh nghiệp.
Chỉ số thanh toán hiện hành cho biết tỉ số giữa tài sản lưu động hiện có và nợ ngắn hạn, phản ánh khả năng hiện tại của doanh nghiệp trong việc thanh toán các khoản nợ ngắn hạn đó.
Chỉ số thanh toán nhanh thể hiện khả năng doanh nghiệp có thể thanh toán các khoản vay ngắn hạn bằng những tài sản có tính thanh khoản cao như tiền và các khoản tương đương tiền. 
EPS (Lợi nhuận trên mỗi cổ phần) chỉ số đo lường lợi nhuận sau thuế của một công ty được phân bổ cho mỗi cổ phiếu phổ thông đang lưu hành

IV. Giá trị đạt được

4.1 Kỹ năng phát triển

Xử lý và phân tích dữ liệu tài chính bằng Power BI

Thiết kế dashboard chuyên nghiệp về tài chính

Viết công thức DAX để để trực quan hóa các chỉ số tài chính 

Kể chuyện bằng dữ liệu (Data Storytelling)

4.2 Kiến thức nắm được

Hiểu cấu trúc và vận hành của chỉ số VN30, VNINDEX

Nắm rõ các chỉ số tài chính cơ bản và cách diễn giải

Biết cách so sánh và đánh giá cổ phiếu theo thời gian và theo ngành

Thành thạo quy trình ETL (Extract – Transform – Load) và mô hình hóa dữ liệu trong Power BI

