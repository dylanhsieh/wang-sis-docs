# WangSIS 倉儲管理系統 — 產品文件

瀚碩科技 WMS (Warehouse Management System) 產品需求文件與使用者指南。

## 文件目錄

| 文件 | 說明 |
|------|------|
| [產品需求文件 (PRD)](./PRD.md) | 系統功能規格、架構、流程圖 |
| [使用者指南 — 行動版](./USER_GUIDE_MOBILE.md) | 行動用戶操作手冊 |
| [使用者指南 — 管理版](./USER_GUIDE_ADMIN.md) | 管理員操作手冊 |
| [成本估算](./COST.md) | GCP 基礎建設 + 每次掃描成本 |

## 系統概覽

```
┌─────────────────────────────────────────────────────────┐
│                    WangSIS 倉儲管理系統                    │
├──────────────────────┬──────────────────────────────────┤
│   行動版 (Mobile)     │        管理版 (Admin)             │
│   m.dylan-reha-2gether.net   admin.dylan-reha-2gether.net│
├──────────────────────┴──────────────────────────────────┤
│              API: api-admin.dylan-reha-2gether.net       │
├─────────────────────────────────────────────────────────┤
│   GCP Cloud Run │ Cloud SQL │ Gemini AI │ SendGrid      │
└─────────────────────────────────────────────────────────┘
```

## 技術棧

- **前端**: React 18 + TypeScript + Ant Design 5
- **後端**: Python 3.12 + FastAPI + SQLAlchemy
- **資料庫**: PostgreSQL (GCP Cloud SQL)
- **AI**: Google Gemini 2.5 Flash Lite (標籤 OCR)
- **基礎建設**: GCP Cloud Run, Load Balancer, Secret Manager
- **DNS**: Cloudflare
