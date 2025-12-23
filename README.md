# 📚 書店庫存管理系統 - Legacy REST API

這是一個「傳統的 API」範例，展示傳統 REST 架構。
包含完整的 MCP Server 遷移範例，展示如何將傳統 API 轉換為 AI 可呼叫的工具。

## 🎯 專案目的

這個 API 模擬了一個真實的書店庫存管理系統，包含：

- 書籍管理（CRUD）
- 作者資訊查詢
- 分類瀏覽
- 庫存統計

## REST API (舊架構)

### 1. 環境安裝

#### 下載專案

方法一：使用 Git Clone

```bash
git clone git@github.com:livhg/demo-convert-api-to-mcp-server.git
```

方法二：直接下載 ZIP 檔

透過 [https://github.com/livhg/demo-convert-api-to-mcp-server/archive/refs/heads/main.zip](https://github.com/livhg/demo-convert-api-to-mcp-server/archive/refs/heads/main.zip) 下載並解壓縮

#### 使用 VS Code 開啟專案

```bash
cd demo-convert-api-to-mcp-server
code .
```

開啟 VS Code 後，使用快捷鍵開啟內建 Terminal：

- **Windows/Linux**: ` Ctrl + ``  ` `` (Ctrl + 反引號)
- **macOS**: ` Cmd + ``  ` ``

或從選單：`Terminal` → `New Terminal`

#### 建立虛擬環境

使用 uv 建立 Python 3.14 虛擬環境：

```bash
# 建立並啟動 Python 3.14 虛擬環境
uv venv .venv --python 3.14

# 啟動虛擬環境
# Windows
.venv\Scripts\activate

# macOS/Linux
source .venv/bin/activate
```

#### 安裝依賴

```bash
# 使用 uv 安裝專案依賴
uv pip install -r requirements.txt
```

### 2. 啟動 REST API

```bash
python app.py
```

服務啟動後會在 `http://localhost:8012` 運行

### 3. 查看 API 文件

- **Swagger UI**: http://localhost:8012/docs
- **ReDoc**: http://localhost:8012/redoc
- **OpenAPI JSON**: http://localhost:8012/openapi.json

---

## 為什麼要遷移到 MCP？

### REST API 的限制

```
┌─────────────┐      HTTP Request      ┌─────────────┐
│   Client    │ ───────────────────►   │  REST API   │
│  (前端/App) │ ◄───────────────────   │             │
└─────────────┘      JSON Response     └─────────────┘
```

- 需要人工撰寫 API 呼叫程式碼
- AI 無法直接理解和操作
- 需要額外的 API 文件

### MCP 的優勢

```
┌─────────────┐                                  ┌─────────────┐
│  AI Client  │ ◄─── Model Context Protocol ───► │ MCP Server  │
│ (Cursor/    │     (雙向通訊)                    │  (Tools/    │
│  Claude)    │                                  │  Resources) │
└─────────────┘                                  └─────────────┘
```

- AI 可以直接發現和呼叫工具
- 自動理解參數和用途
- 支援雙向通訊（包括 Sampling）

---

## 方法一：自動轉換（OpenAPI → MCP）

> **最快速的方式**：讓 FastMCP 自動從 OpenAPI 規格生成 MCP Server

### Step 1: 建立 MCP Server 檔案

建立 `bookstore_mcp.py`：

```python
# bookstore_mcp.py
# 將書店 REST API 自動轉換為 MCP Server

import httpx
from fastmcp import FastMCP

# 1. 建立 HTTP Client 連接到 REST API
client = httpx.AsyncClient(base_url="http://localhost:8012")

# 2. 取得 OpenAPI 規格
openapi_spec = httpx.get("http://localhost:8012/openapi.json").json()

# 3. 自動轉換為 MCP Server
mcp = FastMCP.from_openapi(
    openapi_spec=openapi_spec,
    client=client,
    name="📚 書店庫存管理 MCP Server"
)

if __name__ == "__main__":
    mcp.run()
```

### Step 2: 運行 MCP Server

```bash
# 確保 REST API 正在運行（在另一個終端）
# python app.py

# 啟動 MCP Server
fastmcp dev bookstore_mcp.py
```

就這樣！所有 REST API 端點都會自動轉換為 MCP Tools。

---

## 方法二：手動遷移

> **更好的 LLM 效能**：手動設計的 MCP Server 讓 AI 表現更好

### 建立 MCP Server

[bookstore_mcp_manual.py](bookstore_mcp_manual.py)

### 根據用途設定為不同模式

    - GET /books/{id} → MCP Resource
    - POST/PUT/DELETE → MCP Tools
    - 加入 Prompt Templates

### 運行 MCP Server

```bash
fastmcp dev bookstore_mcp_manual.py
```

---

## REST vs MCP 對照表

| REST API                  | MCP Server                         | 說明             |
| ------------------------- | ---------------------------------- | ---------------- |
| `GET /books`              | `@mcp.resource()` 或 `@mcp.tool()` | 讀取資料         |
| `GET /books/{id}`         | `@mcp.resource("uri/{id}")`        | 動態資源         |
| `GET /books/search?q=...` | `@mcp.tool()`                      | 有參數的查詢     |
| `POST /books`             | `@mcp.tool()`                      | 建立資源         |
| `PUT /books/{id}`         | `@mcp.tool()`                      | 更新資源         |
| `DELETE /books/{id}`      | `@mcp.tool()`                      | 刪除資源         |
| OpenAPI description       | Tool docstring                     | 告訴 AI 何時使用 |
| Query parameters          | Tool arguments                     | 函數參數         |
| JSON response             | 字串/結構化回傳                    | 回傳值           |

### 轉換原則

1. **讀取操作（無副作用）** → 優先使用 `Resource`
2. **需要參數的讀取** → 使用 `Resource Template` 或 `Tool`
3. **寫入操作（有副作用）** → 必須使用 `Tool`
4. **複雜業務邏輯** → 使用 `Tool` + 詳細 docstring

---

## 📝 注意事項

- 這是 Demo 專案，資料存放於記憶體中
- 重啟服務後資料會重置
- 不需要外部資料庫
