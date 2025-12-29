# TÀI LIỆU ĐẶC TẢ YÊU CẦU KẾT NỐI CPO VÀO HUB TRUNG TÂM

## 📋 Mục lục
- [Thuật ngữ](#thuật-ngữ)
- [1. Tổng quan](#1-tổng-quan)
- [2. Lợi ích khi kết nối](#2-lợi-ích-khi-kết-nối)
- [3. Kiến trúc tổng thể](#3-kiến-trúc-tổng-thể)
- [4. Luồng kết nối](#4-luồng-kết-nối)
- [5. Yêu cầu kỹ thuật](#5-yêu-cầu-kỹ-thuật)
- [6. Quy trình onboarding](#6-quy-trình-onboarding)
- [7. Hỗ trợ và liên hệ](#7-hỗ-trợ-và-liên-hệ)

---

## 📖 Thuật ngữ

### Thuật ngữ OCPI cốt lõi

| Thuật ngữ | Tiếng Việt | Định nghĩa | Ví dụ |
|-----------|------------|------------|-------|
| **OCPI** | Giao thức sạc mở | Open Charge Point Interface - Giao thức chuẩn để trao đổi dữ liệu giữa các hệ thống sạc xe điện | OCPI 2.2.1 |
| **CPO** | Nhà vận hành trạm sạc | Charge Point Operator - Đơn vị sở hữu và vận hành trạm sạc xe điện | ABC Charging Co. |
| **eMSP** | Nhà cung cấp dịch vụ | e-Mobility Service Provider - Đơn vị cung cấp app/dịch vụ cho người dùng cuối | VinFast App, Xanh SM |
| **Hub** | Trung tâm kết nối | Nền tảng trung gian kết nối CPO và eMSP, cho phép roaming | Hub Trung tâm |
| **Roaming** | Kết nối liên mạng | Cho phép người dùng của eMSP A sạc tại trạm của CPO B | User Grab sạc tại trạm VinFast |

### Thuật ngữ hạ tầng sạc

| Thuật ngữ | Tiếng Việt | Định nghĩa | Cấu trúc |
|-----------|------------|------------|----------|
| **Location** | Trạm sạc | Vị trí địa lý có chứa 1 hoặc nhiều thiết bị sạc | Trạm sạc Bình Thạnh |
| **EVSE** | Thiết bị sạc | Electric Vehicle Supply Equipment - Thiết bị cung cấp điện cho xe (1 cột sạc) | 1 Location có nhiều EVSE |
| **Connector** | Đầu sạc/Vòi sạc | Đầu cắm vật lý để kết nối với xe điện | 1 EVSE có 1-2 Connector |
| **Charge Point** | Điểm sạc | Tương đương EVSE, thuật ngữ khác | - |
| **Charging Station** | Trạm sạc | Tương đương Location | - |

**Cấu trúc phân cấp**:
```
Location (Trạm sạc)
  └── EVSE (Thiết bị sạc/Cột sạc)
       └── Connector (Đầu sạc/Vòi sạc)
```

### Thuật ngữ phiên sạc

| Thuật ngữ | Tiếng Việt | Định nghĩa | Lifecycle |
|-----------|------------|------------|-----------|
| **Session** | Phiên sạc | Một lần sạc xe từ lúc cắm đến lúc rút vòi | START → ACTIVE → COMPLETED |
| **CDR** | Bản ghi thanh toán | Charge Detail Record - Bản ghi chi tiết về phiên sạc đã hoàn thành, dùng cho thanh toán | Sau khi Session kết thúc |
| **Token** | Mã xác thực | Thông tin nhận diện người dùng (RFID, QR, App User ID) | TOKEN-12345 |
| **Authorization** | Ủy quyền | Quá trình xác thực người dùng có quyền sạc không | Check token trước khi start |
| **Tariff** | Biểu phí/Bảng giá | Cấu trúc giá cho việc sạc điện | 3,500 VND/kWh |

### Thuật ngữ kỹ thuật

| Thuật ngữ | Tiếng Việt | Định nghĩa | Sử dụng |
|-----------|------------|------------|---------|
| **Credentials** | Thông tin xác thực | Thông tin để xác thực giữa 2 hệ thống (token, URL, party_id) | Credentials exchange |
| **Module** | Module chức năng | Một nhóm chức năng trong OCPI (Locations, Sessions, Commands...) | OCPI có 10+ modules |
| **Endpoint** | Điểm kết nối API | URL API để gọi chức năng cụ thể | POST /commands/START_SESSION |
| **Sender** | Bên gửi | Role trong OCPI - bên chủ động gửi dữ liệu | CPO gửi Locations đến Hub |
| **Receiver** | Bên nhận | Role trong OCPI - bên nhận và xử lý requests | CPO nhận Commands từ Hub |
| **Party ID** | Mã định danh | Mã nhận diện duy nhất của CPO hoặc eMSP trong mạng lưới | "ABC", "VIN", "XYZ" |
| **Country Code** | Mã quốc gia | Mã ISO 3166-1 alpha-2 của quốc gia | "VN", "TH", "SG" |

### Thuật ngữ Commands

| Thuật ngữ | Tiếng Việt | Định nghĩa | Ví dụ |
|-----------|------------|------------|-------|
| **START_SESSION** | Bắt đầu sạc | Lệnh yêu cầu CPO kích hoạt phiên sạc từ xa | User bấm "Bắt đầu sạc" trên app |
| **STOP_SESSION** | Dừng sạc | Lệnh yêu cầu CPO dừng phiên sạc từ xa | User bấm "Dừng sạc" trên app |
| **RESERVE_NOW** | Đặt chỗ | Lệnh đặt trước một connector | Đặt connector trước 15 phút |
| **UNLOCK_CONNECTOR** | Mở khóa connector | Lệnh mở khóa vật lý của connector khi bị kẹt | Emergency unlock |
| **Command ID** | Mã lệnh | ID unique cho mỗi command request | CMD-789ABC |

### Thuật ngữ Connector Types

| Thuật ngữ | Tên đầy đủ | Mô tả | Phổ biến |
|-----------|-----------|-------|----------|
| **CCS** | Combined Charging System | Chuẩn sạc nhanh DC phổ biến nhất | ✅ Rất phổ biến |
| **Type 2** / **IEC 62196-T2** | Mennekes | Chuẩn sạc AC châu Âu | ✅ Rất phổ biến |
| **CHAdeMO** | CHAdeMO | Chuẩn sạc nhanh DC của Nhật | ⚠️ Ít hơn |
| **CCS2** / **Combo 2** | CCS Type 2 | Type 2 + DC pins | ✅ Rất phổ biến |
| **GB/T** | Guobiao/Chinese Standard | Chuẩn sạc của Trung Quốc | ⚠️ Ít (ở VN) |

### Thuật ngữ Status

| Status | Module | Ý nghĩa | Khi nào |
|--------|--------|---------|---------|
| **AVAILABLE** | EVSE | Sẵn sàng sử dụng | Connector rảnh |
| **CHARGING** | EVSE | Đang sạc | Xe đang sạc |
| **BLOCKED** | EVSE | Bị chặn | Connector bị blocked |
| **OUTOFORDER** | EVSE | Hỏng | Cần bảo trì |
| **RESERVED** | EVSE | Đã đặt chỗ | User đã reserve |
| **ACTIVE** | Session | Đang hoạt động | Phiên sạc đang active |
| **COMPLETED** | Session | Đã hoàn thành | Phiên sạc đã xong |
| **PENDING** | Session | Đang chờ | Chờ xe kết nối |
| **INVALID** | Session | Không hợp lệ | Authorization failed |

### Thuật ngữ Authentication

| Thuật ngữ | Tiếng Việt | Định nghĩa | Ví dụ |
|-----------|------------|------------|-------|
| **TOKEN_A** | Token đăng ký | Token Hub cấp cho CPO lần đầu, dùng để credentials exchange | One-time use |
| **TOKEN_C** | Token vận hành | Token Hub trả về sau credentials exchange, CPO dùng cho tất cả operations | Long-term token |
| **CPO_TOKEN** | Token của CPO | Token CPO tạo và gửi cho Hub, Hub dùng khi gọi vào CPO | CPO self-generated |
| **Bearer Token** | Token xác thực | Format: `Authorization: Token {token_value}` | RFC 6750 |
| **API Key** | Khóa API | Token để authenticate API calls | - |

### Thuật ngữ Tariff

| Thuật ngữ | Tiếng Việt | Định nghĩa | Đơn vị |
|-----------|------------|------------|--------|
| **ENERGY** | Giá theo điện năng | Giá tính theo kWh tiêu thụ | VND/kWh |
| **TIME** | Giá theo thời gian | Giá tính theo thời gian sạc | VND/phút |
| **FLAT** | Phí cố định | Phí cố định mỗi lần sạc | VND/lần |
| **PARKING_TIME** | Phí đỗ xe | Phí đỗ xe trong khi sạc | VND/giờ |
| **Step Size** | Bước nhảy | Đơn vị tính nhỏ nhất | 1 kWh, 60s |

### Thuật ngữ kinh doanh

| Thuật ngữ | Tiếng Việt | Định nghĩa | Giải thích |
|-----------|------------|------------|------------|
| **Roaming Agreement** | Hợp đồng roaming | Thỏa thuận cho phép interoperability giữa các mạng lưới | CPO A ⇄ CPO B |
| **Settlement** | Thanh toán qua lại | Quá trình thanh toán giữa các bên | Monthly settlement |
| **Clearing House** | Trung tâm thanh toán | Đơn vị trung gian xử lý thanh toán | Hub as clearing house |
| **Revenue Sharing** | Chia sẻ doanh thu | Cách chia tiền giữa CPO, eMSP, Hub | 70% CPO, 20% eMSP, 10% Hub |
| **SLA** | Cam kết dịch vụ | Service Level Agreement - Cam kết về chất lượng dịch vụ | 99.5% uptime |

### Viết tắt thường gặp

| Viết tắt | Tiếng Anh | Tiếng Việt |
|----------|-----------|------------|
| **OCPI** | Open Charge Point Interface | Giao thức điểm sạc mở |
| **CPO** | Charge Point Operator | Nhà vận hành trạm sạc |
| **eMSP** | e-Mobility Service Provider | Nhà cung cấp dịch vụ di động điện |
| **EVSE** | Electric Vehicle Supply Equipment | Thiết bị cấp điện cho xe |
| **EV** | Electric Vehicle | Xe điện |
| **CDR** | Charge Detail Record | Bản ghi chi tiết sạc |
| **kWh** | Kilowatt-hour | Kilowatt-giờ (đơn vị điện năng) |
| **kW** | Kilowatt | Kilowatt (đơn vị công suất) |
| **AC** | Alternating Current | Dòng điện xoay chiều |
| **DC** | Direct Current | Dòng điện một chiều |
| **RFID** | Radio-Frequency Identification | Nhận dạng tần số radio |
| **QR** | Quick Response | Mã phản hồi nhanh |
| **API** | Application Programming Interface | Giao diện lập trình ứng dụng |
| **REST** | Representational State Transfer | Kiến trúc API RESTful |
| **JSON** | JavaScript Object Notation | Định dạng dữ liệu JSON |
| **HTTPS** | HTTP Secure | Giao thức HTTP bảo mật |
| **TLS** | Transport Layer Security | Bảo mật tầng truyền tải |
| **ISO** | International Organization for Standardization | Tổ chức tiêu chuẩn quốc tế |
| **UTC** | Coordinated Universal Time | Giờ phối hợp quốc tế |

### Đơn vị đo lường

| Đơn vị | Tên đầy đủ | Ý nghĩa | Ví dụ |
|--------|-----------|---------|-------|
| **kWh** | Kilowatt-hour | Điện năng tiêu thụ | Sạc được 45.5 kWh |
| **kW** | Kilowatt | Công suất sạc | Trạm sạc 50 kW |
| **A** | Ampere | Cường độ dòng điện | Max 32A |
| **V** | Volt | Điện áp | 400V AC, 500V DC |
| **VND** | Vietnamese Dong | Đồng Việt Nam | 3,500 VND/kWh |
| **s** | Second | Giây | Response < 30s |

### Các trạng thái quan trọng cần nhớ

#### EVSE Status Flow
```
PLANNED → AVAILABLE → CHARGING → AVAILABLE
                   ↓
              OUTOFORDER → (repair) → AVAILABLE
                   ↓
              REMOVED
```

#### Session Status Flow
```
(START_SESSION command)
         ↓
    PENDING → ACTIVE → COMPLETED
         ↓              ↓
     INVALID       (send CDR)
```

---

## 1. Tổng quan

### 1.1 Giới thiệu
Hub Trung tâm là nền tảng tích hợp cho phép kết nối và quản lý tập trung các mạng lưới CPO (Charge Point Operator) trong hệ sinh thái sạc xe điện. Hệ thống tuân thủ tiêu chuẩn **OCPI 2.2.1** (Open Charge Point Interface), đảm bảo khả năng tương tác và mở rộng cao.

### 1.2 Đối tượng người đọc
- **Nhà quản lý CPO**: Hiểu rõ lợi ích kinh doanh và yêu cầu tổng quan
- **Kỹ sư phát triển**: Nắm bắt yêu cầu kỹ thuật chi tiết để triển khai

### 1.3 Phạm vi áp dụng
Tài liệu này áp dụng cho tất cả các CPO mong muốn kết nối vào Hub Trung tâm để:
- Chia sẻ dữ liệu trạm sạc và điểm sạc
- Tham gia vào mạng lưới roaming toàn quốc/khu vực
- Tối ưu hóa vận hành và tăng doanh thu

---

## 2. Lợi ích khi kết nối

### 2.1 Lợi ích kinh doanh

```mermaid
mindmap
  root((Hub Trung tâm))
    Mở rộng mạng lưới
      Tiếp cận hàng triệu người dùng
      Tăng 40-60% tỷ lệ sử dụng trạm sạc
      Mở rộng thị phần
    Tối ưu doanh thu
      Giảm 30% chi phí vận hành
      Tăng 50% hiệu suất sử dụng
      Thanh toán tự động và nhanh chóng
    Quản lý hiệu quả
      Dashboard tập trung
      Phân tích dữ liệu real-time
      Báo cáo tự động
    Tiêu chuẩn hóa
      Tuân thủ OCPI 2.2.1
      Tương thích đa nền tảng
      Giảm complexity kỹ thuật
```

### 2.2 Bảng so sánh lợi ích

| Lợi ích | Trước khi kết nối | Sau khi kết nối | Cải thiện |
|---------|-------------------|-----------------|-----------|
| **Phạm vi khách hàng** | Chỉ người dùng riêng của CPO | Toàn bộ mạng lưới Hub | +300-500% |
| **Tỷ lệ sử dụng trạm** | 30-40% | 60-80% | +100% |
| **Chi phí vận hành** | 100% | 70% | -30% |
| **Thời gian settlement** | 30-45 ngày | 7-14 ngày | -70% |
| **Số lượng tích hợp** | N integrations | 1 integration | Giảm N-1 |
| **Độ phủ dữ liệu** | Hạn chế | Toàn diện | +200% |

---

## 3. Kiến trúc tổng thể

### 3.1 Sơ đồ kiến trúc hệ thống

![Kiến trúc hệ thống](Overview.jpeg)


```mermaid
graph TD
    %% CPO Networks Layer
    CPO[CPO Networks<br/>CPO 1, CPO 2, CPO 3, ... CPO N]
    
    %% API Gateway Layer
    API[API Gateway<br/>Authentication & Rate Limiting]
    
    %% OCPI Server Layer
    OCPI[OCPI 2.2.1 Server<br/>Protocol Handler]
    
    %% Core Services Layer - Stacked Vertically
    CORE_HEADER[CORE SERVICES]
    LOC[📍 Locations Service<br/>Quản lý thông tin trạm sạc]
    TARIFF[💰 Tariffs Service<br/>Quản lý biểu phí]
    SESSION[🔌 Sessions Service<br/>Quản lý phiên sạc]
    CDR[📊 CDR Service<br/>Quản lý bản ghi sạc]
    TOKEN[🎫 Tokens Service<br/>Quản lý token xác thực]
    CMD[⚡ Commands Service<br/>Xử lý lệnh điều khiển]
    
    %% Business Logic Layer - Stacked Vertically
    BUSINESS_HEADER[BUSINESS LOGIC]
    ROUTE[🔀 Smart Routing<br/>Định tuyến thông minh]
    PRICING[💵 Dynamic Pricing<br/>Định giá động]
    ANALYTICS[📈 Analytics Engine<br/>Phân tích & báo cáo]
    
    %% Data Layer
    DATA_HEADER[DATA LAYER]
    DB[(Central Database<br/>PostgreSQL/MySQL)]
    CACHE[(Redis Cache<br/>High Performance)]
    
    %% eMSP Layer
    EMSP[eMSP Networks<br/>eMSP 1, eMSP 2, ... eMSP N]
    
    %% Main Flow
    CPO -->|HTTPS/JSON<br/>OCPI 2.2.1| API
    API --> OCPI
    
    %% OCPI to Services Header
    OCPI --> CORE_HEADER
    
    %% Services in Vertical Stack
    CORE_HEADER --> LOC
    LOC --> TARIFF
    TARIFF --> SESSION
    SESSION --> CDR
    CDR --> TOKEN
    TOKEN --> CMD
    
    %% Business Logic Flow
    OCPI --> BUSINESS_HEADER
    BUSINESS_HEADER --> ROUTE
    ROUTE --> PRICING
    PRICING --> ANALYTICS
    
    %% Services to Data Layer
    CMD --> DATA_HEADER
    ANALYTICS --> DATA_HEADER
    DATA_HEADER --> DB
    DATA_HEADER --> CACHE
    
    %% OCPI to eMSP
    OCPI -->|Forward Requests| EMSP

    %% Styling
    style CPO fill:#e6e6ff,stroke:#6666cc,stroke-width:2px
    style API fill:#ff9900,stroke:#cc7700,stroke-width:3px
    style OCPI fill:#00cc66,stroke:#009944,stroke-width:3px
    
    style CORE_HEADER fill:#cce6ff,stroke:#0066cc,stroke-width:2px,stroke-dasharray: 5 5
    style LOC fill:#e6f3ff,stroke:#0066cc,stroke-width:2px
    style TARIFF fill:#e6f3ff,stroke:#0066cc,stroke-width:2px
    style SESSION fill:#e6f3ff,stroke:#0066cc,stroke-width:2px
    style CDR fill:#e6f3ff,stroke:#0066cc,stroke-width:2px
    style TOKEN fill:#e6f3ff,stroke:#0066cc,stroke-width:2px
    style CMD fill:#e6f3ff,stroke:#0066cc,stroke-width:2px
    
    style BUSINESS_HEADER fill:#ffe6cc,stroke:#ff9900,stroke-width:2px,stroke-dasharray: 5 5
    style ROUTE fill:#fff0e6,stroke:#ff9900,stroke-width:2px
    style PRICING fill:#fff0e6,stroke:#ff9900,stroke-width:2px
    style ANALYTICS fill:#fff0e6,stroke:#ff9900,stroke-width:2px
    
    style DATA_HEADER fill:#cce0ff,stroke:#0066cc,stroke-width:2px,stroke-dasharray: 5 5
    style DB fill:#0066cc,stroke:#004499,stroke-width:3px,color:#fff
    style CACHE fill:#0066cc,stroke:#004499,stroke-width:3px,color:#fff
    
    style EMSP fill:#e6ffe6,stroke:#009944,stroke-width:2px
```

### 3.2 Mô hình triển khai

```mermaid
graph LR
    subgraph "CPO Infrastructure"
        CMS[Charging Management<br/>System]
        CPDB[(CPO Database)]
        CMS --> CPDB
    end
    
    subgraph "Integration Layer"
        ADAPTER[OCPI Adapter]
        MAPPER[Data Mapper]
        ADAPTER --> MAPPER
    end
    
    subgraph "Hub Platform"
        HUB[Hub API<br/>OCPI 2.2.1]
    end
    
    CMS -->|API Calls| ADAPTER
    MAPPER -->|HTTPS<br/>TLS 1.2+| HUB
    
    style ADAPTER fill:#ffcc00
    style HUB fill:#00cc66
```

---

## 4. Luồng kết nối

### 4.1 Quy trình kết nối tổng quan

```mermaid
sequenceDiagram
    autonumber
    participant CPO as CPO System
    participant HUB as Hub Trung tâm
    participant DB as Database
    
    Note over CPO,DB: PHASE 1: Onboarding & Registration
    CPO->>HUB: Đăng ký kết nối (offline)
    HUB->>CPO: Cấp TOKEN_A và versions endpoint URL
    
    Note over CPO,DB: PHASE 2: Version Negotiation
    CPO->>HUB: GET /versions (TOKEN_A)
    HUB->>CPO: Return supported versions
    CPO->>HUB: GET /2.2.1 (TOKEN_A)
    HUB->>CPO: Return all module endpoints
    
    Note over CPO,DB: PHASE 3: Credentials Exchange
    CPO->>HUB: POST /credentials (TOKEN_A)
    Note over CPO,HUB: CPO gửi token riêng của mình cho Hub
    HUB->>DB: Store CPO credentials
    HUB->>CPO: Return Hub credentials (TOKEN_C)
    Note over CPO: CPO lưu TOKEN_C để dùng cho tất cả operations
    
    Note over CPO,DB: PHASE 4: Data Synchronization
    CPO->>HUB: PUT /locations (TOKEN_C)
    HUB->>DB: Store locations
    CPO->>HUB: PUT /tariffs (TOKEN_C)
    HUB->>DB: Store tariffs
    
    Note over CPO,DB: PHASE 5: Live Operations
    CPO->>HUB: Real-time updates (PATCH) với TOKEN_C
    HUB->>CPO: Acknowledgment
    HUB->>CPO: Commands (POST) với CPO_TOKEN
    CPO->>HUB: Command responses
```

---

### 4.1.1 Giải thích chi tiết về các loại TOKEN

Trong quy trình kết nối OCPI, có **2 loại TOKEN chính** được sử dụng:

#### **TOKEN_A - Registration Token** (Token đăng ký ban đầu)

**Đặc điểm**:
- 🔑 **Ai cấp**: Hub cấp cho CPO trong quá trình onboarding (offline)
- 🎯 **Mục đích**: Chỉ dùng cho **lần kết nối đầu tiên** để:
  - GET /versions
  - GET /2.2.1 (version details)
  - POST /credentials (credentials exchange)
- ⏰ **Thời gian sống**: Thường là **one-time use** hoặc có thời hạn ngắn (1-7 ngày)
- 🔒 **Quyền hạn**: Giới hạn - chỉ được phép truy cập endpoints liên quan đến registration

**Ví dụ TOKEN_A**:
```
Authorization: Token a1b2c3d4-e5f6-4789-abcd-1234567890ab
```

**Sử dụng**:
```http
GET /ocpi/versions HTTP/1.1
Host: hub.example.com
Authorization: Token a1b2c3d4-e5f6-4789-abcd-1234567890ab
```

---

#### **TOKEN_C - Operational Token** (Token vận hành chính)

**Đặc điểm**:
- 🔑 **Ai cấp**: Hub trả về cho CPO sau khi POST /credentials thành công
- 🎯 **Mục đích**: Token chính thức để **tất cả** giao tiếp OCPI sau khi đã kết nối
- ⏰ **Thời gian sống**: Dài hạn (90 ngày, 1 năm, hoặc không giới hạn)
- 🔒 **Quyền hạn**: Full access tới tất cả OCPI modules

**Ví dụ TOKEN_C**:
```
Authorization: Token c1111111-2222-3333-4444-555555555555
```

**Sử dụng cho TẤT CẢ operations sau đó**:
```http
# Locations Module
PUT /ocpi/sender/2.2.1/locations/VN/CPO/LOC-001 HTTP/1.1
Authorization: Token c1111111-2222-3333-4444-555555555555

# Sessions Module
PUT /ocpi/sender/2.2.1/sessions/VN/CPO/SESSION-123 HTTP/1.1
Authorization: Token c1111111-2222-3333-4444-555555555555

# CDRs Module
POST /ocpi/sender/2.2.1/cdrs HTTP/1.1
Authorization: Token c1111111-2222-3333-4444-555555555555
```

---

#### **CPO_TOKEN - CPO's Own Token** (Token của CPO)

**Đặc điểm**:
- 🔑 **Ai tạo**: **CPO tự tạo**
- 🎯 **Mục đích**: CPO gửi token này cho Hub trong POST /credentials, Hub sẽ dùng để authenticate khi gọi vào CPO
- 🔄 **Direction**: Hub → CPO (ngược lại với TOKEN_C)
- 🔒 **Quyền hạn**: CPO tự quyết định scope

**Sử dụng** (Hub gửi commands đến CPO):
```http
POST /commands/START_SESSION HTTP/1.1
Host: cpo.example.com
Authorization: Token cpo-secret-token-12345
```

---

### 4.1.2 Bảng so sánh các loại TOKEN

| Thuộc tính | TOKEN_A (Registration) | TOKEN_C (Operational) | CPO_TOKEN |
|------------|------------------------|----------------------|-----------|
| **Người tạo** | Hub | Hub | CPO |
| **Người sử dụng** | CPO → Hub | CPO → Hub | Hub → CPO |
| **Thời gian sống** | 1-7 ngày (one-time) | 90+ ngày | Tùy CPO |
| **Scope** | Version, Credentials | All modules | Commands, Tokens |
| **Khi nào dùng** | Lần đầu kết nối | Tất cả operations | Hub calls CPO |
| **Có thể rotate?** | ❌ No | ✅ Yes | ✅ Yes |

---

### 4.1.3 Luồng chuyển đổi TOKEN

```
PHASE 1: Onboarding (Offline)
┌─────────────────────────────────────┐
│ Hub cấp cho CPO:                    │
│ - TOKEN_A: a1b2c3d4-e5f6-4789...    │
│ - Versions URL                      │
└─────────────────────────────────────┘
           ↓

PHASE 2-3: Credentials Exchange (Online)
┌─────────────────────────────────────┐
│ CPO dùng TOKEN_A để:                │
│ - GET /versions                     │
│ - GET /2.2.1                        │
│ - POST /credentials                 │
│   + Authorization: TOKEN_A          │
│   + Body: {token: "CPO_TOKEN", ...} │
│                                     │
│ Hub response:                       │
│ - Trả về TOKEN_C: c1111111-2222...  │
└─────────────────────────────────────┘
           ↓

PHASE 4+: All Operations (Ongoing)
┌─────────────────────────────────────┐
│ CPO → Hub (dùng TOKEN_C):           │
│ - Locations                         │
│ - Tariffs                           │
│ - Sessions                          │
│ - CDRs                              │
│                                     │
│ Hub → CPO (dùng CPO_TOKEN):         │
│ - Commands (START/STOP)             │
│ - Token validation                  │
│                                     │
│ TOKEN_A không dùng nữa              │
└─────────────────────────────────────┘
```

---

### 4.1.4 TOKEN nào dùng cho module nào?

#### ✅ Phân biệt rõ ràng direction

**CPO → Hub (Sender modules):**

| Module | Endpoint Example | Authorization Header |
|--------|-----------------|---------------------|
| **Locations** | `PUT /locations/VN/CPO/{id}` | `Token TOKEN_C` |
| **Tariffs** | `PUT /tariffs/VN/CPO/{id}` | `Token TOKEN_C` |
| **Sessions** | `PUT /sessions/VN/CPO/{id}` | `Token TOKEN_C` |
| **CDRs** | `POST /cdrs` | `Token TOKEN_C` |

**Hub → CPO (Receiver modules):**

| Module | Endpoint Example | Authorization Header |
|--------|-----------------|---------------------|
| **Commands** | `POST /commands/START_SESSION` | `Token {CPO_TOKEN}` |
| **Tokens** | `POST /tokens/{uid}/authorize` | `Token {CPO_TOKEN}` |

**Lưu ý quan trọng**:
- **CPO → Hub**: Dùng **TOKEN_C** (do Hub cấp sau credentials exchange)
- **Hub → CPO**: Dùng **CPO_TOKEN** (do CPO tạo và gửi cho Hub trong POST /credentials)

---

### 4.1.5 Ví dụ đầy đủ: Credentials Exchange

**Step 1: CPO dùng TOKEN_A**
```http
POST /ocpi/2.2.1/credentials HTTP/1.1
Host: hub.example.com
Authorization: Token a1b2c3d4-e5f6-4789-abcd-1234567890ab
Content-Type: application/json

{
  "token": "cpo-secret-token-12345",
  "url": "https://cpo.example.com/ocpi/hub/",
  "business_details": {
    "name": "ABC CPO"
  },
  "party_id": "ABC",
  "country_code": "VN"
}
```

**Step 2: Hub response với TOKEN_C**
```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "status_code": 1000,
  "data": {
    "token": "c1111111-2222-3333-4444-555555555555",
    "url": "https://hub.example.com/ocpi/cpo/",
    "business_details": {
      "name": "Hub Trung tâm"
    },
    "party_id": "HUB",
    "country_code": "VN"
  }
}
```

**Step 3: CPO cập nhật, dùng TOKEN_C từ giờ**
```http
PUT /ocpi/sender/2.2.1/locations/VN/ABC/LOC-001 HTTP/1.1
Host: hub.example.com
Authorization: Token c1111111-2222-3333-4444-555555555555
Content-Type: application/json

{
  "country_code": "VN",
  "party_id": "ABC",
  "id": "LOC-001",
  ...
}
```

---

### 4.1.6 Token Security Best Practices

**1. Storage**:
- ✅ Lưu TOKEN_C trong database encrypted
- ✅ Không log TOKEN vào file logs
- ✅ Không commit TOKEN vào Git

**2. Rotation**:
- ✅ Rotate TOKEN_C mỗi 90 ngày
- ✅ Sử dụng `PUT /credentials` để update token mới
- ✅ Grace period khi rotate (cho phép cả token cũ và mới hoạt động 24h)

**3. Revocation**:
- ✅ Implement `DELETE /credentials` để revoke kết nối
- ✅ Hub phải invalidate TOKEN ngay lập tức

**Example Token Rotation**:
```http
PUT /ocpi/2.2.1/credentials HTTP/1.1
Host: hub.example.com
Authorization: Token TOKEN_C

{
  "token": "cpo-new-token-67890",
  "url": "https://cpo.example.com/ocpi/hub/",
  "party_id": "ABC",
  "country_code": "VN"
}
```

---

### 4.1.7 Quick Reference: TOKEN Usage Summary

| Scenario | Authorization Header | Token Value | Direction |
|----------|---------------------|-------------|-----------|
| **Registration** | `Authorization: Token TOKEN_A` | Hub cấp | CPO → Hub |
| **Locations** | `Authorization: Token TOKEN_C` | Hub cấp | CPO → Hub |
| **Tariffs** | `Authorization: Token TOKEN_C` | Hub cấp | CPO → Hub |
| **Sessions** | `Authorization: Token TOKEN_C` | Hub cấp | CPO → Hub |
| **CDRs** | `Authorization: Token TOKEN_C` | Hub cấp | CPO → Hub |
| **Commands** | `Authorization: Token CPO_TOKEN` | CPO tạo | Hub → CPO |
| **Token validation** | `Authorization: Token CPO_TOKEN` | CPO tạo | Hub → CPO |

**Công thức đơn giản**:
- 📤 **CPO gửi đến Hub**: Dùng `TOKEN_C`
- 📥 **Hub gửi đến CPO**: Dùng `CPO_TOKEN`
- 🔐 **Lần đầu kết nối**: Dùng `TOKEN_A`

---

### 4.2 Luồng xử lý Session

```mermaid
sequenceDiagram
    autonumber
    participant USER as EV Driver
    participant EMSP as eMSP App
    participant HUB as Hub
    participant CPO as CPO System
    participant CS as Charge Station
    
    USER->>EMSP: Quét QR/Select station
    EMSP->>HUB: POST /commands/START_SESSION
    HUB->>CPO: POST /commands/START_SESSION
    CPO->>CS: Start charging
    CS->>CPO: Charging started
    CPO->>HUB: POST /sessions (new session)
    HUB->>EMSP: Session started notification
    
    loop During Charging
        CS->>CPO: Energy meter updates
        CPO->>HUB: PATCH /sessions/{id}
        HUB->>EMSP: Session update
    end
    
    USER->>EMSP: Stop charging
    EMSP->>HUB: POST /commands/STOP_SESSION
    HUB->>CPO: POST /commands/STOP_SESSION
    CPO->>CS: Stop charging
    CS->>CPO: Charging stopped
    CPO->>HUB: PATCH /sessions/{id} (status=COMPLETED)
    CPO->>HUB: POST /cdrs (CDR record)
    HUB->>EMSP: CDR notification
```

---

## 5. Yêu cầu kỹ thuật

### 5.1 Yêu cầu tối thiểu (MANDATORY)

Các yêu cầu sau là **BẮT BUỘC** để CPO có thể kết nối thành công vào Hub:

| STT | Yêu cầu | Mô tả chi tiết | Tiêu chuẩn | Mức độ |
|-----|---------|----------------|------------|--------|
| 1 | **OCPI 2.2.1 Compliance** | Hỗ trợ đầy đủ OCPI version 2.2.1 | OCPI 2.2.1 Specification | ⭐⭐⭐ CRITICAL |
| 2 | **Credentials Module** | Implement credentials handshake (GET, POST, PUT /credentials) | OCPI §2.1.2 | ⭐⭐⭐ CRITICAL |
| 3 | **Locations Module (Sender)** | Push locations, EVSEs, connectors data | OCPI §4 | ⭐⭐⭐ CRITICAL |
| 4 | **Tariffs Module (Sender)** | Push tariff information | OCPI §5 | ⭐⭐⭐ CRITICAL |
| 5 | **Sessions Module (Sender)** | Push và update session data real-time | OCPI §6 | ⭐⭐⭐ CRITICAL |
| 6 | **CDRs Module (Sender)** | Gửi CDR sau khi session kết thúc | OCPI §7 | ⭐⭐⭐ CRITICAL |
| 7 | **Commands Module (Receiver)** | Nhận và xử lý START_SESSION, STOP_SESSION | OCPI §8 | ⭐⭐⭐ CRITICAL |
| 8 | **TLS/SSL** | HTTPS with TLS 1.2 or higher | RFC 5246 | ⭐⭐⭐ CRITICAL |
| 9 | **Authentication** | Token-based authentication (Bearer token) | OCPI §2.1.3 | ⭐⭐⭐ CRITICAL |
| 10 | **Response Time** | API response < 3 seconds (95th percentile) | SLA | ⭐⭐⭐ CRITICAL |

### 5.2 Yêu cầu khuyến nghị (RECOMMENDED)

| STT | Yêu cầu | Mô tả chi tiết | Lợi ích | Mức độ |
|-----|---------|----------------|---------|--------|
| 11 | **Tokens Module (Receiver)** | Nhận token validation requests | Tăng bảo mật, real-time authorization | ⭐⭐ HIGH |
| 12 | **Commands: RESERVE_NOW** | Hỗ trợ đặt chỗ trước | Tăng trải nghiệm người dùng | ⭐⭐ HIGH |
| 13 | **Commands: UNLOCK_CONNECTOR** | Remote unlock connector | Hỗ trợ khách hàng tốt hơn | ⭐⭐ HIGH |
| 14 | **Webhooks/Callbacks** | Push notifications cho Hub | Giảm polling, real-time updates | ⭐⭐ HIGH |
| 15 | **ChargingProfiles** | Dynamic charging profiles | Smart charging, load management | ⭐⭐ HIGH |
| 16 | **Heartbeat/Health Check** | Endpoint /health hoặc /ping | Monitoring và alerting | ⭐⭐ HIGH |
| 17 | **Idempotency** | Hỗ trợ idempotency keys | Tránh duplicate transactions | ⭐⭐ HIGH |
| 18 | **Rate Limiting** | Xử lý HTTP 429 responses | Tránh overload | ⭐ MEDIUM |

### 5.3 Yêu cầu tùy chọn (OPTIONAL)

| STT | Yêu cầu | Mô tả chi tiết | Lợi ích | Mức độ |
|-----|---------|----------------|---------|--------|
| 19 | **Hub Client Info** | Thông tin về hub platform | Analytics, reporting | ⭐ MEDIUM |
| 20 | **Custom Extensions** | Custom fields trong data objects | Flexibility | ⭐ LOW |
| 21 | **Multi-language Support** | Locations/Tariffs description đa ngôn ngữ | Tăng UX cho thị trường quốc tế | ⭐ LOW |

### 5.4 Bảng modules OCPI 2.2.1

| Module | Role | Endpoint Examples | Mô tả | Mức độ yêu cầu |
|--------|------|-------------------|-------|----------------|
| **Credentials** | Sender & Receiver | GET/POST/PUT /credentials | Trao đổi thông tin xác thực | MANDATORY |
| **Locations** | Sender | PUT/PATCH /locations/{country}/{party}/{id} | Quản lý thông tin trạm sạc | MANDATORY |
| **Tariffs** | Sender | PUT/PATCH /tariffs/{country}/{party}/{id} | Quản lý giá cả | MANDATORY |
| **Sessions** | Sender | PUT/PATCH /sessions/{country}/{party}/{id} | Quản lý phiên sạc | MANDATORY |
| **CDRs** | Sender | POST /cdrs | Gửi bản ghi sạc | MANDATORY |
| **Commands** | Receiver | POST /commands/{command} | Nhận lệnh từ Hub/eMSP | MANDATORY |
| **Tokens** | Receiver | GET/POST /tokens/{country}/{party}/{uid} | Xác thực token | RECOMMENDED |
| **ChargingProfiles** | Receiver | PUT /chargingprofiles/{session_id} | Quản lý profile sạc | RECOMMENDED |
| **HubClientInfo** | Sender | PUT /hubclientinfo | Thông tin client | OPTIONAL |

---

## 5.5 Chi tiết kỹ thuật theo module

### 📋 Tổng quan CPO Role cho các modules

Để CPO dễ dàng hiểu vai trò của mình trong từng module OCPI, bảng dưới đây tóm tắt CPO đóng vai trò **Sender** (gửi data) hay **Receiver** (nhận request):

| Module | CPO Role | Direction | Mô tả | Example Endpoint |
|--------|----------|-----------|-------|------------------|
| **Credentials** | Sender & Receiver | ⇄ Bidirectional | Trao đổi credentials 2 chiều | `POST /credentials` |
| **Locations** | **Sender** | → CPO to Hub | CPO gửi thông tin trạm sạc | `PUT /locations/{id}` |
| **Tariffs** | **Sender** | → CPO to Hub | CPO gửi thông tin giá | `PUT /tariffs/{id}` |
| **Sessions** | **Sender** | → CPO to Hub | CPO gửi thông tin phiên sạc | `PUT /sessions/{id}` |
| **CDRs** | **Sender** | → CPO to Hub | CPO gửi bản ghi thanh toán | `POST /cdrs` |
| **Commands** | **Receiver** | ← Hub to CPO | CPO nhận lệnh từ Hub | `POST /commands/START_SESSION` |
| **Tokens** | **Receiver** | ← Hub to CPO | CPO xác thực token | `POST /tokens/{uid}/authorize` |

**Giải thích**:
- 🟢 **Sender** = CPO **chủ động gửi** data đến Hub (CPO là client, Hub là server)
- 🔵 **Receiver** = CPO **nhận và xử lý** requests từ Hub (CPO là server, Hub là client)
- 🟡 **Sender & Receiver** = Cả 2 chiều (bidirectional)

**Quy tắc Authentication**:
- CPO → Hub: Dùng **TOKEN_C** (Hub cấp cho CPO)
- Hub → CPO: Dùng **CPO_TOKEN** (CPO cung cấp cho Hub trong credentials)

---

### 5.5.1 Credentials Module

**Mục đích**: Trao đổi thông tin xác thực giữa CPO và Hub

**Endpoints cần implement**:

| Method | Endpoint | CPO Role | Mô tả |
|--------|----------|----------|-------|
| GET | /credentials | Receiver | Hub lấy thông tin credentials của CPO |
| POST | /credentials | Receiver | Hub gửi credentials lần đầu |
| PUT | /credentials | Receiver | Cập nhật credentials |
| DELETE | /credentials | Receiver | Ngắt kết nối |

**Request/Response Format**:
```json
{
  "url": "https://cpo.example.com/ocpi/2.2.1/",
  "token": "ebf3b399-779f-4497-9b9d-ac6ad3cc44d2",
  "party_id": "CPO",
  "country_code": "VN",
  "business_details": {
    "name": "Example CPO",
    "website": "https://example-cpo.com"
  }
}
```

**Validation Rules**:
- `country_code`: ISO 3166-1 alpha-2 (VN, TH, SG...)
- `party_id`: 3 characters max, unique identifier
- `token`: UUID format recommended
- `url`: Must be HTTPS

### 5.5.2 Locations Module

**Mục đích**: Quản lý thông tin trạm sạc, EVSE, connector

**Authentication**: Sử dụng **TOKEN_C** cho tất cả requests (CPO → Hub)

**Data Structure**:
```
Location (Trạm sạc)
└── EVSE (Thiết bị sạc)
    └── Connector (Đầu sạc)
```

**Endpoints cần implement**:

| Method | Endpoint | CPO Role | Frequency | Mô tả |
|--------|----------|----------|-----------|-------|
| GET | /locations | Sender | On request | Hub pull toàn bộ locations |
| PUT | /locations/{country}/{party}/{id} | Sender | On create | CPO push location mới |
| PATCH | /locations/{country}/{party}/{id} | Sender | On update | CPO update location |
| PUT | /locations/{country}/{party}/{loc_id}/evses/{evse_id} | Sender | On create | Push EVSE mới |
| PATCH | /locations/{country}/{party}/{loc_id}/evses/{evse_id} | Sender | On update | Update EVSE |
| PUT | /locations/{...}/evses/{evse_id}/connectors/{conn_id} | Sender | On create | Push connector mới |
| PATCH | /locations/{...}/evses/{evse_id}/connectors/{conn_id} | Sender | Real-time | Update connector status |

**Yêu cầu dữ liệu bắt buộc**:

| Field | Type | Required | Validation |
|-------|------|----------|------------|
| id | string(36) | ✅ | Unique, stable identifier |
| name | string(255) | ✅ | Human-readable name |
| address | string(45) | ✅ | Street address |
| city | string(45) | ✅ | City name |
| country | string(3) | ✅ | ISO 3166-1 alpha-3 |
| coordinates | GeoLocation | ✅ | Latitude, Longitude (WGS84) |
| evses | EVSE[] | ✅ | At least 1 EVSE |
| operator | BusinessDetails | ✅ | CPO information |
| time_zone | string(255) | ✅ | IANA timezone (Asia/Ho_Chi_Minh) |

**EVSE Status Values**:
- `AVAILABLE`: Sẵn sàng sử dụng
- `BLOCKED`: Bị chặn
- `CHARGING`: Đang sạc
- `INOPERATIVE`: Không hoạt động
- `OUTOFORDER`: Hỏng
- `PLANNED`: Đang lên kế hoạch
- `REMOVED`: Đã gỡ bỏ
- `RESERVED`: Đã đặt chỗ
- `UNKNOWN`: Không rõ

**Update Frequency**:
- Location info: On change
- EVSE status: Real-time (< 30 seconds delay)
- Connector status: Real-time (< 10 seconds delay)

---

### 5.5.2.1 Quy tắc Uniqueness của ID trong OCPI

**⚠️ QUAN TRỌNG: Hiểu rõ phạm vi unique của các ID**

OCPI sử dụng **composite key** (khóa tổng hợp) để đảm bảo uniqueness trong toàn bộ hệ thống roaming. ID chỉ cần unique trong phạm vi CPO, nhưng kết hợp với `country_code` và `party_id` sẽ tạo ra global uniqueness.

#### Công thức Uniqueness

```
Global Unique ID = country_code + party_id + local_id
```

| ID Type | Phạm vi Unique | Global Identifier | Ví dụ |
|---------|----------------|-------------------|-------|
| **location_id** | Unique trong CPO | `{country}/{party}/{location_id}` | `VN/CPO/LOC-001` |
| **evse_uid** | Unique trong CPO | `{country}/{party}/{location_id}/{evse_uid}` | `VN/CPO/LOC-001/EVSE-A` |
| **connector_id** | Unique trong EVSE | `{country}/{party}/{loc_id}/{evse_uid}/{conn_id}` | `VN/CPO/LOC-001/EVSE-A/1` |
| **session_id** | Unique trong CPO | `{country}/{party}/{session_id}` | `VN/CPO/SESSION-123` |
| **tariff_id** | Unique trong CPO | `{country}/{party}/{tariff_id}` | `VN/CPO/TARIFF-001` |
| **cdr_id** | Unique trong CPO | `{country}/{party}/{cdr_id}` | `VN/CPO/CDR-12345` |

#### Giải thích chi tiết

**1. Location ID** (`location_id`)
- ✅ **Phạm vi**: Unique trong phạm vi **CPO của bạn**
- ❌ **KHÔNG cần**: Unique toàn hệ thống roaming
- 🔑 **Global key**: Được tạo bởi URL path `/{country}/{party}/{location_id}`

**Ví dụ**:
```
CPO A có thể có: location_id = "LOC-001"
CPO B có thể có: location_id = "LOC-001"

Nhưng trong hệ thống roaming:
- CPO A: VN/ABC/LOC-001  → Unique
- CPO B: VN/XYZ/LOC-001  → Unique
```

**2. EVSE UID** (`evse_uid`)
- ✅ **Phạm vi**: Unique trong phạm vi **CPO của bạn**
- ❌ **KHÔNG cần**: Unique toàn hệ thống
- 🔑 **Global key**: `{country}/{party}/{location_id}/{evse_uid}`

**Ví dụ**:
```
CPO A - Location LOC-001 có: evse_uid = "EVSE-A"
CPO A - Location LOC-002 có: evse_uid = "EVSE-A"  ← OK, vì khác location

CPO B - Location LOC-001 có: evse_uid = "EVSE-A"  ← OK, vì khác party_id
```

**3. Connector ID** (`connector_id`)
- ✅ **Phạm vi**: Unique trong phạm vi **EVSE**
- ❌ **KHÔNG cần**: Unique trong CPO hay toàn hệ thống
- 🔑 **Global key**: `{country}/{party}/{location_id}/{evse_uid}/{connector_id}`

**Ví dụ**:
```
EVSE-A có connector_id: "1", "2"
EVSE-B có connector_id: "1", "2"  ← OK, vì khác EVSE
```

#### Best Practices cho CPO

**✅ Khuyến nghị**:

1. **Location ID**: Sử dụng format có ý nghĩa
   - ✅ `HCM-BTX-001` (City-District-Number)
   - ✅ `STATION-001`
   - ✅ `VINCOM-HN-L1`
   - ❌ `1`, `2`, `3` (quá đơn giản, khó trace)

2. **EVSE UID**: Nên unique trong toàn CPO (dù không bắt buộc)
   - ✅ `EVSE-{location}-{number}` → `EVSE-HCM001-A`
   - ✅ Sequential: `EVSE-00001`, `EVSE-00002`
   - ⚠️ `EVSE-A`, `EVSE-B` (OK nhưng nên thêm prefix)

3. **Connector ID**: Đơn giản, clear
   - ✅ `1`, `2` (số thứ tự)
   - ✅ `A`, `B` (chữ cái)
   - ✅ `L` (Left), `R` (Right)

#### Ví dụ thực tế đầy đủ

```json
{
  "country_code": "VN",
  "party_id": "ABC",
  "id": "HCM-BTX-001",           // ← Unique trong CPO ABC
  "evses": [
    {
      "uid": "EVSE-001",         // ← Unique trong CPO ABC (best practice)
      "evse_id": "VN*ABC*E001",  // ← EVSE ID standard (optional)
      "connectors": [
        {
          "id": "1",             // ← Unique trong EVSE-001
          "standard": "IEC_62196_T2"
        },
        {
          "id": "2",             // ← Unique trong EVSE-001
          "standard": "CHADEMO"
        }
      ]
    },
    {
      "uid": "EVSE-002",         // ← Unique trong CPO ABC
      "evse_id": "VN*ABC*E002",
      "connectors": [
        {
          "id": "1",             // ← OK: Unique trong EVSE-002
          "standard": "IEC_62196_T2"
        }
      ]
    }
  ]
}
```

#### Tại sao OCPI thiết kế như vậy?

**Lợi ích của Composite Key**:

1. ✅ **Tự do cho CPO**: Mỗi CPO quản lý ID riêng, không cần phối hợp với CPO khác
2. ✅ **Scalability**: Hàng ngàn CPO có thể tham gia mà không lo conflict
3. ✅ **Simplicity**: ID đơn giản hơn, không cần UUID phức tạp
4. ✅ **Readability**: `VN/ABC/LOC-001` dễ đọc hơn `550e8400-e29b-41d4-a716-446655440000`

**Ví dụ conflict resolution**:

```
Scenario: 2 CPO đều có location_id = "STATION-001"

CPO A (party_id=ABC):
  URL: /locations/VN/ABC/STATION-001
  Global ID: VN/ABC/STATION-001

CPO B (party_id=XYZ):
  URL: /locations/VN/XYZ/STATION-001
  Global ID: VN/XYZ/STATION-001

→ Không conflict vì khác party_id
```

#### Validation Rules

**Hub sẽ validate**:

| Validation | Rule | Lý do |
|------------|------|-------|
| **location_id unique** | Unique trong `{country}/{party}` | Tránh duplicate trong CPO |
| **evse_uid unique** | Unique trong CPO | Best practice OCPI |
| **connector_id unique** | Unique trong EVSE | Tránh confuse |
| **ID format** | Alphanumeric + `-` `_` | URL safe |
| **Max length** | 36 chars | Database optimization |

#### Checklist cho CPO

Khi thiết kế ID schema, đảm bảo:

- [ ] `location_id` unique trong toàn bộ locations của CPO
- [ ] `evse_uid` unique trong toàn bộ EVSEs của CPO (khuyến nghị)
- [ ] `connector_id` unique trong mỗi EVSE
- [ ] `session_id` unique trong toàn bộ sessions của CPO
- [ ] `tariff_id` unique trong toàn bộ tariffs của CPO
- [ ] `cdr_id` unique trong toàn bộ CDRs của CPO
- [ ] Tất cả IDs đều stable (không thay đổi sau khi tạo)
- [ ] Format IDs có ý nghĩa, dễ trace khi troubleshooting

#### So sánh với các hệ thống khác

| Hệ thống | Uniqueness Approach | Example |
|----------|---------------------|---------|
| **OCPI** | Composite key (country + party + id) | `VN/CPO/LOC-001` |
| **OCPP** | ChargePointId unique toàn hệ thống | `VN-CPO-LOC001-EVSE001` |
| **ISO 15118** | EVSE ID global theo ISO | `VN*ABC*E12345` |
| **Database** | UUID toàn cục | `550e8400-e29b-41d4-a716...` |

**OCPI approach cân bằng giữa**:
- ✅ Simplicity (ID ngắn, dễ nhớ)
- ✅ Autonomy (CPO tự quản lý)
- ✅ Uniqueness (Guaranteed bởi composite key)

---

### 5.5.2.2 Sample Requests - Locations Module

Phần này cung cấp các ví dụ request thực tế để CPO dễ dàng tích hợp.

#### Sample 1: Tạo Location mới với 1 EVSE, 2 Connectors

**Request**:
```http
PUT /ocpi/sender/2.2.1/locations/VN/CPO/HCM-BTX-001 HTTP/1.1
Host: hub.example.com
Authorization: Token TOKEN_C
Content-Type: application/json
```
**Lưu ý**: Sử dụng **TOKEN_C** vì đây là CPO gửi data đến Hub (Sender role)

**Body**:
```json
{
  "country_code": "VN",
  "party_id": "CPO",
  "id": "HCM-BTX-001",
  "publish": true,
  "name": "Trạm sạc Bình Thạnh",
  "address": "123 Đường Điện Biên Phủ",
  "city": "Hồ Chí Minh",
  "postal_code": "700000",
  "country": "VNM",
  "coordinates": {
    "latitude": "10.8031",
    "longitude": "106.6867"
  },
  "related_locations": [],
  "parking_type": "PARKING_LOT",
  "evses": [
    {
      "uid": "EVSE-001",
      "evse_id": "VN*CPO*E001",
      "status": "AVAILABLE",
      "capabilities": [
        "REMOTE_START_STOP_CAPABLE",
        "RFID_READER"
      ],
      "connectors": [
        {
          "id": "1",
          "standard": "IEC_62196_T2",
          "format": "SOCKET",
          "power_type": "AC_3_PHASE",
          "max_voltage": 400,
          "max_amperage": 32,
          "max_electric_power": 22000,
          "tariff_ids": ["TARIFF-AC-22KW"],
          "last_updated": "2025-12-28T10:00:00Z"
        },
        {
          "id": "2",
          "standard": "IEC_62196_T2_COMBO",
          "format": "CABLE",
          "power_type": "DC",
          "max_voltage": 500,
          "max_amperage": 100,
          "max_electric_power": 50000,
          "tariff_ids": ["TARIFF-DC-50KW"],
          "last_updated": "2025-12-28T10:00:00Z"
        }
      ],
      "physical_reference": "Cột số 1",
      "floor_level": "0",
      "last_updated": "2025-12-28T10:00:00Z"
    }
  ],
  "directions": [
    {
      "language": "vi",
      "text": "Từ ngã tư Hàng Xanh, đi thẳng 500m về hướng Bắc"
    },
    {
      "language": "en",
      "text": "From Hang Xanh junction, go straight 500m to the North"
    }
  ],
  "operator": {
    "name": "ABC Charge Point Operator"
  },
  "suboperator": {
    "name": "XYZ Facility Management"
  },
  "owner": {
    "name": "Building Owner Co., Ltd"
  },
  "facilities": ["HOTEL", "RESTAURANT", "WIFI"],
  "time_zone": "Asia/Ho_Chi_Minh",
  "opening_times": {
    "twentyfourseven": true
  },
  "charging_when_closed": true,
  "images": [
    {
      "url": "https://cpo.example.com/images/HCM-BTX-001-main.jpg",
      "thumbnail": "https://cpo.example.com/images/HCM-BTX-001-thumb.jpg",
      "category": "LOCATION",
      "type": "jpeg",
      "width": 1920,
      "height": 1080
    }
  ],
  "energy_mix": {
    "is_green_energy": true,
    "energy_sources": [
      {
        "source": "SOLAR",
        "percentage": 30.0
      },
      {
        "source": "WIND",
        "percentage": 20.0
      },
      {
        "source": "GENERAL_FOSSIL",
        "percentage": 50.0
      }
    ]
  },
  "last_updated": "2025-12-28T10:00:00Z"
}
```

**Response (Success)**:
```json
{
  "status_code": 1000,
  "status_message": "Success",
  "timestamp": "2025-12-28T10:00:05Z"
}
```

---

#### Sample 2: Cập nhật trạng thái EVSE (AVAILABLE → CHARGING)

**Request**:
```http
PATCH /ocpi/sender/2.2.1/locations/VN/CPO/HCM-BTX-001/evses/EVSE-001 HTTP/1.1
Host: hub.example.com
Authorization: Token TOKEN_C
Content-Type: application/json
```

**Body**:
```json
{
  "status": "CHARGING",
  "last_updated": "2025-12-28T10:30:15Z"
}
```

---

#### Sample 3: Cập nhật trạng thái Connector

**Request**:
```http
PATCH /ocpi/sender/2.2.1/locations/VN/CPO/HCM-BTX-001/evses/EVSE-001/connectors/1 HTTP/1.1
Host: hub.example.com
Authorization: Token TOKEN_C
Content-Type: application/json
```

**Body**:
```json
{
  "status": "AVAILABLE",
  "last_updated": "2025-12-28T11:45:30Z"
}
```

---

#### Sample 4: Location với nhiều EVSE (trạm sạc lớn)

**Request**:
```http
PUT /ocpi/sender/2.2.1/locations/VN/CPO/HN-CG-001 HTTP/1.1
Host: hub.example.com
Authorization: Token TOKEN_C
Content-Type: application/json
```

**Body**:
```json
{
  "country_code": "VN",
  "party_id": "CPO",
  "id": "HN-CG-001",
  "publish": true,
  "name": "Trạm sạc Cầu Giấy",
  "address": "456 Đường Xuân Thủy",
  "city": "Hà Nội",
  "postal_code": "100000",
  "country": "VNM",
  "coordinates": {
    "latitude": "21.0389",
    "longitude": "105.7899"
  },
  "parking_type": "UNDERGROUND_GARAGE",
  "evses": [
    {
      "uid": "EVSE-A1",
      "evse_id": "VN*CPO*EA1",
      "status": "AVAILABLE",
      "capabilities": ["REMOTE_START_STOP_CAPABLE", "RFID_READER"],
      "connectors": [
        {
          "id": "1",
          "standard": "IEC_62196_T2",
          "format": "SOCKET",
          "power_type": "AC_3_PHASE",
          "max_voltage": 400,
          "max_amperage": 32,
          "max_electric_power": 22000,
          "tariff_ids": ["TARIFF-AC-22KW"],
          "last_updated": "2025-12-28T10:00:00Z"
        }
      ],
      "physical_reference": "Tầng B1 - Vị trí A1",
      "floor_level": "-1",
      "last_updated": "2025-12-28T10:00:00Z"
    },
    {
      "uid": "EVSE-A2",
      "evse_id": "VN*CPO*EA2",
      "status": "AVAILABLE",
      "capabilities": ["REMOTE_START_STOP_CAPABLE", "RFID_READER"],
      "connectors": [
        {
          "id": "1",
          "standard": "IEC_62196_T2",
          "format": "SOCKET",
          "power_type": "AC_3_PHASE",
          "max_voltage": 400,
          "max_amperage": 32,
          "max_electric_power": 22000,
          "tariff_ids": ["TARIFF-AC-22KW"],
          "last_updated": "2025-12-28T10:00:00Z"
        }
      ],
      "physical_reference": "Tầng B1 - Vị trí A2",
      "floor_level": "-1",
      "last_updated": "2025-12-28T10:00:00Z"
    },
    {
      "uid": "EVSE-B1",
      "evse_id": "VN*CPO*EB1",
      "status": "AVAILABLE",
      "capabilities": ["REMOTE_START_STOP_CAPABLE", "CHARGING_PROFILE_CAPABLE"],
      "connectors": [
        {
          "id": "1",
          "standard": "IEC_62196_T2_COMBO",
          "format": "CABLE",
          "power_type": "DC",
          "max_voltage": 920,
          "max_amperage": 250,
          "max_electric_power": 180000,
          "tariff_ids": ["TARIFF-DC-180KW"],
          "last_updated": "2025-12-28T10:00:00Z"
        },
        {
          "id": "2",
          "standard": "CHADEMO",
          "format": "CABLE",
          "power_type": "DC",
          "max_voltage": 500,
          "max_amperage": 125,
          "max_electric_power": 62500,
          "tariff_ids": ["TARIFF-DC-62KW"],
          "last_updated": "2025-12-28T10:00:00Z"
        }
      ],
      "physical_reference": "Tầng B1 - Vị trí B1",
      "floor_level": "-1",
      "last_updated": "2025-12-28T10:00:00Z"
    }
  ],
  "directions": [
    {
      "language": "vi",
      "text": "Vào cổng chính, xuống tầng B1, khu vực A và B"
    }
  ],
  "operator": {
    "name": "ABC Charge Point Operator"
  },
  "facilities": ["MALL", "HOTEL"],
  "time_zone": "Asia/Ho_Chi_Minh",
  "opening_times": {
    "twentyfourseven": true
  },
  "charging_when_closed": true,
  "last_updated": "2025-12-28T10:00:00Z"
}
```

---

#### Sample 5: Cập nhật thông tin Location (thay đổi giờ mở cửa)

**Request**:
```http
PATCH /ocpi/sender/2.2.1/locations/VN/CPO/HCM-BTX-001 HTTP/1.1
Host: hub.example.com
Authorization: Token TOKEN_C
Content-Type: application/json
```

**Body**:
```json
{
  "opening_times": {
    "regular_hours": [
      {
        "weekday": 1,
        "period_begin": "06:00",
        "period_end": "22:00"
      },
      {
        "weekday": 2,
        "period_begin": "06:00",
        "period_end": "22:00"
      },
      {
        "weekday": 3,
        "period_begin": "06:00",
        "period_end": "22:00"
      },
      {
        "weekday": 4,
        "period_begin": "06:00",
        "period_end": "22:00"
      },
      {
        "weekday": 5,
        "period_begin": "06:00",
        "period_end": "22:00"
      },
      {
        "weekday": 6,
        "period_begin": "08:00",
        "period_end": "20:00"
      },
      {
        "weekday": 7,
        "period_begin": "08:00",
        "period_end": "20:00"
      }
    ]
  },
  "charging_when_closed": false,
  "last_updated": "2025-12-28T12:00:00Z"
}
```

---

#### Sample 6: Thêm EVSE mới vào Location hiện có

**Request**:
```http
PUT /ocpi/sender/2.2.1/locations/VN/CPO/HCM-BTX-001/evses/EVSE-002 HTTP/1.1
Host: hub.example.com
Authorization: Token TOKEN_C
Content-Type: application/json
```

**Body**:
```json
{
  "uid": "EVSE-002",
  "evse_id": "VN*CPO*E002",
  "status": "AVAILABLE",
  "capabilities": [
    "REMOTE_START_STOP_CAPABLE",
    "RFID_READER",
    "UNLOCK_CAPABLE"
  ],
  "connectors": [
    {
      "id": "1",
      "standard": "IEC_62196_T2",
      "format": "SOCKET",
      "power_type": "AC_3_PHASE",
      "max_voltage": 400,
      "max_amperage": 16,
      "max_electric_power": 11000,
      "tariff_ids": ["TARIFF-AC-11KW"],
      "last_updated": "2025-12-28T14:00:00Z"
    }
  ],
  "physical_reference": "Cột số 2",
  "floor_level": "0",
  "last_updated": "2025-12-28T14:00:00Z"
}
```

---

#### Sample 7: Đánh dấu EVSE là OUT_OF_ORDER (bảo trì)

**Request**:
```http
PATCH /ocpi/sender/2.2.1/locations/VN/CPO/HCM-BTX-001/evses/EVSE-001 HTTP/1.1
Host: hub.example.com
Authorization: Token TOKEN_C
Content-Type: application/json
```

**Body**:
```json
{
  "status": "OUTOFORDER",
  "status_schedule": [
    {
      "period_begin": "2025-12-28T14:00:00Z",
      "period_end": "2025-12-29T08:00:00Z",
      "status": "OUTOFORDER"
    }
  ],
  "last_updated": "2025-12-28T14:00:00Z"
}
```

---

#### Sample 8: Kiểm tra Location ID Format

**✅ Location ID hợp lệ**:
- `HCM-BTX-001` (Hồ Chí Minh - Bình Thạnh - 001)
- `HN-CG-A01` (Hà Nội - Cầu Giấy - A01)
- `LOC-001`
- `VN-CPO-STATION-123`

**❌ Location ID không nên dùng**:
- `Trạm Bình Thạnh` (có dấu, khoảng trắng)
- `station#123` (ký tự đặc biệt)
- `001` (quá ngắn, không mang ý nghĩa)

**Best Practices**:
- Dùng format: `{City}-{District}-{Number}`
- Chỉ dùng: A-Z, 0-9, dấu gạch ngang `-`, gạch dưới `_`
- Tối đa 36 ký tự
- Nên có ý nghĩa dễ trace (location, area...)

---

### 5.5.3 Tariffs Module

**Authentication**: Sử dụng **TOKEN_C** cho tất cả requests (CPO → Hub)

**Endpoints**:

| Method | Endpoint | CPO Role | Mô tả |
|--------|----------|----------|-------|
| GET | /tariffs | Sender | Hub pull toàn bộ tariffs |
| PUT | /tariffs/{country}/{party}/{id} | Sender | Push/Update tariff |

**Tariff Structure**:
```json
{
  "country_code": "VN",
  "party_id": "CPO",
  "id": "TARIFF-001",
  "currency": "VND",
  "elements": [
    {
      "price_components": [
        {
          "type": "ENERGY",
          "price": 3500.00,
          "step_size": 1
        },
        {
          "type": "TIME",
          "price": 500.00,
          "step_size": 60
        }
      ],
      "restrictions": {
        "start_time": "08:00",
        "end_time": "18:00",
        "day_of_week": ["MONDAY", "TUESDAY", "WEDNESDAY", "THURSDAY", "FRIDAY"]
      }
    }
  ],
  "last_updated": "2025-12-28T10:00:00Z"
}
```

**Price Component Types**:
- `ENERGY`: Giá theo kWh
- `FLAT`: Phí cố định
- `PARKING_TIME`: Phí đỗ xe
- `TIME`: Phí theo thời gian

**Yêu cầu**:
- Currency phải theo ISO 4217 (VND, THB, USD...)
- Price: 2 decimal places
- Update khi có thay đổi giá

### 5.5.4 Sessions Module

**Lifecycle**: START → ACTIVE → COMPLETED

**Authentication**: Sử dụng **TOKEN_C** cho tất cả requests (CPO → Hub)

**Endpoints**:

| Method | Endpoint | CPO Role | Trigger | Max Delay |
|--------|----------|----------|---------|-----------|
| PUT | /sessions/{country}/{party}/{id} | Sender | Session starts | 10 seconds |
| PATCH | /sessions/{country}/{party}/{id} | Sender | Status/meter updates | 30 seconds |
| PATCH | /sessions/{country}/{party}/{id} | Sender | Session ends | 10 seconds |

**Giải thích các tham số URL**:

| Tham số | Mô tả | Ví dụ | Lưu ý |
|---------|-------|-------|-------|
| `{country}` | Mã quốc gia ISO 3166-1 alpha-2 | `VN`, `TH`, `SG` | 2 ký tự viết hoa |
| `{party}` | Party ID của CPO (từ credentials) | `CPO`, `ABC`, `XYZ` | 3 ký tự max |
| `{id}` | **Session ID do CPO tạo** | `SESSION-12345`, `SES-20251228-001` | Unique, stable identifier |

**⚠️ Quan trọng về Session ID**:
- `{id}` là **Session ID** được CPO tự sinh ra khi tạo session mới
- Phải **unique** trong toàn bộ hệ thống CPO
- Phải **stable** (không thay đổi trong suốt vòng đời session)
- Nên có format dễ trace (ví dụ: `SES-{date}-{sequence}`)
- **KHÔNG phải** là location_id, evse_uid hay connector_id

**Ví dụ endpoint đầy đủ**:
```
PUT https://hub.example.com/ocpi/2.2.1/sender/sessions/VN/CPO/SESSION-20251228-103045
                                                              ↑   ↑   ↑
                                                           country party session_id
```

**Session Object**:
```json
{
  "country_code": "VN",
  "party_id": "CPO",
  "id": "SESSION-12345",
  "start_date_time": "2025-12-28T10:30:00Z",
  "kwh": 25.5,
  "currency": "VND",
  "total_cost": 89250.00,
  "status": "ACTIVE",
  "auth_id": "TOKEN-789",
  "location_id": "LOC-001",
  "evse_uid": "EVSE-01",
  "connector_id": "1",
  "meter_id": "METER-001",
  "charging_periods": [
    {
      "start_date_time": "2025-12-28T10:30:00Z",
      "dimensions": [
        {
          "type": "ENERGY",
          "volume": 25.5
        },
        {
          "type": "TIME",
          "volume": 3600
        }
      ]
    }
  ],
  "last_updated": "2025-12-28T11:30:00Z"
}
```

**Update Requirements**:
- **Tần suất cập nhật**: Mỗi **30 giây** trong khi session đang ACTIVE
- **Cost calculation**: Real-time based on tariff
- **Status transitions**: Immediate (< 10 seconds)
- **Energy precision**: 2 decimal places (kWh)

**Session Status Values**:

| Status | Mô tả | Khi nào | Action |
|--------|-------|---------|--------|
| `ACTIVE` | Đang sạc | Charging in progress | CPO update mỗi 30s |
| `COMPLETED` | Hoàn thành | Session ended normally | CPO gửi CDR |
| `INVALID` | Không hợp lệ | Authorization failed | No charging |
| `PENDING` | Đang chờ | Waiting for EV ready | Monitor status |

**Sự kiện CPO cần báo về Hub**:

CPO phải gửi PATCH request đến Hub trong các trường hợp sau:

| Sự kiện | Trigger | Max Delay | Status | Mô tả |
|---------|---------|-----------|--------|-------|
| **Sạc bắt đầu** | Connector connected & charging | 10s | ACTIVE | Xe đã kết nối và bắt đầu sạc |
| **Cập nhật định kỳ** | Mỗi 30 giây | 30s | ACTIVE | Update kWh, cost, duration |
| **Sạc đầy** | Battery 100% or max SoC reached | 10s | COMPLETED | Xe đã sạc đầy |
| **Rút sạc sớm** | User unplugged connector | 10s | COMPLETED | Khách hàng rút vòi trước khi đầy |
| **Lỗi/dừng khẩn cấp** | Error or emergency stop | 5s | COMPLETED | Có lỗi xảy ra, dừng sạc |
| **Timeout** | Session timeout (no activity) | 10s | COMPLETED | Quá thời gian chờ |

**Ví dụ luồng sự kiện Session**:

```mermaid
sequenceDiagram
    participant Hub as Hub
    participant CPO as CPO System
    participant CS as Charge Station
    participant EV as Electric Vehicle
    
    Note over Hub,EV: 1. START SESSION (Hub initiated)
    Hub->>CPO: START_SESSION command
    CPO->>CS: Activate connector
    CS->>EV: Start charging
    CPO->>Hub: PUT /sessions (status: ACTIVE, kwh: 0)
    
    Note over Hub,EV: 2. ACTIVE - Cập nhật mỗi 30s
    loop Every 30 seconds
        CS->>CPO: Meter reading (kwh: 5.2, 10.8, 15.3...)
        CPO->>Hub: PATCH /sessions (kwh, cost, duration)
    end
    
    Note over Hub,EV: 3a. COMPLETED - Sạc đầy
    EV->>CS: Battery full (100%)
    CS->>CPO: Charging completed
    CPO->>Hub: PATCH /sessions (status: COMPLETED, kwh: 45.5)
    CPO->>Hub: POST /cdrs (final CDR)
    
    Note over Hub,EV: 3b. COMPLETED - Rút sạc sớm
    EV->>CS: User unplugged
    CS->>CPO: Session interrupted
    CPO->>Hub: PATCH /sessions (status: COMPLETED, kwh: 30.2)
    CPO->>Hub: POST /cdrs (final CDR)
```

**Ví dụ cập nhật Session mỗi 30s**:

```json
// PATCH /sessions/VN/CPO/SESSION-12345
// Update #1 - After 30s
{
  "kwh": 1.2,
  "total_cost": 4200.00,
  "status": "ACTIVE",
  "last_updated": "2025-12-28T10:30:30Z"
}

// Update #2 - After 60s
{
  "kwh": 2.5,
  "total_cost": 8750.00,
  "status": "ACTIVE",
  "last_updated": "2025-12-28T10:31:00Z"
}

// Update #3 - After 90s
{
  "kwh": 3.8,
  "total_cost": 13300.00,
  "status": "ACTIVE",
  "last_updated": "2025-12-28T10:31:30Z"
}
```

**Ví dụ kết thúc Session - Sạc đầy**:

```json
// PATCH /sessions/VN/CPO/SESSION-12345
{
  "kwh": 45.5,
  "total_cost": 159250.00,
  "status": "COMPLETED",
  "end_date_time": "2025-12-28T12:15:22Z",
  "last_updated": "2025-12-28T12:15:22Z"
}
```

**Ví dụ kết thúc Session - Rút sạc sớm**:

```json
// PATCH /sessions/VN/CPO/SESSION-12345
{
  "kwh": 30.2,
  "total_cost": 105700.00,
  "status": "COMPLETED",
  "end_date_time": "2025-12-28T11:45:18Z",
  "last_updated": "2025-12-28T11:45:18Z"
}
```


### 5.5.5 CDRs Module

**Trigger**: Gửi CDR trong vòng 24h sau khi session kết thúc

**Authentication**: Sử dụng **TOKEN_C** cho tất cả requests (CPO → Hub)

**Endpoint**:

| Method | Endpoint | CPO Role | Trigger | Max Delay |
|--------|----------|----------|---------|-----------|
| POST | /cdrs | Sender | Session completed | 24 hours |

**CDR Object**:
```json
{
  "country_code": "VN",
  "party_id": "CPO",
  "id": "CDR-12345",
  "start_date_time": "2025-12-28T10:30:00Z",
  "end_date_time": "2025-12-28T11:30:00Z",
  "session_id": "SESSION-12345",
  "cdr_token": {
    "uid": "TOKEN-789",
    "type": "RFID",
    "contract_id": "CONTRACT-456"
  },
  "auth_method": "AUTH_REQUEST",
  "location_id": "LOC-001",
  "evse_uid": "EVSE-01",
  "connector_id": "1",
  "meter_id": "METER-001",
  "currency": "VND",
  "total_cost": 89250.00,
  "total_energy": 25.5,
  "total_time": 1.0,
  "charging_periods": [...],
  "remark": "Normal charging session",
  "last_updated": "2025-12-28T11:35:00Z"
}
```

**Validation**:
- CDR ID must be unique
- Total cost must match tariff calculation
- Energy values must be non-negative
- Timestamps must be chronological

### 5.5.6 Commands Module (Receiver)

CPO phải implement các endpoints để nhận commands từ Hub:

**Authentication**: Hub sử dụng **CPO_TOKEN** khi gọi vào CPO (Hub → CPO)

**Endpoints**:

| Command | Endpoint | CPO Role | Priority | Response Time |
|---------|----------|----------|----------|---------------|
| START_SESSION | POST /commands/START_SESSION | Receiver | MANDATORY | < 30 seconds |
| STOP_SESSION | POST /commands/STOP_SESSION | Receiver | MANDATORY | < 30 seconds |
| UNLOCK_CONNECTOR | POST /commands/UNLOCK_CONNECTOR | Receiver | RECOMMENDED | < 30 seconds |
| RESERVE_NOW | POST /commands/RESERVE_NOW | Receiver | RECOMMENDED | < 30 seconds |

**⚠️ Quan trọng về Command ID**:

Command ID là mã định danh unique cho mỗi lệnh được Hub tạo ra và gửi đến CPO.

| Thành phần | Mô tả | Ai tạo | Ví dụ |
|------------|-------|--------|-------|
| **Command ID** | ID unique cho mỗi command request | **Hub tạo** | `CMD-789ABC`, `CMD-20251228-103045` |
| **Response URL** | URL chứa Command ID để CPO gửi kết quả về | **Hub tạo** | `https://hub.com/.../commands/START_SESSION/CMD-789ABC` |
| **Session ID** | ID của session (CPO tạo sau khi start thành công) | **CPO tạo** | `SESSION-67890` |

**Luồng Command ID**:
```
1. Hub tạo Command ID: "CMD-789ABC"
2. Hub gửi START_SESSION với response_url chứa Command ID
3. CPO nhận command, trả về ACCEPTED
4. CPO xử lý command (start charging)
5. CPO gửi kết quả về response_url (dùng Command ID từ Hub)
6. CPO tạo Session ID mới: "SESSION-67890"
7. CPO gửi Session data về Hub với Session ID
```

**Ví dụ minh họa**:
```
Hub Command ID:     CMD-789ABC
  ↓
CPO Response URL:   https://hub.com/ocpi/2.2.1/commands/START_SESSION/CMD-789ABC
  ↓
CPO Session ID:     SESSION-67890 (mới tạo)
```

### 5.5.6.1 START_SESSION Command

**Mục đích**: Hub yêu cầu CPO kích hoạt phiên sạc từ xa

**Tham số bắt buộc**:

| Tham số | Type | Bắt buộc | Mô tả | Ví dụ |
|---------|------|----------|-------|-------|
| `response_url` | URL | ✅ | URL để CPO gửi kết quả async (chứa Command ID do Hub tạo) | `https://hub.com/ocpi/2.2.1/commands/START_SESSION/CMD-789ABC` |
| `token` | Token | ✅ | Token xác thực người dùng | `{"uid": "TOKEN-789", "type": "RFID"}` |
| `location_id` | string(36) | ✅ | ID của trạm sạc | `LOC-001` |
| `evse_uid` | string(36) | ✅ | UID của EVSE | `EVSE-01` |
| `connector_id` | string(36) | ✅ | ID của connector cần kích hoạt | `1` |
| `authorization_reference` | string(36) | ❌ | Reference ID cho authorization | `AUTH-REF-456` |

**START_SESSION Request từ Hub**:
```json
{
  "response_url": "https://hub.example.com/ocpi/2.2.1/commands/START_SESSION/CMD-789ABC",
  "token": {
    "uid": "USER-TOKEN-12345",
    "type": "APP_USER",
    "contract_id": "VN-EMSP-USER001",
    "visual_number": "EMSP-****-1234",
    "issuer": "eMSP Provider A",
    "valid": true,
    "whitelist": "ALLOWED"
  },
  "location_id": "HCM-BTX-001",
  "evse_uid": "EVSE-02",
  "connector_id": "1",
  "authorization_reference": "QR-SCAN-20251228-103045"
}
```

**CPO Response (Immediate)**:
```json
{
  "result": "ACCEPTED",
  "timeout": 30,
  "message": "Command accepted, processing..."
}
```

**CPO Async Result (gửi về response_url sau khi thực thi)**:

CPO gửi POST request đến URL: `https://hub.example.com/ocpi/2.2.1/commands/START_SESSION/CMD-789ABC`

Thành công:
```json
{
  "result": "SUCCESS",
  "message": "Charging session started successfully",
  "session_id": "SESSION-67890",
  "connector_id": "1",
  "timestamp": "2025-12-28T10:30:45Z"
}
```

Thất bại:
```json
{
  "result": "FAILED",
  "message": "Connector is currently in use by another vehicle",
  "timestamp": "2025-12-28T10:30:45Z"
}
```

**Giải thích luồng Command ID và Session ID**:

```
BƯỚC 1 - Hub tạo Command:
  Hub sinh Command ID: "CMD-789ABC"
  Hub tạo response_url: "https://hub.com/.../START_SESSION/CMD-789ABC"
  
BƯỚC 2 - Hub gửi Command đến CPO:
  POST /commands/START_SESSION
  Body: { response_url: "...CMD-789ABC", token: {...}, location_id: "..." }
  
BƯỚC 3 - CPO nhận và trả lời ngay:
  Response: { result: "ACCEPTED", timeout: 30 }
  
BƯỚC 4 - CPO xử lý (0-30 giây):
  - Kích hoạt connector
  - TẠO Session ID mới: "SESSION-67890"
  - Bắt đầu charging
  
BƯỚC 5 - CPO gửi kết quả về Hub:
  POST https://hub.com/.../START_SESSION/CMD-789ABC  ← Dùng Command ID từ response_url
  Body: { result: "SUCCESS", session_id: "SESSION-67890" }  ← Session ID do CPO tạo
  
BƯỚC 6 - CPO gửi Session data:
  PUT /sessions/VN/CPO/SESSION-67890  ← Dùng Session ID
  Body: { status: "ACTIVE", kwh: 0, ... }
```

**Luồng xử lý START_SESSION**:

```mermaid
sequenceDiagram
    participant User as Người dùng
    participant App as eMSP App
    participant Hub as Hub API
    participant CPO as CPO System
    participant CS as Charge Station
    
    User->>App: Quét QR code tại connector
    App->>Hub: Request start charging
    
    Note over Hub: Hub tạo Command ID<br/>CMD-789ABC
    
    Hub->>CPO: POST /commands/START_SESSION
    Note over Hub,CPO: response_url: .../CMD-789ABC<br/>token: USER-TOKEN-12345<br/>location: HCM-BTX-001<br/>evse: EVSE-02<br/>connector: 1
    
    CPO->>Hub: 200 OK - ACCEPTED
    Note over CPO: Timeout: 30s
    
    CPO->>CS: Activate connector #1
    CS-->>CPO: Connector activated
    
    Note over CPO: CPO tạo Session ID<br/>SESSION-67890
    
    CPO->>Hub: POST .../CMD-789ABC (async result)
    Note over CPO,Hub: Result: SUCCESS<br/>session_id: SESSION-67890
    
    CPO->>Hub: PUT /sessions/VN/CPO/SESSION-67890
    Note over CPO,Hub: Status: ACTIVE<br/>kwh: 0<br/>cost: 0
    
    Hub->>App: Session started ✅
    App->>User: Đang sạc...
```

### 5.5.6.2 STOP_SESSION Command

**Mục đích**: Hub yêu cầu CPO dừng phiên sạc từ xa

**Tham số bắt buộc**:

| Tham số | Type | Bắt buộc | Mô tả | Ví dụ |
|---------|------|----------|-------|-------|
| `response_url` | URL | ✅ | URL để CPO gửi kết quả async (chứa Command ID do Hub tạo) | `https://hub.com/ocpi/2.2.1/commands/STOP_SESSION/CMD-DEF123` |
| `session_id` | string(36) | ✅ | ID của session cần dừng (CPO đã tạo trước đó) | `SESSION-67890` |

**STOP_SESSION Request từ Hub**:
```json
{
  "response_url": "https://hub.example.com/ocpi/2.2.1/commands/STOP_SESSION/CMD-DEF123",
  "session_id": "SESSION-67890"
}
```
**Lưu ý**: 
- `CMD-DEF123` là Command ID do **Hub tạo** cho lệnh STOP này
- `SESSION-67890` là Session ID mà **CPO đã tạo** khi START_SESSION thành công trước đó

**CPO Response (Immediate)**:
```json
{
  "result": "ACCEPTED",
  "timeout": 30,
  "message": "Stop command accepted, processing..."
}
```

**CPO Async Result (gửi về response_url)**:

CPO gửi POST request đến URL: `https://hub.example.com/ocpi/2.2.1/commands/STOP_SESSION/CMD-DEF123`

Thành công:
```json
{
  "result": "SUCCESS",
  "message": "Charging session stopped successfully",
  "session_id": "SESSION-67890",
  "final_kwh": 25.8,
  "final_cost": 90300.00,
  "timestamp": "2025-12-28T11:45:22Z"
}
```

Thất bại:
```json
{
  "result": "FAILED",
  "message": "Session not found or already completed",
  "session_id": "SESSION-67890",
  "timestamp": "2025-12-28T11:45:22Z"
}
```

**Luồng xử lý STOP_SESSION**:

```mermaid
sequenceDiagram
    participant User as Người dùng
    participant App as eMSP App
    participant Hub as Hub API
    participant CPO as CPO System
    participant CS as Charge Station
    
    User->>App: Nhấn nút "Dừng sạc"
    App->>Hub: Request stop charging
    
    Note over Hub: Hub tạo Command ID<br/>CMD-DEF123
    
    Hub->>CPO: POST /commands/STOP_SESSION
    Note over Hub,CPO: response_url: .../CMD-DEF123<br/>session_id: SESSION-67890
    
    CPO->>Hub: 200 OK - ACCEPTED
    
    CPO->>CS: Stop charging session
    CS-->>CPO: Charging stopped
    
    CPO->>Hub: POST .../CMD-DEF123 (async result)
    Note over CPO,Hub: Result: SUCCESS<br/>final_kwh: 25.8<br/>final_cost: 90300
    
    CPO->>Hub: PATCH /sessions/VN/CPO/SESSION-67890
    Note over CPO,Hub: Status: COMPLETED<br/>end_date_time: 11:45:22Z
    
    CPO->>Hub: POST /cdrs
    Note over CPO,Hub: Final CDR with total cost
    
    Hub->>App: Session stopped ✅
    App->>User: Hoàn tất sạc<br/>25.8 kWh - 90,300 VND
```

### 5.5.6.3 Tổng hợp Result Codes

| Result Code | HTTP Status | Mô tả | Khi nào xảy ra |
|-------------|-------------|-------|----------------|
| `ACCEPTED` | 200 | Command được chấp nhận | CPO nhận được và sẽ xử lý |
| `REJECTED` | 200 | Command bị từ chối | Connector busy, invalid params |
| `SUCCESS` | N/A | Thực thi thành công | Command completed successfully (gửi về response_url) |
| `FAILED` | N/A | Thực thi thất bại | Error during execution (gửi về response_url) |
| `NOT_SUPPORTED` | 200 | Command không được hỗ trợ | Command type not implemented |
| `UNKNOWN_SESSION` | 200 | Session không tồn tại | Session ID not found |

**Phân biệt Response Types**:

| Response Type | Timing | Mục đích | HTTP Status |
|---------------|--------|----------|-------------|
| **Immediate Response** | Ngay lập tức (< 1s) | Confirm Hub rằng CPO đã nhận được command | 200 OK |
| **Async Result** | Sau khi xử lý (0-30s) | Báo kết quả thực tế sau khi thực thi command | POST đến response_url |

### 5.5.6.4 Error Scenarios và Xử lý

**Scenario 1: Connector đang được sử dụng**
```json
// START_SESSION response
{
  "result": "REJECTED",
  "message": "Connector is currently charging another vehicle (Session: SESSION-11111)"
}
```

**Scenario 2: Token không hợp lệ**
```json
// START_SESSION async result
{
  "result": "FAILED",
  "message": "Token USER-TOKEN-99999 is not authorized for this charging network",
  "timestamp": "2025-12-28T10:30:45Z"
}
```

**Scenario 3: EVSE offline/lỗi**
```json
// START_SESSION async result
{
  "result": "FAILED",
  "message": "EVSE EVSE-02 is currently offline or in error state",
  "timestamp": "2025-12-28T10:30:45Z"
}
```

**Scenario 4: Session đã kết thúc trước khi STOP**
```json
// STOP_SESSION async result
{
  "result": "SUCCESS",
  "message": "Session already completed (user unplugged)",
  "session_id": "SESSION-67890",
  "timestamp": "2025-12-28T11:45:22Z"
}
```

---

## 5.6 Tiêu chuẩn mã QR cho định vị EVSE/Connector

### 5.6.1 Tổng quan

Để ứng dụng eMSP có thể hoạt động liền mạch với tất cả các CPO trong mạng lưới Hub, **mã QR phải được dán tại mỗi connector (vòi sạc)** để người dùng có thể quét và kích hoạt phiên sạc chính xác.

**Lợi ích của tiêu chuẩn QR thống nhất**:
- ✅ Định vị chính xác Location → EVSE → Connector
- ✅ Trải nghiệm người dùng nhất quán trên toàn hệ thống
- ✅ Giảm lỗi kích hoạt sai connector
- ✅ Hỗ trợ offline QR scanning (không cần GPS)
- ✅ Tương thích với tất cả ứng dụng eMSP

### 5.6.2 Cấu trúc mã QR chuẩn

#### Format URI Scheme

CPO **BẮT BUỘC** sử dụng format URI scheme sau cho mã QR:

```
evcharge://{country_code}/{party_id}/{location_id}/{evse_uid}/{connector_id}?params
```

**Ví dụ**:
```
evcharge://VN/CPO/LOC001/EVSE-01/1
evcharge://VN/ABC/HCM-BTX-001/EVSE-02/2?network=hub
```

#### Chi tiết các thành phần

| Thành phần | Mô tả | Format | Ví dụ | Bắt buộc |
|------------|-------|--------|-------|----------|
| `country_code` | Mã quốc gia ISO 3166-1 alpha-2 | 2 ký tự viết hoa | VN, TH, SG | ✅ |
| `party_id` | ID của CPO (OCPI party_id) | 3 ký tự max | CPO, ABC, XYZ | ✅ |
| `location_id` | ID của Location (trạm sạc) | string(36) | LOC001, HCM-BTX-001 | ✅ |
| `evse_uid` | UID của EVSE (thiết bị sạc) | string(36) | EVSE-01, E12345 | ✅ |
| `connector_id` | ID của Connector (vòi sạc) | string(36) | 1, 2, A, B | ✅ |
| `network` | Tên Hub network (optional) | string | hub, icorp | ❌ |

**Lưu ý quan trọng**:
- ✅ Tất cả các ID phải khớp chính xác với dữ liệu trong OCPI Locations Module
- ✅ Sử dụng URL encoding cho các ký tự đặc biệt
- ✅ Case-sensitive (phân biệt chữ hoa/thường)

### 5.6.3 Format thay thế (Alternative)

Nếu CPO có hệ thống QR hiện tại, có thể sử dụng **format HTTPS** như sau:

```
https://hub.example.com/charge?cpo={party_id}&loc={location_id}&evse={evse_uid}&conn={connector_id}
```

**Ví dụ**:
```
https://hub.example.com/charge?cpo=ABC&loc=LOC001&evse=EVSE-01&conn=1
```

### 5.6.4 Sơ đồ luồng kích hoạt bằng QR

```mermaid
sequenceDiagram
    participant User as Người dùng
    participant App as eMSP App
    participant QR as Mã QR
    participant Hub as Hub API
    participant CPO as CPO System
    participant CS as Trạm sạc
    
    User->>QR: Quét mã QR tại connector
    QR->>App: evcharge://VN/CPO/LOC001/EVSE-01/1
    
    Note over App: Parse QR data<br/>Extract: location_id, evse_uid, connector_id
    
    App->>Hub: POST /commands/START_SESSION
    Note over App,Hub: {<br/>"location_id": "LOC001",<br/>"evse_uid": "EVSE-01",<br/>"connector_id": "1",<br/>"token": "USER-TOKEN-123"<br/>}
    
    Hub->>CPO: Forward START_SESSION command
    CPO->>CS: Activate connector #1
    CS->>CPO: Connector activated
    CPO->>Hub: Command SUCCESS
    Hub->>App: Session started
    App->>User: ✅ Đang sạc...
```

### 5.6.5 Yêu cầu triển khai mã QR vật lý

#### Vị trí dán QR

| Vị trí | Yêu cầu | Lý do |
|--------|---------|-------|
| **Tại mỗi connector** | BẮT BUỘC | Định vị chính xác vòi sạc |
| **Tại EVSE** | KHUYẾN NGHỊ | Backup nếu QR tại connector hỏng |
| **Tại bảng chỉ dẫn trạm** | TÙY CHỌN | Hướng dẫn chung |

#### Thông số kỹ thuật vật lý

| Thông số | Giá trị | Mô tả |
|----------|---------|-------|
| **Kích thước QR** | Tối thiểu 5x5 cm | Đảm bảo quét dễ dàng |
| **Chất liệu** | Nhựa PVC, chống thấm nước | Chịu thời tiết outdoor |
| **Độ phân giải** | High (300 DPI+) | Tránh bị mờ |
| **Error correction** | Level H (30%) | Hoạt động khi bị hư hỏng nhẹ |
| **Màu sắc** | QR đen trên nền trắng | Contrast cao |
| **Thông tin kèm** | Connector ID, Power (kW) | Dễ nhận biết |

#### Mẫu thiết kế QR sticker

```
┌──────────────────────────────┐
│  [LOGO CPO]      CONNECTOR 1 │
│                              │
│   ┌─────────────────┐        │
│   │                 │        │
│   │   [QR CODE]     │  50kW  │
│   │                 │  CCS2  │
│   │                 │        │
│   └─────────────────┘        │
│                              │
│ Quét để bắt đầu sạc          │
│ Scan to start charging       │
└──────────────────────────────┘
```

### 5.6.6 QR Code Testing & Validation

CPO phải đảm bảo:

| Kiểm tra | Yêu cầu | Công cụ |
|----------|---------|---------|
| **QR scannable** | 100% QR phải quét được | Smartphone camera, QR apps |
| **Data accuracy** | ID khớp với OCPI data | Manual verification |
| **Durability test** | QR còn đọc được sau 6 tháng outdoor | Field testing |
| **Multiple devices** | Hoạt động trên iOS & Android | Cross-platform testing |

**Công cụ tạo QR Code**:
- https://www.qr-code-generator.com/ (High quality, error correction Level H)
- https://goqr.me/
- Hoặc sử dụng thư viện: `qrcode` (Python), `qr-image` (Node.js)

### 5.6.7 Ví dụ implementation

#### Python - Tạo QR Code
```python
import qrcode
from qrcode.image.styledpil import StyledPilImage

# Tạo QR data theo chuẩn
qr_data = "evcharge://VN/CPO/LOC001/EVSE-01/1"

# Tạo QR với error correction cao
qr = qrcode.QRCode(
    version=1,
    error_correction=qrcode.constants.ERROR_CORRECT_H,  # 30% error tolerance
    box_size=10,
    border=4,
)
qr.add_data(qr_data)
qr.make(fit=True)

# Tạo image
img = qr.make_image(fill_color="black", back_color="white")
img.save(f"connector_qr_EVSE-01_1.png")
```

#### JavaScript - Parse QR Data
```javascript
// Parse QR code scanned từ app
function parseQRCode(qrData) {
  // evcharge://VN/CPO/LOC001/EVSE-01/1
  const regex = /^evcharge:\/\/([A-Z]{2})\/([A-Z0-9]{1,3})\/([^\/]+)\/([^\/]+)\/([^\?]+)/;
  const match = qrData.match(regex);
  
  if (!match) {
    throw new Error('Invalid QR code format');
  }
  
  return {
    country_code: match[1],
    party_id: match[2],
    location_id: match[3],
    evse_uid: match[4],
    connector_id: match[5]
  };
}

// Sử dụng
const scannedData = parseQRCode("evcharge://VN/CPO/LOC001/EVSE-01/1");
// { country_code: 'VN', party_id: 'CPO', location_id: 'LOC001', ... }
```

### 5.6.8 Checklist triển khai QR

CPO cần hoàn thành các bước sau:

- [ ] **Thiết kế QR theo chuẩn URI scheme**
  - [ ] Format: `evcharge://{country}/{party}/{location}/{evse}/{connector}`
  - [ ] Error correction Level H
  - [ ] Kích thước tối thiểu 5x5cm

- [ ] **In và dán QR vật lý**
  - [ ] QR tại mỗi connector (BẮT BUỘC)
  - [ ] Chất liệu chống nước
  - [ ] Kèm thông tin connector (ID, power, type)

- [ ] **Test QR codes**
  - [ ] Quét thử 100% QR codes
  - [ ] Verify data khớp với OCPI Locations
  - [ ] Test với nhiều smartphone khác nhau

- [ ] **Cập nhật documentation**
  - [ ] Cung cấp mapping: Physical location → QR data
  - [ ] Photo documentation các QR đã dán
  - [ ] Quy trình thay thế QR khi hỏng

- [ ] **Maintenance plan**
  - [ ] Kiểm tra QR định kỳ (mỗi 3 tháng)
  - [ ] Quy trình thay thế QR bị mờ/hỏng
  - [ ] Backup QR codes (digital storage)

---

### 5.6.9 Phương án cho CPO đã có QR Code riêng

Đối với CPO đã triển khai mã QR và app eMSP riêng, Hub hỗ trợ 2 phương án tích hợp:

#### Phương án 1: QR Code Mapping API (Khuyến nghị)

CPO giữ nguyên QR code hiện tại, nhưng cung cấp API để Hub/eMSP mapping từ QR code sang OCPI identifiers.

**Yêu cầu CPO cung cấp**:

| Thành phần | Mô tả | Bắt buộc |
|------------|-------|----------|
| **QR Format Documentation** | Tài liệu mô tả cấu trúc QR hiện tại | ✅ |
| **QR Mapping API** | API endpoint để resolve QR → EVSE/Connector | ✅ |
| **QR Code Registry** | Database mapping tất cả QR codes | ✅ |

**API Endpoint yêu cầu**:

```http
POST /api/v1/qr-resolve
Host: cpo.example.com
Authorization: Bearer {CPO_API_KEY}
Content-Type: application/json
```

**Request Body**:
```json
{
  "qr_data": "CPO-ABC-STATION-001-PORT-A",
  "scanned_at": "2025-12-28T14:30:00Z",
  "app_id": "hub-mobile-app"
}
```

**Response Body**:
```json
{
  "status": "success",
  "data": {
    "country_code": "VN",
    "party_id": "ABC",
    "location_id": "STATION-001",
    "evse_uid": "EVSE-A",
    "connector_id": "1",
    "location_name": "Trạm sạc ABC - Chi nhánh 1",
    "connector_type": "IEC_62196_T2",
    "power_kw": 22.0,
    "availability": "AVAILABLE"
  },
  "timestamp": "2025-12-28T14:30:01Z"
}
```

**Error Response**:
```json
{
  "status": "error",
  "error_code": "QR_NOT_FOUND",
  "message": "QR code not found in system",
  "timestamp": "2025-12-28T14:30:01Z"
}
```

**Luồng tích hợp với QR Mapping API**:

```mermaid
sequenceDiagram
    participant User as Người dùng
    participant HubApp as Hub Mobile App
    participant HubAPI as Hub API
    participant CPOAPI as CPO QR API
    participant CPOSys as CPO System
    
    User->>HubApp: Quét QR code CPO
    Note over HubApp: QR data: "CPO-ABC-STATION-001-PORT-A"
    
    HubApp->>HubAPI: Request resolve QR
    HubAPI->>CPOAPI: POST /qr-resolve
    Note over HubAPI,CPOAPI: qr_data: "CPO-ABC-STATION-001-PORT-A"
    
    CPOAPI->>CPOAPI: Lookup QR in registry
    CPOAPI->>HubAPI: Return OCPI identifiers
    Note over CPOAPI,HubAPI: location_id: STATION-001<br/>evse_uid: EVSE-A<br/>connector_id: 1
    
    HubAPI->>HubApp: Return location info
    HubApp->>User: Hiển thị thông tin trạm sạc
    
    User->>HubApp: Nhấn "Bắt đầu sạc"
    HubApp->>HubAPI: Request START_SESSION
    HubAPI->>CPOSys: START_SESSION command
    Note over HubAPI,CPOSys: location_id: STATION-001<br/>evse_uid: EVSE-A<br/>connector_id: 1
    
    CPOSys->>CPOSys: Activate connector
    CPOSys->>HubAPI: Session started
    HubAPI->>HubApp: Success
    HubApp->>User: ✅ Đang sạc
```

**API Specifications chi tiết**:

| Field | Type | Required | Description | Validation |
|-------|------|----------|-------------|------------|
| `qr_data` | string | ✅ | Raw data từ QR code | Max 256 chars |
| `scanned_at` | ISO 8601 | ✅ | Thời gian quét QR | Valid datetime |
| `app_id` | string | ❌ | App identifier | Optional for tracking |
| **Response Fields** ||||
| `country_code` | string(2) | ✅ | ISO 3166-1 alpha-2 | VN, TH, SG |
| `party_id` | string(3) | ✅ | CPO party ID | From OCPI credentials |
| `location_id` | string(36) | ✅ | OCPI location ID | Must exist in Locations |
| `evse_uid` | string(36) | ✅ | OCPI EVSE UID | Must exist in Location |
| `connector_id` | string(36) | ✅ | OCPI connector ID | Must exist in EVSE |

**Performance Requirements**:

| Metric | Target | Measurement |
|--------|--------|-------------|
| **Response Time** | < 500ms (p95) | From request to response |
| **Availability** | 99.9% | Monthly uptime |
| **Rate Limit** | 100 req/sec minimum | Per API key |
| **Error Rate** | < 0.1% | 5xx errors |

---

#### Phương án 2: QR Format Registration

CPO cung cấp documentation về format QR và Hub pre-load mapping configuration.

**Bước 1: CPO cung cấp QR Format Specification**

**Document mẫu**:
```yaml
cpo_qr_format:
  name: "ABC CPO QR Code Format"
  version: "1.0"
  pattern: "{prefix}-{station_code}-{port_id}"
  
  components:
    prefix:
      description: "CPO identifier"
      value: "CPO-ABC"
      fixed: true
    
    station_code:
      description: "Station identifier"
      format: "STATION-{nnn}"
      example: "STATION-001"
      mapping:
        field: "location_id"
        ocpi_format: "{station_code}"
    
    port_id:
      description: "Port/Connector identifier"
      format: "PORT-{A-Z}"
      example: "PORT-A"
      mapping:
        evse_uid: "EVSE-{port_letter}"
        connector_id: "1"
  
  examples:
    - qr_code: "CPO-ABC-STATION-001-PORT-A"
      ocpi_mapping:
        country_code: "VN"
        party_id: "ABC"
        location_id: "STATION-001"
        evse_uid: "EVSE-A"
        connector_id: "1"
    
    - qr_code: "CPO-ABC-STATION-025-PORT-B"
      ocpi_mapping:
        country_code: "VN"
        party_id: "ABC"
        location_id: "STATION-025"
        evse_uid: "EVSE-B"
        connector_id: "1"
```

**Bước 2: CPO cung cấp Complete QR Registry**

**CSV Format**:
```csv
qr_code,country_code,party_id,location_id,evse_uid,connector_id,location_name,power_kw,connector_type
CPO-ABC-STATION-001-PORT-A,VN,ABC,STATION-001,EVSE-A,1,Trạm ABC Chi nhánh 1,22.0,IEC_62196_T2
CPO-ABC-STATION-001-PORT-B,VN,ABC,STATION-001,EVSE-B,1,Trạm ABC Chi nhánh 1,50.0,IEC_62196_T2_COMBO
CPO-ABC-STATION-002-PORT-A,VN,ABC,STATION-002,EVSE-A,1,Trạm ABC Chi nhánh 2,22.0,IEC_62196_T2
```

**JSON Format**:
```json
{
  "qr_registry": [
    {
      "qr_code": "CPO-ABC-STATION-001-PORT-A",
      "ocpi": {
        "country_code": "VN",
        "party_id": "ABC",
        "location_id": "STATION-001",
        "evse_uid": "EVSE-A",
        "connector_id": "1"
      },
      "metadata": {
        "location_name": "Trạm ABC Chi nhánh 1",
        "power_kw": 22.0,
        "connector_type": "IEC_62196_T2",
        "last_updated": "2025-12-28T00:00:00Z"
      }
    }
  ]
}
```

---

#### Phương án 3: Hybrid Approach

CPO sử dụng cả 2 phương pháp:
- **Mã QR mới**: Theo chuẩn Hub `evcharge://` URI scheme
- **Mã QR cũ**: Giữ nguyên, cung cấp QR Mapping API

**Lợi ích**:
- ✅ Không cần thay thế QR code hiện tại ngay lập tức
- ✅ Triển khai QR mới dần dần (khi bảo trì, thêm trạm mới)
- ✅ Hỗ trợ cả app CPO và app Hub seamlessly

---

#### So sánh các phương án

| Tiêu chí | Phương án 1: API | Phương án 2: Registry | Phương án 3: Hybrid |
|----------|------------------|----------------------|---------------------|
| **Chi phí thay QR** | Không cần | Không cần | Thấp (chỉ QR mới) |
| **Effort tích hợp** | Trung bình | Thấp | Cao |
| **Realtime update** | ✅ Yes | ❌ No | ✅ Yes |
| **Offline support** | ❌ No | ✅ Yes (cached) | ✅ Yes |
| **Maintenance** | API uptime | Update registry | Both |
| **Flexibility** | Cao | Thấp | Rất cao |
| **Khuyến nghị** | ⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐⭐ |

---

#### Yêu cầu chung cho tất cả phương án

**1. QR Code Documentation**:
- [ ] Mô tả format QR hiện tại
- [ ] Ví dụ QR codes thực tế
- [ ] Mapping rules sang OCPI identifiers

**2. OCPI Data Consistency**:
- [ ] Tất cả `location_id`, `evse_uid`, `connector_id` trong QR mapping phải tồn tại trong OCPI Locations data
- [ ] Cập nhật mapping khi thêm/xóa EVSE/Connector
- [ ] Validate mapping trong quá trình testing

**3. Testing Requirements**:
- [ ] Test 100% QR codes với Hub mobile app
- [ ] Test fallback scenarios (API down, invalid QR...)
- [ ] Test performance (< 500ms response time)

**4. Documentation & Support**:
- [ ] API documentation (nếu dùng Phương án 1)
- [ ] QR registry file (nếu dùng Phương án 2)
- [ ] Emergency contact khi có issue với QR resolution

---

### 5.6.10 Error Handling cho Legacy QR Codes

**Error Codes chuẩn**:

| Error Code | HTTP Status | Mô tả | Action |
|------------|-------------|-------|--------|
| `QR_NOT_FOUND` | 404 | QR code không tồn tại | Kiểm tra lại QR registry |
| `QR_INVALID_FORMAT` | 400 | Format QR không hợp lệ | Review QR format spec |
| `EVSE_NOT_FOUND` | 404 | EVSE không tồn tại trong OCPI | Sync OCPI Locations data |
| `CONNECTOR_UNAVAILABLE` | 409 | Connector đang bận | Hiển thị status cho user |
| `API_TIMEOUT` | 504 | QR resolve API timeout | Retry với exponential backoff |
| `RATE_LIMIT_EXCEEDED` | 429 | Quá nhiều requests | Implement rate limiting ở client |

**Fallback Strategy**:

```javascript
async function resolveQRCode(qrData) {
  try {
    // Try API resolution first
    const result = await cpoQRApi.resolve(qrData);
    return result;
  } catch (error) {
    if (error.code === 'API_TIMEOUT' || error.code === 'SERVICE_UNAVAILABLE') {
      // Fallback to cached registry
      const cachedResult = await localRegistry.lookup(qrData);
      if (cachedResult) {
        return cachedResult;
      }
    }
    
    // Show user-friendly error
    throw new Error('Không thể định vị trạm sạc. Vui lòng thử lại.');
  }
}
```

---

### 5.7.1 Network & Security

| Yêu cầu | Specification | Lý do |
|---------|---------------|-------|
| **Protocol** | HTTPS only (HTTP forbidden) | Bảo mật dữ liệu |
| **TLS Version** | TLS 1.2+ (TLS 1.3 recommended) | Modern security standards |
| **Certificate** | Valid SSL certificate (not self-signed) | Trust & authentication |
| **Firewall** | Whitelist Hub IPs | Access control |
| **DDoS Protection** | Rate limiting, IP blocking | Service availability |
| **API Keys** | Rotate every 90 days | Security best practice |

**Hub IP Ranges** (để whitelist):
```
Production: 123.45.67.0/24
Staging: 123.45.68.0/24
```

### 5.7.2 Performance Requirements

| Metric | Target | Measurement |
|--------|--------|-------------|
| **API Response Time** | < 3 seconds (95th percentile) | From request to response |
| **Availability** | 99.5% uptime | Monthly average |
| **Throughput** | 100 requests/second minimum | Peak capacity |
| **Data Freshness** | Status updates < 30 seconds | From event to Hub |
| **Error Rate** | < 1% of requests | 5xx errors |

### 5.7.3 Data Format

| Aspect | Specification |
|--------|---------------|
| **Encoding** | UTF-8 |
| **Date/Time** | ISO 8601 (YYYY-MM-DDTHH:mm:ssZ) |
| **Timezone** | UTC for all timestamps |
| **Number Format** | Decimal with dot separator (e.g., 25.5) |
| **Currency** | ISO 4217 codes (VND, USD, EUR...) |
| **Coordinates** | WGS84 decimal degrees |

### 5.7.4 Error Handling

CPO phải xử lý các HTTP status codes từ Hub:

| Status Code | Meaning | Action |
|-------------|---------|--------|
| 200 OK | Success | Continue |
| 201 Created | Resource created | Continue |
| 400 Bad Request | Invalid data | Fix payload and retry |
| 401 Unauthorized | Invalid token | Refresh credentials |
| 403 Forbidden | Access denied | Check permissions |
| 404 Not Found | Resource not found | Check endpoint URL |
| 409 Conflict | Duplicate resource | Check idempotency |
| 429 Too Many Requests | Rate limit exceeded | Implement backoff |
| 500 Server Error | Hub error | Retry with exponential backoff |
| 502/503/504 | Hub unavailable | Retry with exponential backoff |

**Retry Strategy**:
```
Attempt 1: Immediate
Attempt 2: After 1 second
Attempt 3: After 2 seconds
Attempt 4: After 4 seconds
Attempt 5: After 8 seconds
Max attempts: 5
```

### 5.7.5 Logging & Monitoring

CPO nên log các thông tin sau:

| Log Type | Information | Retention |
|----------|-------------|-----------|
| **API Requests** | Method, URL, status code, response time | 30 days |
| **Errors** | Error code, message, stack trace | 90 days |
| **Credentials** | Token rotation events | 365 days |
| **Business Events** | Session start/stop, CDR creation | 365 days |
| **Security** | Authentication failures, suspicious activity | 365 days |

**Monitoring Metrics**:
- API response time (p50, p95, p99)
- Error rate by endpoint
- Request volume
- Session duration
- Energy delivered
- Revenue

---

## 5.8 Testing Requirements

### 5.8.1 Test Environment

Hub cung cấp môi trường test:

| Environment | URL | Purpose |
|-------------|-----|---------|
| **Sandbox** | https://sandbox.hub.example.com | Development & testing |
| **Staging** | https://staging.hub.example.com | Pre-production validation |
| **Production** | https://api.hub.example.com | Live operations |

### 5.8.2 Test Scenarios

CPO phải pass các test scenarios sau:

| # | Scenario | Expected Result | Priority |
|---|----------|-----------------|----------|
| 1 | Credentials handshake | Successful token exchange | CRITICAL |
| 2 | Push 1 location with 2 EVSEs | Locations visible in Hub | CRITICAL |
| 3 | Update EVSE status to CHARGING | Status reflected < 30s | CRITICAL |
| 4 | Push tariff with time-based pricing | Tariff visible in Hub | CRITICAL |
| 5 | Receive START_SESSION command | Charging starts, session created | CRITICAL |
| 6 | Update session every 30 seconds | Energy/cost values accurate | CRITICAL |
| 7 | Receive STOP_SESSION command | Charging stops, CDR generated | CRITICAL |
| 8 | Handle 429 rate limit | Implement backoff retry | HIGH |
| 9 | Token validation request | Return valid/invalid status | HIGH |
| 10 | Reservation (RESERVE_NOW) | Reserve for 15 minutes | MEDIUM |

### 5.8.3 Certification Checklist

Trước khi go-live, CPO phải complete checklist:

**Kỹ thuật OCPI**:
- [ ] All MANDATORY modules implemented
- [ ] Passed all CRITICAL test scenarios
- [ ] SSL certificate installed and valid
- [ ] Monitoring dashboards configured
- [ ] Error handling and retry logic implemented

**QR Code System**:
- [ ] QR codes designed theo chuẩn `evcharge://` URI scheme
- [ ] QR codes in và dán tại 100% connectors
- [ ] Test quét QR thành công trên iOS & Android
- [ ] QR data khớp chính xác với OCPI Locations data
- [ ] Có quy trình bảo trì và thay thế QR

**Vận hành**:
- [ ] Documentation reviewed and understood
- [ ] Support contact designated
- [ ] Incident response plan prepared
- [ ] Data backup strategy in place
- [ ] Performance benchmarks met

---

## 6. Quy trình Onboarding

### 6.1 Các bước triển khai

```mermaid
gantt
    title CPO Onboarding Timeline
    dateFormat  YYYY-MM-DD
    section Registration
    Đăng ký và ký hợp đồng           :a1, 2025-01-01, 7d
    Nhận credentials sandbox         :a2, after a1, 1d
    
    section Development
    Develop OCPI adapter             :b1, after a2, 14d
    Integration testing (Sandbox)    :b2, after b1, 7d
    Fix issues                       :b3, after b2, 7d
    
    section Validation
    UAT trong Staging                :c1, after b3, 7d
    Performance testing              :c2, after c1, 3d
    Security audit                   :c3, after c2, 3d
    
    section Go-live
    Production credentials           :d1, after c3, 1d
    Pilot launch (10% traffic)       :d2, after d1, 7d
    Full production launch           :d3, after d2, 1d
```

**Timeline ước tính**: 6-8 tuần

### 6.2 Checklist theo từng giai đoạn

#### Phase 1: Registration (Week 1)
- [ ] Điền form đăng ký tại: [Link registration form]
- [ ] Ký hợp đồng kết nối (online hoặc offline)
- [ ] Cung cấp thông tin công ty (business_details)
- [ ] Chỉ định technical contact và business contact
- [ ] Nhận credentials cho Sandbox environment

#### Phase 2: Development (Week 2-4)
- [ ] Setup development environment
- [ ] Implement Credentials module
- [ ] Implement Locations module
- [ ] Implement Tariffs module
- [ ] Implement Sessions module
- [ ] Implement CDRs module
- [ ] Implement Commands receiver
- [ ] Unit testing
- [ ] Integration với Sandbox

#### Phase 3: Testing (Week 5-6)
- [ ] Functional testing (all test scenarios)
- [ ] Load testing (100 req/s minimum)
- [ ] Security testing (penetration test)
- [ ] Data accuracy validation
- [ ] Error handling validation
- [ ] Monitoring setup

#### Phase 4: Staging (Week 7)
- [ ] Deploy to Staging environment
- [ ] User Acceptance Testing (UAT)
- [ ] Performance benchmarking
- [ ] Fix critical and high priority bugs
- [ ] Documentation finalization

#### Phase 5: Production (Week 8)
- [ ] Final security review
- [ ] Production credentials issued
- [ ] Pilot launch with limited traffic
- [ ] Monitor metrics for 24-48 hours
- [ ] Full production rollout
- [ ] Post-launch support (24/7 for first week)

---

## 6.3 Tài liệu cần chuẩn bị

| Tài liệu | Nội dung | Người phụ trách |
|----------|----------|-----------------|
| **API Documentation** | Mô tả API endpoints của CPO | Tech Lead |
| **Network Diagram** | Sơ đồ hạ tầng và network | Solution Architect |
| **Data Mapping** | Mapping giữa CPO data model và OCPI | Developer |
| **Runbook** | Hướng dẫn vận hành, troubleshooting | DevOps |
| **Incident Response Plan** | Quy trình xử lý sự cố | Operations Manager |
| **Contact List** | Danh sách liên hệ 24/7 | All |

---

## 7. Hỗ trợ và liên hệ

### 7.1 Kênh hỗ trợ

| Kênh | Mục đích | SLA Response Time |
|------|----------|-------------------|
| **Email** | tech-support@hub.example.com | < 4 hours (business hours) |
| **Hotline** | +84-xxx-xxx-xxx | < 30 minutes (24/7) |
| **Slack Channel** | #cpo-integration | < 1 hour (business hours) |
| **Ticket System** | https://support.hub.example.com | < 2 hours (by priority) |
| **Documentation** | https://docs.hub.example.com | Self-service |

### 7.2 Escalation Matrix

| Level | Role | Contact | Response Time |
|-------|------|---------|---------------|
| L1 | Technical Support | support@hub.example.com | 1 hour |
| L2 | Integration Engineer | integration@hub.example.com | 2 hours |
| L3 | Solution Architect | architect@hub.example.com | 4 hours |
| L4 | CTO | cto@hub.example.com | 8 hours |

### 7.3 SLA & Support Hours

| Tier | Business Hours | After Hours | Severity 1 Response |
|------|----------------|-------------|---------------------|
| **Standard** | Mon-Fri 9-18h | Email only | 4 hours |
| **Premium** | Mon-Fri 8-20h | On-call | 1 hour |
| **Enterprise** | 24/7 | 24/7 | 30 minutes |

**Severity Levels**:
- **Severity 1**: Production down, no workaround
- **Severity 2**: Major function impaired
- **Severity 3**: Minor issue, workaround available
- **Severity 4**: General question, feature request

### 7.4 Tài nguyên tham khảo

| Resource | URL | Description |
|----------|-----|-------------|
| **OCPI 2.2.1 Specification** | https://evroaming.org/app/uploads/2020/06/OCPI-2.2.1.pdf | Official OCPI spec |
| **Hub API Documentation** | https://docs.hub.example.com/api | API reference |
| **Developer Portal** | https://developer.hub.example.com | Integration guides |
| **Postman Collection** | https://hub.example.com/postman | Ready-to-use API calls |
| **GitHub Examples** | https://github.com/hub/examples | Sample code |
| **FAQ** | https://docs.hub.example.com/faq | Common questions |
| **Change Log** | https://docs.hub.example.com/changelog | Platform updates |
| **Status Page** | https://status.hub.example.com | System status |

---

## Appendix A: Glossary

| Term | Definition |
|------|------------|
| **CPO** | Charge Point Operator - Đơn vị vận hành trạm sạc |
| **eMSP** | e-Mobility Service Provider - Nhà cung cấp dịch vụ di động điện |
| **OCPI** | Open Charge Point Interface - Giao thức chuẩn cho roaming |
| **EVSE** | Electric Vehicle Supply Equipment - Thiết bị cung cấp điện cho xe |
| **Connector** | Đầu sạc vật lý |
| **CDR** | Charge Detail Record - Bản ghi chi tiết giao dịch sạc |
| **Session** | Phiên sạc từ khi cắm đến khi rút |
| **Token** | Phương tiện xác thực (RFID card, app...) |
| **Roaming** | Khả năng sử dụng trạm sạc của các mạng lưới khác nhau |
| **Hub** | Trung tâm kết nối các CPO và eMSP |

---

## Appendix B: Sample API Calls

### B.1 GET Versions
```bash
curl -X GET https://api.hub.example.com/ocpi/versions \
  -H "Authorization: Token TOKEN_A"
```

Response:
```json
{
  "data": [
    {
      "version": "2.2.1",
      "url": "https://api.hub.example.com/ocpi/2.2.1"
    }
  ],
  "status_code": 1000,
  "timestamp": "2025-12-28T10:00:00Z"
}
```

### B.2 POST Credentials
```bash
curl -X POST https://api.hub.example.com/ocpi/2.2.1/credentials \
  -H "Authorization: Token TOKEN_A" \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://cpo.example.com/ocpi/2.2.1/",
    "token": "cpo-token-12345",
    "party_id": "CPO",
    "country_code": "VN",
    "business_details": {
      "name": "Example CPO Ltd",
      "website": "https://example-cpo.com"
    }
  }'
```

### B.3 PUT Location
```bash
curl -X PUT https://api.hub.example.com/ocpi/2.2.1/locations/VN/CPO/LOC001 \
  -H "Authorization: Token TOKEN_C" \
  -H "Content-Type: application/json" \
  -d '{
    "id": "LOC001",
    "name": "Central Mall Charging Station",
    "address": "123 Main Street",
    "city": "Ho Chi Minh City",
    "country": "VNM",
    "coordinates": {
      "latitude": "10.7769",
      "longitude": "106.7009"
    },
    "time_zone": "Asia/Ho_Chi_Minh",
    "evses": [...]
  }'
```

---

## Appendix C: Error Codes

| Code | Message | Action |
|------|---------|--------|
| 1000 | Success | - |
| 2000 | Invalid or missing parameters | Check request payload |
| 2001 | Not enough information | Provide all required fields |
| 2002 | Unknown location | Verify location_id |
| 2003 | Unknown token | Verify token uid |
| 3000 | Generic server error | Contact support |
| 3001 | Unable to use client API | Check CPO endpoint availability |
| 3002 | Unsupported version | Use OCPI 2.2.1 |
| 3003 | No matching endpoints | Check module implementation |

---

## Appendix D: Contact Information

**Hub Central Team**

| Role | Name | Email | Phone |
|------|------|-------|-------|
| Integration Manager | [TBD] | integration@hub.example.com | +84-xxx-xxx-xxx |
| Technical Lead | [TBD] | tech@hub.example.com | +84-xxx-xxx-xxx |
| Business Development | [TBD] | bd@hub.example.com | +84-xxx-xxx-xxx |
| Support Team | - | support@hub.example.com | +84-xxx-xxx-xxx |

**Office Address**:
```
Hub Central Office
[Address Line 1]
[Address Line 2]
[City, Country, Postal Code]
```

---

## Document Information

| Field | Value |
|-------|-------|
| **Version** | 1.0 |
| **Last Updated** | 2025-12-28 |
| **Author** | Hub Integration Team |
| **Approved By** | CTO, Head of Operations |
| **Next Review** | 2026-03-28 |
| **Document Status** | ACTIVE |

---

**© 2025 Hub Central. All Rights Reserved.**

*Tài liệu này là tài sản của Hub Central và chỉ dành cho các đối tác CPO được ủy quyền. Nghiêm cấm sao chép hoặc phân phối mà không có sự cho phép.*
