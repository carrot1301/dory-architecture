# DORY - Quant & AI Architecture

> DORY là dashboard học và thực hành đầu tư định lượng cho thị trường chứng khoán Việt Nam.
> Mục tiêu của dự án là giúp người mới đọc dữ liệu thị trường có hệ thống hơn, đồng thời tạo một nền tảng để thử nghiệm các ý tưởng quant, risk và AI explanation.

## Đọc Nhanh

DORY không được thiết kế để "phím hàng" hoặc dự đoán chắc thắng. Dự án tập trung vào 4 việc:

1. Thu thập và chuẩn hóa dữ liệu thị trường Việt Nam.
2. Tính các tín hiệu định lượng cơ bản như momentum, valuation proxy, sức khỏe tài chính và rủi ro.
3. Biến kết quả phân tích thành dashboard dễ đọc cho người mới.
4. Dùng AI assistant để giải thích lại insight, giả định và rủi ro bằng ngôn ngữ dễ hiểu hơn.

## Pipeline Tổng Quan

```mermaid
flowchart LR
    A[Market Data<br/>OHLCV cổ phiếu<br/>VN-Index / VN30<br/>Tin tức thị trường] --> B[Data Engine<br/>Làm sạch dữ liệu<br/>Chuẩn hóa timestamp<br/>Cache & fallback]
    C[Financial Data<br/>BCTC<br/>P/E, P/B, EPS<br/>Vốn hóa] --> B
    D[User Inputs<br/>Ticker<br/>Danh mục<br/>Số vốn<br/>Risk profile] --> E[Quant Modules]

    B --> E[Quant Modules<br/>TA signals<br/>Factor scoring<br/>VN30 ranking<br/>Risk assumptions]
    E --> F[Risk & Simulation<br/>VaR<br/>Stress test<br/>Backtest<br/>Portfolio view]
    E --> G[Market Overview<br/>VN-Index / VN30<br/>Trend score<br/>MA / RSI / Volatility]
    F --> H[Frontend Dashboard<br/>Plotly charts<br/>Score cards<br/>Ranking table<br/>Learning UI]
    G --> H
    H --> I[AI Assistant<br/>Giải thích kết quả<br/>Tóm tắt rủi ro<br/>Gợi ý câu hỏi học tiếp]
```

## Cấu Trúc Hệ Thống

```mermaid
flowchart TB
    subgraph Frontend["Frontend"]
        UI[HTML / CSS / JavaScript]
        Charts[Plotly charts]
        UX[Gamified learning UI]
    end

    subgraph Backend["Backend API - FastAPI"]
        MarketAPI[Market overview API]
        StockAPI[Stock OHLCV API]
        QuantAPI[Quant score API]
        RiskAPI[Risk & simulation API]
        AiAPI[AI advisor API]
        AuthAPI[Auth / user state API]
    end

    subgraph Core["Core Engines"]
        DataEngine[data_engine.py]
        TaEngine[ta_engine.py]
        RiskEngine[risk_engine.py]
        FactorEngine[factor_model.py]
        ForecastEngine[forecast_engine.py]
        BctcEngine[bctc_engine.py]
    end

    subgraph External["External / Supporting Services"]
        MarketSource[Market data endpoints]
        Supabase[Supabase Auth & DB]
        News[RSS / News sources]
        AIProvider[AI model provider]
    end

    UI --> MarketAPI
    UI --> StockAPI
    UI --> QuantAPI
    UI --> RiskAPI
    UI --> AiAPI
    UI --> AuthAPI

    MarketAPI --> DataEngine
    StockAPI --> DataEngine
    QuantAPI --> FactorEngine
    QuantAPI --> TaEngine
    RiskAPI --> RiskEngine
    AiAPI --> AIProvider
    AuthAPI --> Supabase

    DataEngine --> MarketSource
    BctcEngine --> Supabase
    BctcEngine --> MarketSource
    DataEngine --> News
```

## Luồng Phân Tích

| Bước | Module | Vai trò |
|---|---|---|
| 1 | Market overview | Đọc bối cảnh VN-Index/VN30 trước khi chọn từng mã |
| 2 | Stock data | Lấy OHLCV, giá gần nhất và dữ liệu lịch sử của cổ phiếu |
| 3 | Technical analysis | Tính MA, RSI, volatility, trend và các tín hiệu kỹ thuật cơ bản |
| 4 | Fundamental snapshot | Tổng hợp BCTC, định giá và chỉ số tài chính quan trọng |
| 5 | Quant scoring | Chấm điểm theo momentum, value và financial health |
| 6 | Risk simulation | Ước lượng VaR, stress test, backtest và rủi ro danh mục |
| 7 | AI explanation | Giải thích kết quả theo ngôn ngữ dễ hiểu, không thay thế quyết định đầu tư |

