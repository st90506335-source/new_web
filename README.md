# 宣德科技全新官方網站 (SpeedTech New Official Website)

宣德科技股份有限公司（SpeedTech Corp.）官方網站全端建置專案，採用 **Nginx 高效靜態閘道 + Node.js 20 (Express + Prisma ORM) 後端 API + PostgreSQL 16 關聯式資料庫** 三層現代化容器架構，並深度整合 **Windows Active Directory (AD) 網域 SSO 單一簽入** 與 **RBAC 角色權限分級管理後台**。

---

## 📌 系統架構概覽 (Architecture)

```text
[ 客戶端瀏覽器 (公眾訪客 / 內部同仁) ]
                 │
                 ▼  Port 80 (HTTP)
┌──────────────────────────────────────────────────────────┐
│                   Nginx 逆向代理與閘道                    │
│                 (speedtech-website 容器)                 │
├────────────────────────────┬─────────────────────────────┤
│   靜態資源與 SPA 直出       │      API 反向代理分流        │
│   • 官網前端靜態頁面 (/)    │      • 轉發至內部 Port 3000   │
│   • 管理後台 SPA (/admin/)  │      • 路由: /api/*          │
│   • 靜態快取與 Gzip 壓縮    │                             │
└─────────────┬──────────────┴──────────────┬──────────────┘
              │                             │
              │                             ▼
              │                ┌───────────────────────────┐
              │                │    Node.js 20 後端 API     │
              │                │   (speedtech-api 容器)    │
              │                ├───────────────────────────┤
              │                │ • Express RESTful API     │
              │                │ • Prisma ORM 資料抽象層   │
              │                │ • JWT Token 簽發與驗證    │
              │                │ • RBAC 角色存取權限控制   │
              │                └──────┬─────────────┬──────┘
              │                       │             │
              ▼                       ▼             ▼
       ┌──────────────┐      ┌────────────────┐  ┌────────────────┐
       │ 靜態 HTML/CSS│      │ PostgreSQL 16  │  │  Windows AD DC │
       │ JS / 圖片 /  │      │ 資料庫持久化   │  │  網域控制器    │
       │ 字型檔案庫   │      │ (speedtech-db) │  │  (LDAP SSO)    │
       └──────────────┘      └────────────────┘  └────────────────┘
```

* **極致輕量化**：整套容器化服務常駐記憶體總佔用僅約 **49 MB**，CPU 負載接近 0%。
* **高可用與安全性**：資料庫與後端 API 均運行於內部隔離 Docker Bridge 網路，僅 Nginx 對外開放 Port 80，PostgreSQL 開放 Port 5432 供內網管理。

---

## 🚀 核心功能與模組

### 1. 官網公眾前台
* **多語系與完整企業形象**：涵蓋公司沿革、營運據點、研發實力與企業永續 (ESG)。
* **全系列產品展示**：
  * 車用高壓連接器 (Automotive)
  * 低軌衛星 (Starlink) 核心連接零組件
  * 雲端網通 (Networking) 與高速高頻傳輸
  * 聲學元器件 (Acoustics)
  * 精密連接器零組件 (Components)
  * ODM / OEM 整合解決方案
* **投資人專區 (IR)**：財務營收、財報公布、法說會、股東會日程、股利資訊與公司治理政策。
* **聯絡我們 (Contact Us)**：訪客在線提交諮詢表單，由後端即時寫入 PostgreSQL 資料庫儲存，防止漏單。

### 2. 管理後台系統 (`/admin/`)
透過 Single Page Application (Vue 3 + Tailwind CSS) 建構，具備直覺高效的管理面板：
* **Windows AD 網域 SSO 單一簽入**：
  * 對接公司核心 AD 網域控制站 (`172.30.36.7:389` / `172.30.36.5:389`，網域 `STECH.COM.TW`)。
  * 員工直接以日常 Windows 登入帳密認證，免重複註冊建號。
  * 支援離線/網路故障時之 **本地緊急管理員 (`admin`)** 後備機制。
