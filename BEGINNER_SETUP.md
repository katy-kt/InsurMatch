# InsurMatch

智慧保險比較與主動決策輔助系統 — 透過五步驟問卷，依使用者的預算與需求，產出個人化的健康保險組合建議，並提供商品比較與 AI 保險顧問問答功能。

> **在開始之前請先了解**：這個 repo 只是「前端網站」。要讓它真正跑起來並產出推薦結果，還需要另外啟動一組**後端 AI 服務**（Qdrant 向量資料庫 + LightRAG）。這份 README 會把「前端」和「後端」分成兩大部分，一步一步教你裝。如果你只想看網站畫面、不需要 AI 推薦功能，可以跳過後端部分，但送出問卷後的推薦與 AI 問答會失敗。

## 這是什麼？

使用者填寫一份關於年齡、性別、預算與保障需求的問卷，系統會透過 AI（背後是 RAG 檢索技術）從保單資料中找出合適的方案，整理成「推薦方案」「商品比較表」，並提供一個 AI 顧問讓使用者針對已選保單提問。

## 開始之前：你需要先裝好這些東西

以下皆為**免費**工具，第一次使用也沒關係，照順序裝即可。

| 工具 | 用途 | 下載 / 安裝方式 |
| --- | --- | --- |
| **Git** | 把程式碼抓下來 | [git-scm.com](https://git-scm.com/downloads)，裝好後終端機輸入 `git --version` 應該要有版本號 |
| **Node.js**（建議 LTS 版，20 以上） | 執行前端網站 | [nodejs.org](https://nodejs.org/)，裝好後輸入 `node --version` 與 `npm --version` 應該要有版本號 |
| **Docker Desktop** | 一鍵跑起 Qdrant（向量資料庫） | [docker.com/get-started](https://www.docker.com/products/docker-desktop/)，裝好後輸入 `docker --version` 應該要有版本號 |
| **Python 3.10 以上** | 執行 LightRAG 後端服務 | [python.org](https://www.python.org/downloads/)，裝好後輸入 `python3 --version` 應該要有版本號 |
| **一組 LLM API Key** | LightRAG 需要呼叫語言模型才能運作（例如 OpenAI） | 到對應平台（如 [platform.openai.com](https://platform.openai.com/)）申請 |

> 終端機（Terminal）在哪裡？Windows 用「PowerShell」或「Command Prompt」，macOS 用「終端機 / Terminal」。以下所有指令都是在終端機裡輸入後按 Enter。

---

## 第一步：把專案載下來

```bash
git clone https://github.com/blacktea12TW/InsurMatch.git
cd InsurMatch
```

## 第二步：啟動後端 AI 服務（Qdrant + LightRAG）

這一步的目的，是讓後端能提供一個「OpenAI 相容的 `/v1/chat/completions` API」，前端才能呼叫它拿到推薦結果。

> ⚠️ **請注意**：Qdrant、LightRAG 是獨立於本專案之外的開源工具，並**不包含在這個 repo 裡**。下面的安裝方式是依照 Qdrant / LightRAG 官方文件整理的「標準流程」，但原作者實際用來建立保單資料索引（「testbed」）的腳本或資料集，並未包含在此 repo 中。若你不是原作者本人，建議先向專案負責人確認是否有額外的後端 repo 或資料索引腳本，否則你需要自行準備保單資料並建立索引，以下僅供起步參考。

### 2-1　啟動 Qdrant（向量資料庫）

用 Docker 一行指令即可啟動，不需要額外安裝：

```bash
docker run -p 6333:6333 -p 6334:6334 qdrant/qdrant
```

啟動後保持這個終端機視窗開著；之後需要時再重新執行同一行指令即可（資料預設不會持久化，如需保留資料請加 `-v` 掛載volume，可參考 [Qdrant 官方文件](https://qdrant.tech/documentation/quickstart/)）。

### 2-2　安裝並啟動 LightRAG

打開**新的**終端機視窗：

```bash
pip install "lightrag-hku[api]"
```

設定好你的 LLM API Key（以 OpenAI 為例，macOS/Linux）：

```bash
export OPENAI_API_KEY=你的key
```

Windows（PowerShell）：

```powershell
$env:OPENAI_API_KEY="你的key"
```

接著啟動 LightRAG 伺服器：

```bash
lightrag-server
```

預設會在 `http://127.0.0.1:9621` 啟動（確切 port 請參考啟動時終端機顯示的訊息，或 [LightRAG 官方文件](https://github.com/HKUDS/LightRAG)）。這組網址就是等一下要填入前端 `.env.local` 的 `LLM_API_BASE_URL`。

### 2-3　建立保單資料索引（testbed）

這一步是把保單資料「餵」給 LightRAG，讓它之後能根據這些資料回答問題。**這個步驟只需執行一次**，之後每次開啟前端只要保持 Qdrant 與 LightRAG 伺服器在背景執行即可，不用重跑。

> 這部分依賴原作者的資料與腳本，本 repo 中未附上，請向專案負責人索取，或參考 LightRAG 文件自行建立索引流程。

---

## 第三步：安裝並啟動前端

打開**另一個新的**終端機視窗，回到專案資料夾：

```bash
cd InsurMatch
npm install
```

在專案根目錄建立一個新檔案 `.env.local`，內容如下（依照第 2-2 步 LightRAG 實際啟動的網址調整）：

```bash
LLM_API_BASE_URL=http://127.0.0.1:9621
LLM_API_KEY=你的key（若 LightRAG 沒要求可省略）
```

> `.env.local` 不會被提交到 git，請勿把它分享給別人。`LLM_API_KEY` 只在伺服器端被讀取，不會外洩到瀏覽器。

啟動開發伺服器：

```bash
npm run dev
```

終端機會顯示一組網址（通常是 `http://localhost:3000`），用瀏覽器打開即可看到網站。

---

## 疑難排解

| 現象 | 可能原因 | 解法 |
| --- | --- | --- |
| 網站打得開，但送出問卷後顯示「推薦方案產出失敗」 | 後端服務（Qdrant / LightRAG）沒有在跑，或 `.env.local` 網址設錯 | 確認 Qdrant、LightRAG 兩個終端機視窗都還開著且沒有報錯；確認 `.env.local` 裡的 `LLM_API_BASE_URL` 跟 LightRAG 實際啟動的網址一致 |
| `npm install` 失敗 | Node.js 版本太舊 | 確認 `node --version` 在 20 以上，建議改用 [nvm](https://github.com/nvm-sh/nvm) 管理多版本 |
| `docker run` 出現 port 已被占用的錯誤 | 6333 / 6334 port 被其他程式占用 | 關閉占用該 port 的程式，或改用 `-p 7333:6333` 這類方式改用其他對外 port（需同步調整連線設定） |
| `lightrag-server` 啟動後馬上報錯 | API Key 沒設定，或 Python 版本過舊 | 確認已 `export`/`$env:` 設定好 Key；確認 `python3 --version` 在 3.10 以上 |

---

## 可用指令（前端）

| 指令 | 說明 |
| --- | --- |
| `npm run dev` | 啟動開發伺服器 |
| `npm run build` | 建置正式環境版本 |
| `npm run build:dev` | 以 development 模式建置 |
| `npm run preview` | 預覽建置後的成果 |
| `npm run lint` | 執行 ESLint 檢查 |
| `npm run format` | 使用 Prettier 格式化程式碼 |

## 專案結構（前端）

```text
src/
├── components/
│   ├── insurance/        # 業務元件：問卷、推薦結果、比較矩陣、AI 顧問
│   └── ui/                # shadcn / Radix 基礎 UI 元件
├── data/                  # 保單與方案的靜態資料與型別定義
├── hooks/                 # 自訂 hooks（篩選、配對分數、商品比較等）
├── lib/                   # LLM client、推薦邏輯、工具函式
├── routes/                # 檔案式路由（詳見 src/routes/README.md）
├── store/                 # Zustand 全域狀態
├── types/                 # 共用型別定義
├── server.ts              # 伺服器入口，代理 /api/v1/chat/completions 至後端 RAG 服務
└── router.tsx / start.ts  # TanStack Start 應用程式進入點
```

## 技術棧

- **框架**：[TanStack Start](https://tanstack.com/start)（React 19 + Vite + 檔案式路由 [TanStack Router](https://tanstack.com/router)）
- **狀態管理**：Zustand
- **表單驗證**：react-hook-form + zod
- **UI**：Tailwind CSS v4、Radix UI、shadcn 風格元件
- **後端檢索**：Qdrant（向量資料庫）+ LightRAG（RAG 查詢服務）
