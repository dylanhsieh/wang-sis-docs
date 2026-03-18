# Email Draft — WangSIS POC Demo

**To:** Debby Wang
**Subject:** WangSIS 倉儲管理系統 POC 試用邀請
**Attachments:** PRD.pdf, USER_GUIDE_ADMIN.pdf, USER_GUIDE_MOBILE.pdf, COST.pdf, TEST_GUIDE.pdf

---

Hi Debby，

以下是 WangSIS 倉儲管理系統的 POC 試用資訊，請參考附件文件了解完整功能。

## 系統網址

| 功能 | 網址 |
|------|------|
| 管理後台 | https://admin.dylan-reha-2gether.net |
| 行動版（手機掃描） | https://m.dylan-reha-2gether.net/m/login |
| API 文件 | https://api-admin.dylan-reha-2gether.net/docs |

## 管理後台帳號

| 項目 | 內容 |
|------|------|
| 帳號 | `Debby.wang` |
| 密碼 | `123456` |

登入後可使用以下功能：
- **管理員管理** — 新增/編輯管理員帳號
- **行動用戶** — 建立倉儲人員帳號（供手機掃描用）
- **收貨管理** — 審核倉儲人員提交的掃描收貨紀錄
- **供應商規則** — 管理 AI 辨識的供應商 Prompt 規則 + 測試遊樂場
- **標籤掃描** — 上傳標籤圖片測試 AI 辨識

## 快速測試流程

1. **管理後台登入** → 用上方帳號登入
2. **建立行動用戶** → 行動用戶 → 新增用戶 → 記下密碼
3. **手機掃描測試** → 用手機開啟行動版網址 → 登入 → 掃描收貨 → 選擇供應商 → 拍照
4. **審核收貨** → 回管理後台 → 收貨管理 → 查看掃描結果 → 核准/退回

## AI 辨識說明

- 使用 Google Gemini 2.5 Flash Lite 模型
- 支援 6 家供應商專屬辨識規則（PANJIT、CVILUX、SYNC、SEMIHOW、JIEJIE、UPI）
- 行動版可先選擇供應商加速辨識（約 1.5 秒），或選「自動偵測」（約 3 秒）
- 供應商規則可在管理後台即時新增/修改，不需改程式

## 附件說明

| 文件 | 內容 |
|------|------|
| PRD.pdf | 產品需求文件（架構圖、流程圖、ER 圖） |
| USER_GUIDE_ADMIN.pdf | 管理後台使用指南（含截圖） |
| USER_GUIDE_MOBILE.pdf | 行動版使用指南（含截圖） |
| COST.pdf | 系統成本估算（NTD） |
| TEST_GUIDE.pdf | 測試指南（含 API 範例） |

如有任何問題歡迎隨時聯繫。

Best regards,
Dylan