## Market Overview Realtime Logic

Daily index data có thể bị trễ trong phiên, nên DORY dùng cách ghép dữ liệu:

```mermaid
flowchart LR
    A[Daily index candles<br/>resolution = 1D] --> C[Historical series<br/>MA20 / MA50 / MA200<br/>Volatility / YTD]
    B[Intraday index candles<br/>resolution = 1 minute] --> D[Latest market print<br/>VN-Index / VN30 hiện tại]
    C --> E[Market overview payload]
    D --> E
    E --> F[Frontend refresh mỗi 30 giây<br/>khi tab tổng quan đang mở]
```

Cách này giữ các chỉ báo dài hạn ổn định, nhưng vẫn cho phép giá VN-Index/VN30 trên dashboard phản ánh dữ liệu trong phiên.

## Các Thành Phần Chính

### 1. Market Dashboard

- Theo dõi VN-Index và VN30 trước khi phân tích từng cổ phiếu.
- Hiển thị trend score, MA20/50/200, RSI, volatility và drawdown.
- Giúp người mới tránh nhìn một mã cổ phiếu mà bỏ qua bối cảnh thị trường.

### 2. Stock Analysis

- Xem dữ liệu giá, kỹ thuật, BCTC và tin tức của từng mã.
- Tách các nhóm thông tin để người dùng hiểu vì sao một mã có điểm cao/thấp.

### 3. VN30 Ranking

- Chấm điểm các mã VN30 theo một số nhóm yếu tố định lượng.
- Mục tiêu là hỗ trợ học tư duy ranking, không phải tạo danh sách mua/bán tuyệt đối.

### 4. Risk & Simulation

- Mô phỏng rủi ro danh mục với VaR và stress test.
- Cho người dùng thấy cùng một kỳ vọng lợi nhuận có thể đi kèm mức drawdown khác nhau.

### 5. AI Assistant

- Tóm tắt dashboard bằng ngôn ngữ dễ hiểu.
- Giải thích ý nghĩa của chỉ số, rủi ro và giả định.
- Hướng người mới đặt câu hỏi tốt hơn thay vì chỉ nhìn một con số.

## Nguyên Tắc Thiết Kế

- **Educational first:** ưu tiên học và hiểu dữ liệu hơn là ra tín hiệu mua bán.
- **Quant but readable:** có logic định lượng nhưng giao diện phải đủ dễ hiểu cho người mới.
- **Assumption-aware:** mô hình phải thể hiện giả định, rủi ro và giới hạn.
- **Vietnam-market focused:** ưu tiên dữ liệu, ngôn ngữ và workflow phù hợp thị trường Việt Nam.
- **No guaranteed prediction:** không trình bày kết quả như dự báo chắc chắn.

## Giới Hạn Hiện Tại

- Chưa phải trading bot tự động.
- Chưa thay thế phân tích chuyên nghiệp hoặc tư vấn tài chính.
- Chất lượng kết quả phụ thuộc vào độ đầy đủ và độ trễ của nguồn dữ liệu.
- Một số mô hình scoring vẫn ở mức học thuật/thực nghiệm, cần thêm kiểm định ngoài mẫu.
- Backtest cần tiếp tục được kiểm tra kỹ để tránh overfitting và look-ahead bias.

## Roadmap Gợi Ý

1. Thêm kiểm định factor ngoài mẫu cho thị trường Việt Nam.
2. Bổ sung giải thích chi tiết hơn cho từng điểm số trong VN30 ranking.
3. Mở rộng dữ liệu intraday và lịch sử điều chỉnh giá.
4. Cải thiện portfolio optimizer và risk attribution.
5. Thêm benchmark rõ ràng giữa chiến lược, VN30 và VN-Index.
6. Tạo chế độ học cho người mới: từ chỉ số cơ bản đến factor investing.

## Disclaimer

DORY là công cụ học tập và nghiên cứu dữ liệu. Nội dung trong dashboard chỉ mang tính tham khảo, không phải khuyến nghị đầu tư, không đảm bảo lợi nhuận và không thay thế tư vấn tài chính độc lập.
