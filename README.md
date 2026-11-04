# InsurMatch

智慧保險比較與主動決策輔助系統 — 透過五步驟問卷，依使用者的預算與需求，產出個人化的健康保險組合建議，並提供商品比較與 AI 保險顧問問答功能。

## 功能特色

- **五步驟問卷**：收集性別、年齡、預算、既有保障等資訊，驅動後端 AI 產出推薦保單。
- **推薦方案（Plan Results）**：依預算整理出多種保障策略組合，供使用者快速瀏覽。
- **商品比較矩陣（Comparison Matrix）**：最多可同時比較數款保單的條款與保障內容差異。
- **AI 保險顧問（AI Assistant）**：針對已選保單進行問答，協助使用者理解方案間的差異。
- **RAG 驅動的推薦引擎**：後端串接 Qdrant 向量資料庫與 [LightRAG](https://github.com/HKUDS/LightRAG)，提供具檢索依據的保單推薦與問答。

## 技術棧

- **框架**：TanStack Start（React 19 + Vite + 檔案式路由 TanStack Router）
- **狀態管理**：Zustand
- **表單驗證**：react-hook-form + zod
- **UI**：Tailwind CSS v4、Radix UI、shadcn 風格元件
- **其他**：TanStack Query、Recharts（圖表）、react-markdown、xlsx

## 系統需求

- Node.js（建議搭配 npm 或 [bun](https://bun.sh/)，專案同時附有 `package-lock.json` 與 `bun.lock`）
- 一組可用的 RAG 後端服務（Qdrant + LightRAG），用於保單推薦與 AI 問答

## 安裝與啟動

> 第一次使用、尚未安裝任何開發環境？請看 [新手安裝指南](./BEGINNER_SETUP.md)。

### 1. 安裝套件

```bash
npm install
```

### 2. 啟動後端 RAG 服務

前端依賴一組 OpenAI 相容的 `/v1/chat/completions` 服務，啟動順序：

```text
Qdrant 建置索引
  → LightRAG服務（之後每次開啟前端前執行此檔案即可）
```

### 3. 設定環境變數

在專案根目錄建立 `.env.local`：

```bash
VITE_LLM_API_BASE_URL=/api
LLM_API_BASE_URL=http://127.0.0.1:2828
LLM_API_KEY=YOUR_API_KEY
```

- `LLM_API_KEY` 僅由後端伺服器讀取，不會送進瀏覽器。
- `LLM_API_BASE_URL` 必須指向提供 OpenAI 相容 `/v1/chat/completions` 的服務；若上游不需要 API key，可省略 `LLM_API_KEY`。

### 4. 啟動開發伺服器

```bash
npm run dev
```

## 可用指令

| 指令 | 說明 |
| --- | --- |
| `npm run dev` | 啟動開發伺服器 |
| `npm run build` | 建置正式環境版本 |
| `npm run build:dev` | 以 development 模式建置 |
| `npm run preview` | 預覽建置後的成果 |
| `npm run lint` | 執行 ESLint 檢查 |
| `npm run format` | 使用 Prettier 格式化程式碼 |

## 專案結構

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

路由相關慣例請參考 [`src/routes/README.md`](src/routes/README.md)。

## 後端 API 代理

前端呼叫 `/api/v1/chat/completions`，由 `src/server.ts` 轉發至 `LLM_API_BASE_URL` 指定的上游服務（預設 `http://127.0.0.1:2828`），並附上 `Authorization: Bearer ${LLM_API_KEY}`（若有設定）。若上游服務未啟動，問卷送出後的推薦產出與 AI 問答將會失敗。
