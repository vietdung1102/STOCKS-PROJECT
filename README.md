I. Context

The Final Project MD02 was done with the goal of analyzing VN30 stock data to provide information on: Trends in stock price fluctuations over the years, analysis of business fundamentals (ROA, ROE, EPS, Debt/Equity, etc.) and stock valuation through P/E, P/B ratios
The report was built by using Power BI with the original data being the CSV Datasets.
The goal is to give recommendations on which stocks to invest in.

II. Flowchart

CSV Dataset->Power BI->Dashboard->Analyze and visualize results

III. Steps

3.1: Input data

-Input VN30 stock price data and VNINDEX index (period 2015–2024).

-Input financial indicators of VN30 companies in the period from 2015 to 2024

3.2: Building database schema

-Create these tables in power BI:dim,dim B/S Account,dim Company,dim I/S Account,dim mã chứng khoán,dim ngày giao dịch, dim year, dữ liệu lịch sử VN30,fact B/S, fact I/S, VN30

-Create DAX Measures to calculate some metrics:

Example:

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
    
3.3: Building dashboard

The dashboard is consisted of 3 pages:

1. VN30 overview includes: Compare VN30 vs VNINDEX index, VN30 price table (high, low, close, volatility, volume), VN30 industry structure (pie chart), Candlestick chart & Bollinger Band technical analysis by year

2. Stock filter includes:
Top N stock filter (Top increase, decrease, highest trading volume), P/E, P/B chart by stock code and by year

P/E (Price to Earnings Ratio): is the ratio between stock price and profit per share (EPS). This index shows the price that investors are willing to pay for each dollar of profit that the company generates.Compared to the industry: When the P/E is higher than the industry average, the stock may be overvalued, while a low P/E may indicate that the stock is undervalued.

P/B is the ratio of the stock price to the book value per share (BVPS). It shows the price that investors pay compared to the real value of the company's assets. A high P/B may indicate that the stock price is too high compared to the real value of the assets. Conversely, a low P/B indicates potential undervaluation.

3. Basic analysis of the business

Net revenue: the amount of money earned from business activities after deducting related expenses such as taxes and deductions

Earnings before tax (EBIT – Earnings Before Interest and Taxes): reflects the profitability of the business before deducting interest expenses and corporate income tax.

Profit after tax (Profit After Tax) is the remaining profit after deducting expenses, including corporate income tax.

ROA is the ratio of net profit to assets (Return on total assets), which is an indicator to measure the profitability of each unit of the company's assets.


ROE is the Return on Common Equity, which measures the profitability of each dollar of common stockholders' equity.

EBIT is the profit a company makes from its operations, before interest and corporate income tax payments.

Current ratio shows the ratio of current assets to current liabilities, reflecting the company's current ability to pay off its current liabilities.

The quick ratio shows how well a company can pay off short-term debts with highly liquid assets such as cash and cash equivalents.

EPS (Earnings per Share) measures a company's after-tax profit allocated to each outstanding common share


IV. Values gained

4.1 Skills

Processing and analyzing financial data using Power BI

Designing financial dashboards

Writing DAX formulas to visualize financial indicators

Data Storytelling

4.2 Knowledge

Compare the fluctuations between VN30 and VNINDEX

Understand the basic financial indicators

Compare and evaluate stock values ​​over a period of a time and among different groups of industry.