* **RBAC 角色基礎權限控管**：
  * **`NEWS_EDITOR` (消息發佈員)**：專屬新聞編輯權限，可建立草稿、排程發佈、編輯或下架最新消息。
  * **`CONTACT_VIEWER` (諮詢管理員)**：負責客服與業務團隊，可查閱訪客諮詢留言、流轉處理狀態（待處理 / 處理中 / 已結案）。
  * **`SUPER_ADMIN` (最高管理員)**：可進行全局管理與所有 AD 帳號之角色權限指派。
  * **`GUEST` (訪客)**：未獲授權之一般 AD 帳號，僅具備後台唯讀瀏覽權限。

---

## 📂 專案目錄結構

```text
website/
├── README.md                      # 專案完整說明文件
├── index.html                     # 官網首頁
├── 404.html                       # 404 找不到頁面
├── admin/                         # [後台管理 SPA]
│   └── index.html                 # 後台單頁應用程式 (Vue 3 + Tailwind CSS)
│
├── [企業形象與資訊]
│   ├── about.html                 # 關於宣德
│   ├── about-History.html         # 公司沿革
│   ├── about-Supplier.html        # 供應商專區
│   ├── recruit-talents.html       # 人才招募
│   ├── welfare-system.html        # 福利制度
│   ├── training-system.html       # 培訓制度
│   └── retirement-system.html     # 退休制度
│
├── [產品與解決方案]
│   ├── product.html               # 產品總覽
│   ├── product-automotive.html    # 車用連接器
│   ├── product-starlink.html      # 低軌衛星解決方案
│   ├── product-networking.html    # 網通產品
│   ├── product-acoustics.html     # 聲學產品
│   ├── product-components.html    # 零組件
│   ├── product-odm.html           # ODM 專區
│   └── product-oem.html           # OEM 專區
│
├── [公司治理與投資人專區]
│   ├── shareholders-Meeting.html  # 股東常會
│   ├── shareholders-Dividend.html # 股利分派
│   ├── financial-Statements.html  # 財務報表
│   ├── financial-Revenues.html    # 每月營收
│   └── ...                        # 各功能委員會與公司治理規程
│
├── [最新消息與客戶服務]
│   ├── news.html                  # 最新消息公開列表 (串接 /api/news)
│   ├── news-Post.html             # 消息內頁文章
│   ├── contact.html               # 聯絡我們 (串接 /api/contact)
│   └── contact-Location.html      # 全球營運據點地圖
│
├── css/                           # 樣式庫 (Bootstrap, Tailwind, 自訂樣式)
├── js/                            # 前端互動邏輯腳本
├── fonts/                         # 品牌專用字型檔
├── img/                           # 產品高清圖、Banner 與圖標
└── source/                        # 靜態原型設計素材
```

---

## ⚙️ 快速運行與部署指南

### 1. 本機純靜態測試 (Windows PowerShell)
本專案內建輕量 HTTP Server 腳本，無須額外安裝伺服器軟體：
```powershell
# 在 website 目錄下執行
powershell -File .claude/serve.ps1
```
啟動後於瀏覽器開啟：`http://localhost:8642/` 即可即時預覽。

---

### 2. 正式伺服器 Docker Compose 容器化部署
伺服器環境（Ubuntu Linux）採用三層容器化部署，包含 API 與 PostgreSQL：

```bash
# 1. 進入伺服器部署目錄
cd /opt/website

# 2. 啟動所有容器 (Nginx + Node.js API + PostgreSQL 16)
sudo docker compose up -d --build

# 3. 檢查容器運行狀態
sudo docker compose ps
```

#### 預設服務端口
* **Web 官網與後台**：`http://<SERVER_IP>/` 與 `http://<SERVER_IP>/admin/`
* **後端 API**：`http://<SERVER_IP>/api/health`
* **PostgreSQL**：`Port 5432`（帳號：`speedtech_user`，資料庫：`speedtech_web`）

---

## 🔐 權限管理與維護說明

### 新增或變更新聞發佈人員流程
1. 使用最高管理員帳號登入後台：`http://<SERVER_IP>/admin/`
2. 切換至「**權限管理**」分頁。
3. 點選「**預先授權 AD 帳號**」。
4. 輸入欲授權同仁的 Windows 工號/帳號（例如：`st90506335`），並勾選 **`NEWS_EDITOR`** 角色。
5. 點選確認儲存後立即生效。該同仁即可使用其日常 Windows 密碼登入後台發佈新聞。

---

## 📄 版權聲明
版權所有 © 2026 宣德科技股份有限公司 SpeedTech Corp. 保留所有權利。
