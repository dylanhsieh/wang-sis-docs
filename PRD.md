# WangSIS 產品需求文件 (PRD)

> 版本: 1.0 | 更新日期: 2026-03-12 | 作者: Dylan Xie

---

## 1. 產品概述

WangSIS 是一套專為半導體/電子零件倉儲設計的管理系統，提供 **行動版掃描收貨** 與 **管理端審核庫存** 兩大核心功能。

### 1.1 目標用戶

| 角色 | 說明 | 使用介面 |
|------|------|---------|
| 倉儲人員 | 使用手機掃描標籤、提交收貨紀錄 | 行動版 (m.dylan-reha-2gether.net) |
| 管理員 | 審核收貨、管理帳號、查看報表 | 管理版 (admin.dylan-reha-2gether.net) |

### 1.2 核心價值

- **AI 自動辨識**: 拍照即自動辨識標籤，無需手動輸入
- **供應商智慧匹配**: 根據供應商自動套用專屬解碼規則
- **即時同步**: 行動端提交後，管理端即時可審核
- **成本極低**: 每次掃描僅 $0.0046 NTD

---

## 2. 系統架構

```mermaid
graph TB
    subgraph 使用者端
        M[📱 行動版<br/>倉儲人員]
        A[💻 管理版<br/>管理員]
    end

    subgraph GCP Cloud
        LB[⚖️ 負載平衡器<br/>SSL + 自訂網域]
        FE[🖥️ Cloud Run<br/>前端 React/Nginx]
        BE[⚙️ Cloud Run<br/>後端 FastAPI]
        DB[(🗄️ Cloud SQL<br/>PostgreSQL)]
        AI[🤖 Gemini 2.5 Flash Lite<br/>標籤 OCR]
        SM[🔐 Secret Manager]
    end

    subgraph 外部服務
        SG[📧 SendGrid<br/>密碼重設信]
        CF[🌐 Cloudflare<br/>DNS]
    end

    M -->|HTTPS| CF
    A -->|HTTPS| CF
    CF --> LB
    LB --> FE
    FE --> BE
    BE --> DB
    BE --> AI
    BE --> SG
    BE --> SM
```

---

## 3. 功能規格

### 3.1 標籤掃描收貨流程

```mermaid
sequenceDiagram
    participant U as 📱 倉儲人員
    participant FE as 前端
    participant BE as 後端 API
    participant AI as Gemini AI
    participant DB as 資料庫
    participant Admin as 💻 管理員

    U->>FE: 1. 拍照/選擇標籤圖片
    FE->>BE: 2. POST /api/receiving/scan-label
    BE->>AI: 3. 第一次辨識 (通用prompt)
    AI-->>BE: 4. 回傳: 偵測到供應商 "SEMIHOW"
    BE->>BE: 5. 比對供應商規則庫
    BE->>AI: 6. 第二次辨識 (SEMIHOW專屬prompt)
    AI-->>BE: 7. 回傳: 精確結構化 JSON
    BE-->>FE: 8. 辨識結果
    FE-->>U: 9. 顯示辨識結果供確認

    U->>FE: 10. 確認加入清單
    Note over U,FE: 可重複掃描多張標籤

    U->>FE: 11. 提交收貨
    FE->>BE: 12. POST /api/receiving/records
    FE->>BE: 13. POST /api/receiving/records/{id}/items (每項)
    BE->>DB: 14. 儲存收貨紀錄 + 項目
    BE-->>FE: 15. 收貨單號 RXT260312XXXX

    Note over Admin,DB: 管理端即時可見

    Admin->>BE: 16. GET /api/receiving/admin/records
    BE->>DB: 17. 查詢待審核紀錄
    Admin->>BE: 18. PUT /api/receiving/admin/records/{id}/review
    BE->>DB: 19. 更新狀態: approved/rejected
```

### 3.2 AI 辨識兩階段流程

```mermaid
flowchart LR
    IMG[📷 標籤照片] --> S1[第一階段<br/>通用辨識]
    S1 --> DET{偵測到<br/>已知供應商?}
    DET -->|是| S2[第二階段<br/>供應商專屬辨識]
    DET -->|否| OUT1[輸出通用結果]
    S2 --> OUT2[輸出精確結果]

    style S1 fill:#3b82f6,color:#fff
    style S2 fill:#10b981,color:#fff
    style DET fill:#f59e0b,color:#fff
```

### 3.3 支援供應商與解碼規則

| 供應商 | 類別 | 特殊解碼規則 |
|--------|------|-------------|
| **PANJIT (強茂)** | 二極體 | TYPE: R1=7"Reel, 00001首碼0=無鹵。LOT前4碼=YYMMDD。DC=YYWW |
| **CVILUX (瀚荃)** | 連接器 | LOT: B51G-YYMMDDSN。DC 由 LOT 月份換算 |
| **SYNC (擎力)** | IC/MOSFET | 標準欄位。DATE CODE 可能使用週期碼 |
| **SEMIHOW** | IC | BOX NO 行含兩值：箱號 + LOT/DC碼 (YWWXG格式) |
| **JieJie Micro (捷捷微)** | 微電子 | DC-LOT: YYWW+廠碼。QR含管線分隔資料 |
| **uPI SEMI (力智)** | IC | D/C 使用週期碼表 (A-z對應1-53週) |

### 3.4 收貨紀錄狀態流

```mermaid
stateDiagram-v2
    [*] --> 待審核: 倉儲人員提交
    待審核 --> 已核准: 管理員核准入庫
    待審核 --> 已退回: 管理員退回
    已退回 --> [*]: 需重新掃描

    state 待審核 {
        [*] --> 等待審核
        等待審核 --> 審核中: 管理員檢視
    }
```

---

## 4. 資料模型

```mermaid
erDiagram
    ADMIN ||--o{ RECEIVING_RECORD : "審核"
    MOBILE_USER ||--o{ RECEIVING_RECORD : "掃描"
    RECEIVING_RECORD ||--o{ RECEIVING_ITEM : "包含"

    ADMIN {
        int id PK
        string user_id UK
        string name
        string email
        bool is_active
        bool is_superadmin
    }

    MOBILE_USER {
        int id PK
        string email UK
        string name
        bool is_active
        string created_by
    }

    RECEIVING_RECORD {
        int id PK
        string receiving_no UK
        string supplier
        string invoice_no
        string status
        text notes
        int scanned_by FK
        int reviewed_by FK
        datetime reviewed_at
    }

    RECEIVING_ITEM {
        int id PK
        int receiving_id FK
        string supplier
        string part_no
        string mpn
        string lot
        string dc
        string qty
        int qty_number
        string package_type
        string box_no
        string origin
        json raw_ocr
    }
```

---

## 5. API 端點

### 5.1 行動版 API

| 方法 | 路徑 | 說明 | 認證 |
|------|------|------|------|
| POST | `/api/mobile/auth/login` | 登入 | 無 |
| POST | `/api/mobile/auth/forgot-password` | 忘記密碼 | 無 (IP限速) |
| GET | `/api/mobile/auth/me` | 取得個人資料 | Mobile JWT |
| POST | `/api/receiving/scan-label` | 掃描標籤 (AI辨識) | Mobile JWT |
| POST | `/api/receiving/records` | 建立收貨紀錄 | Mobile JWT |
| POST | `/api/receiving/records/{id}/items` | 新增收貨項目 | Mobile JWT |
| GET | `/api/receiving/my-records` | 我的收貨紀錄 | Mobile JWT |

### 5.2 管理版 API

| 方法 | 路徑 | 說明 | 認證 |
|------|------|------|------|
| POST | `/api/auth/login` | 管理員登入 | 無 |
| GET | `/api/admins/` | 管理員列表 | Admin JWT |
| GET | `/api/mobile-users/` | 行動用戶列表 | Admin JWT |
| GET | `/api/receiving/admin/records` | 所有收貨紀錄 | Admin JWT |
| GET | `/api/receiving/admin/records/{id}` | 收貨詳情 | Admin JWT |
| PUT | `/api/receiving/admin/records/{id}/review` | 審核收貨 | Admin JWT |

---

## 6. 安全機制

| 機制 | 說明 |
|------|------|
| JWT 認證 | HS256 + 24小時過期，admin/mobile 分離 |
| 密碼加密 | bcrypt 雜湊 |
| IP 限速 | 忘記密碼: 5次/分鐘/IP |
| UAT 白名單 | `X-UAT-Token` header 繞過限速 |
| 帳號列舉防護 | 忘記密碼統一回覆，不洩漏帳號是否存在 |
| HTTPS | Google-managed SSL 憑證 |
| Secret Manager | API 金鑰存放 GCP Secret Manager |

---

## 7. 非功能性需求

| 項目 | 規格 |
|------|------|
| 可用性 | 99.9% (Cloud Run SLA) |
| 回應時間 | API < 500ms (不含 AI 辨識) |
| AI 辨識時間 | < 3 秒 (Gemini 2.5 Flash Lite) |
| 支援裝置 | iOS Safari, Android Chrome |
| 語系 | 繁體中文 (zh-TW) |
| 最大圖片 | 10MB |
