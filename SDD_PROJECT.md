# OpenAPI 到 MCP Server - OpenSpec SDD 實作

> **從現有 REST API 到 AI 可用的 MCP Server**

## 專案簡介

### 🎯 目標

本專案將教你如何使用 **OpenSpec** 進行 **Specification-Driven Development (SDD)**，將現有的 REST API 轉換為 AI Agent 可使用的 MCP Server。

### 🏢 真實場景

**情境**：你的公司已經有一個運行中的書店庫存管理 REST API，服務於前端網頁和行動 App。現在公司想要：

- 讓 AI Agent 能夠自動處理庫存查詢
- 讓客服人員可以用 AI 助手快速查找書籍資訊
- 讓管理層能用 AI 生成庫存報表

**挑戰**：

- ❌ 不能改動現有的 REST API（有其他系統在使用）
- ❌ 不能直接讓 AI 呼叫 REST API（需要額外的身份驗證、格式轉換）
- ✅ 需要一個「橋接層」讓 AI 能夠理解和使用現有功能

**解決方案**：使用 OpenSpec 建立 MCP Server 規格，讓 AI 根據規格生成程式碼。

### 🎓 學習目標

完成本專案後，你將能夠：

- ✅ 從現有 REST API 提取並分析 OpenAPI 規範
- ✅ 使用 OpenSpec 管理規格變更
- ✅ 設計適合 AI 使用的 MCP Tool 規格
- ✅ 讓 AI 根據規格實作高品質的程式碼
- ✅ 理解何時使用自動轉換 vs 手動設計規格

### 💡 核心理念

**Spec First, AI Implements**

```
人類（你）→ 設計規格 → AI → 生成程式碼 → 測試 → 調整規格 → 完成
```

- **你的角色**：規格設計師 - 定義「要做什麼」
- **AI 的角色**：程式實作者 - 完成「怎麼做」
- **OpenSpec**：管理規格版本、變更追蹤、自動合併

---

## 環境需求

### 必要軟體

- **Node.js** (v16+) - 用於安裝 OpenSpec
- **Python** (v3.8+) - 用於執行 REST API 和 MCP Server
- **AI Agent** - VS Code、Cursor 或其他

### 必要套件

```bash
# Python 套件
pip install fastapi uvicorn pydantic fastmcp

# OpenSpec CLI
npm install -g @fission-ai/openspec
```

### 選擇 AI 工具

本專案建議使用以下任一 AI 工具：

- **GitHub Copilot**
- **Claude Code**
- **Cursor**
- 其他支援 AGENTS.md 的 AI 工具

---

## Part 1: 環境設定與 OpenAPI 分析

### 🔧 Step 1: 使用 VS Code 打開專案

1. **啟動 VS Code**

   - 打開 VS Code 應用程式

2. **打開專案資料夾**

   - 點選 `File` > `Open Folder...`（或使用快捷鍵 `Ctrl+K Ctrl+O`）
   - 選擇 `demo-convert-api-to-mcp-server` 專案資料夾並打開

3. **開啟整合終端機**

   - 點選 `Terminal` > `New Terminal`（或使用快捷鍵 `` Ctrl+` ``）
   - 終端機會自動在專案根目錄開啟

4. **確認 OpenSpec 已安裝**

在終端機中執行：

```bash
openspec --version
```

> 💡 **提示**：後續的所有指令都將在這個 VS Code 內建的終端機中執行，請保持終端機開啟狀態。

### 🚀 Step 2: 初始化專案

在 VS Code 的終端機中初始化 OpenSpec：

```bash
openspec init
```

> 📝 **說明**：由於終端機已在專案根目錄（`demo-convert-api-to-mcp-server`）開啟，可以直接執行指令。

### 💬 Step 3: 互動式設定

執行 `openspec init` 後，CLI 會進行互動式詢問：

```
? Which AI tools do you use? (Press <space> to select, <a> to toggle all, <i> to invert selection)
❯◯ Claude Code
 ◯ CodeBuddy
 ◯ Cursor
 ◯ GitHub Copilot
 ◯ OpenCode
 ◯ Qoder
 ◯ RooCode
```

**請選擇**：**GitHub Copilot**

OpenSpec 會：

- 自動配置對應的 slash commands
- 生成 `openspec/` 目錄結構
- 建立 `AGENTS.md` 交接文件

### ⭐ Step 4: 初始化專案資訊（重要！）

初始化完成後，OpenSpec 會顯示一段提示文字（**紅框處**）：

```
Populate your project context:
"Please read openspec/project.md and help me fill it out with details about my project, tech stack, and conventions"
```

**重要步驟**：

1. 📋 **複製終端機中顯示的提示文字**
2. 💬 **開啟 GitHub Copilot Chat**
   - 點選 VS Code 左側活動列的 **Chat 圖示**（或使用快捷鍵 `Ctrl+Alt+I`）
3. 🤖 **將提示文字貼到 Copilot Chat 中並送出**
4. AI 會讀取 `openspec/project.md` 並協助填寫專案資訊

> 💡 **提示**：Copilot 會自動分析專案結構，並將專案資訊寫入 `openspec/project.md`。

這個步驟讓 AI 了解你的專案背景，以便後續生成更準確的規格和程式碼。

### 📂 了解 OpenSpec 結構

初始化完成後，你的專案會有以下結構：

```
demo-convert-api-to-mcp-server/
├── openspec/
│   ├── specs/           # 目前的正式規格文件
│   ├── changes/         # 進行中的變更
│   └── project.md       # 專案整體資訊（剛才 AI 幫你填寫的）
├── AGENTS.md           # AI 工具的交接指南
└── ... (其他專案檔案)
```

**重要概念**：

- `openspec/specs/` - 存放「已確定」的規格（真相來源）
- `openspec/changes/` - 存放「進行中」的變更（開發中）

### 🚀 Step 5: 啟動現有的 REST API

在另一個終端視窗執行：

```bash
python app.py
```

你會看到：

```
🚀 啟動書店庫存管理系統 API...
📖 API 文件: http://localhost:8012/docs
📋 ReDoc 文件: http://localhost:8012/redoc
📄 OpenAPI JSON: http://localhost:8012/openapi.json
INFO:     Uvicorn running on http://0.0.0.0:8012
```

### 🌐 Step 6: 瀏覽 Swagger UI

打開瀏覽器，前往：

**http://localhost:8012/docs**

你會看到書店 API 的完整文件，包含：

- 📖 **書籍 (Books)** - 6 個端點
- 📦 **庫存 (Inventory)** - 2 個端點
- 👤 **作者 (Authors)** - 2 個端點
- 📂 **分類 (Categories)** - 2 個端點

花幾分鐘瀏覽並測試幾個端點，了解現有系統的功能。

### 📥 Step 7: 提取 OpenAPI 規範

我們需要 OpenAPI 規範來分析 REST API 的結構。

**方法 1：從瀏覽器下載**

1. 前往 **http://localhost:8012/openapi.json**
2. 右鍵 → 「另存新檔」
3. 儲存為 `bookstore-openapi.json`

**方法 2：使用 curl 下載**

```bash
curl http://localhost:8012/openapi.json > bookstore-openapi.json
```

### 🔍 Step 8: 分析 OpenAPI 規範結構

用文字編輯器打開 `bookstore-openapi.json`，我們來看看結構：

#### 基本資訊

```json
{
  "openapi": "3.1.0",
  "info": {
    "title": "📚 書店庫存管理系統",
    "description": "...",
    "version": "1.0.0"
  },
  "paths": { ... },
  "components": { "schemas": { ... } }
}
```

#### 端點定義 (paths)

找到 `/books/search` 端點：

```json
"/books/search": {
  "get": {
    "tags": ["📖 書籍"],
    "summary": "搜尋書籍",
    "description": "根據多種條件搜尋書籍",
    "parameters": [
      {
        "name": "q",
        "in": "query",
        "required": false,
        "schema": { "type": "string" },
        "description": "搜尋關鍵字（搜尋書名與描述）"
      },
      {
        "name": "author_id",
        "in": "query",
        "required": false,
        "schema": { "type": "integer" }
      }
      // ... 更多參數
    ],
    "responses": {
      "200": {
        "description": "Successful Response",
        "content": {
          "application/json": {
            "schema": {
              "type": "array",
              "items": { "$ref": "#/components/schemas/BookWithDetails" }
            }
          }
        }
      }
    }
  }
}
```

#### 參數類型解讀

OpenAPI 有三種主要參數位置：

| 位置    | 說明         | 範例                      |
| ------- | ------------ | ------------------------- |
| `query` | URL 查詢參數 | `?q=spring&author_id=1`   |
| `path`  | URL 路徑參數 | `/books/{id}` 中的 `{id}` |
| `body`  | 請求主體     | POST/PUT 的 JSON 資料     |

#### Response Schema 分析

找到 `components.schemas.BookWithDetails`：

```json
"BookWithDetails": {
  "properties": {
    "id": { "type": "integer" },
    "title": { "type": "string" },
    "author_id": { "type": "integer" },
    "author_name": { "type": "string" },
    "category_name": { "type": "string" },
    "price": { "type": "number" },
    "stock": { "type": "integer" }
  },
  "type": "object",
  "required": ["id", "title", "author_id", ...]
}
```

### 📊 Step 9: 分析 4 個選定的端點

我們將專注於以下 4 個端點，它們涵蓋不同的複雜度：

#### 1. `GET /books/search` → `search_books` Tool

**特點**：

- 複雜查詢參數（keyword, author_id, category_id, min_price, max_price, in_stock）
- 所有參數都是 optional
- 回傳書籍陣列

**OpenAPI 摘要**：

```json
{
  "parameters": [
    { "name": "q", "in": "query", "type": "string" },
    { "name": "author_id", "in": "query", "type": "integer" },
    { "name": "category_id", "in": "query", "type": "integer" },
    { "name": "min_price", "in": "query", "type": "number" },
    { "name": "max_price", "in": "query", "type": "number" },
    { "name": "in_stock", "in": "query", "type": "boolean" }
  ]
}
```

#### 2. `GET /books/{id}` → `get_book_detail` Tool

**特點**：

- 單一路徑參數 `book_id`
- 回傳單一書籍詳細資料
- 可能回傳 404 錯誤

**OpenAPI 摘要**：

```json
{
  "parameters": [
    { "name": "book_id", "in": "path", "required": true, "type": "integer" }
  ],
  "responses": {
    "200": { "schema": { "$ref": "#/components/schemas/BookWithDetails" } },
    "404": { "description": "書籍不存在" }
  }
}
```

#### 3. `PATCH /books/{id}/stock` → `update_stock` Tool

**特點**：

- 路徑參數 + 請求主體
- 寫入操作（有副作用）
- 需要處理庫存不足的情況

**OpenAPI 摘要**：

```json
{
  "parameters": [
    { "name": "book_id", "in": "path", "required": true, "type": "integer" }
  ],
  "requestBody": {
    "content": {
      "application/json": {
        "schema": {
          "properties": {
            "quantity_change": {
              "type": "integer",
              "description": "正數增加，負數減少"
            }
          }
        }
      }
    }
  }
}
```

#### 4. `GET /inventory/stats` → `get_inventory_report` Tool

**特點**：

- 無參數
- 複雜的回傳資料結構（統計資料）
- 包含低庫存書籍清單

**OpenAPI 摘要**：

```json
{
  "responses": {
    "200": {
      "schema": {
        "$ref": "#/components/schemas/InventoryStats"
      }
    }
  }
}
```

`InventoryStats` schema：

```json
{
  "properties": {
    "total_titles": { "type": "integer" },
    "total_stock": { "type": "integer" },
    "total_inventory_value": { "type": "number" },
    "low_stock_count": { "type": "integer" },
    "out_of_stock_count": { "type": "integer" },
    "low_stock_books": { "type": "array", "items": { ... } }
  }
}
```

### ✅ Part 1 檢查點

完成 Part 1 後，你應該：

- ✅ 已安裝 OpenSpec CLI
- ✅ 已初始化專案並選擇 GitHub Copilot
- ✅ 已複製提示給 AI 填寫 `project.md`
- ✅ REST API 正在 `http://localhost:8012` 運行
- ✅ 已下載並瀏覽 `bookstore-openapi.json`
- ✅ 了解 4 個選定端點的結構

---

## Part 2: OpenSpec 介紹與設計原則

### 📝 OpenSpec 是什麼？

[OpenSpec](https://github.com/Fission-AI/OpenSpec) 是一個 **Spec-driven development** 工具，幫助你：

1. 📋 **管理規格變更** - 用 change proposals 追蹤每個功能
2. 📦 **版本控制規格** - specs/ 是真相來源，changes/ 是開發中
3. 🤖 **AI 協作** - AI 讀取 spec 並生成程式碼
4. 🔄 **自動合併** - 完成後自動將 spec delta 合併到正式規格

### 📝 建立第一個 Change Proposal

OpenSpec 的工作流程從建立 **change proposal** 開始。

**方式 1：請 AI 創建**（自然語言）

```
User：請創建一個 OpenSpec change proposal，用於新增 search_books MCP tool
```

**方式 2：使用 Slash Command**（如果你的 IDE 支援）

```
/openspec:proposal Add search_books MCP tool
```

AI 會創建以下結構：

```
openspec/changes/add-search-books-tool/
├── proposal.md          # 變更說明（為什麼做、做什麼）
├── tasks.md            # 實作任務清單（待辦事項）
└── specs/
    └── mcp-tools/
        └── spec.md     # Tool 規格 delta（新增的需求）
```

### 🎯 從 REST 到 MCP 的轉換思維

為什麼不能 1:1 對應？

| REST API 特性    | 為何不適合 AI            | MCP Tool 優化          |
| ---------------- | ------------------------ | ---------------------- |
| 複雜的 JSON 結構 | AI 難以理解嵌套結構      | 使用結構化字串 + emoji |
| HTTP 狀態碼      | AI 需要額外處理錯誤      | 友善的錯誤訊息字串     |
| 缺少使用時機說明 | AI 不知道何時該用        | Docstring 說明使用情境 |
| RESTful 設計     | 需要多次呼叫才能完成任務 | 合併相關功能           |

**範例對比**：

**REST API 回傳**（JSON）：

```json
[
  {
    "id": 1,
    "title": "挪威的森林",
    "author_id": 1,
    "author_name": "村上春樹",
    "category_id": 1,
    "category_name": "文學小說",
    "price": 380,
    "stock": 25
  }
]
```

**MCP Tool 回傳**（結構化字串）：

```
📚 找到 1 本書籍：
----------------------------------------

📖 [1] 挪威的森林
   作者：村上春樹
   分類：文學小說
   價格：$380 | 庫存：25 ✅
```

**為什麼更好？**

- ✅ AI 更容易理解和總結
- ✅ 使用者直接可讀
- ✅ 已經格式化，無需額外處理

---

## Part 3: 完整示範 - search_books Tool

現在讓我們完整走一遍 OpenSpec 工作流程，從 `GET /books/search` REST API 端點建立 `search_books` MCP Tool。

### 📝 Step 1: 建立 Change Proposal

**請 AI 創建 change proposal**：

```
User：Create an OpenSpec change proposal for adding search_books MCP tool based on the GET /books/search endpoint
```

或使用 slash command（如果你的 IDE 支援）：

```
/openspec:proposal Add search_books MCP tool
```

AI 會建立以下結構：

```
openspec/changes/add-search-books-tool/
├── proposal.md          # 變更說明
├── tasks.md            # 實作任務清單
└── specs/
    └── mcp-tools/
        └── spec.md     # Tool 規格 delta
```

### 🔍 Step 2: 檢視與驗證 Change

檢視所有 changes：

```bash
openspec list
```

輸出：

```
Active changes:
  - add-search-books-tool
```

查看詳細內容：

```bash
openspec show add-search-books-tool
```

### 📋 Step 3: 在 Spec 中定義 Tool 規格

這是最關鍵的步驟！我們要撰寫清晰的規格，讓 AI 能夠根據規格生成程式碼。

打開 `openspec/changes/add-search-books-tool/specs/mcp-tools/spec.md`，撰寫以下內容：

```markdown
# MCP Tools Specification Delta

## ADDED Requirements

### Requirement: search_books Tool

The MCP server SHALL provide a search_books tool that allows searching for books based on multiple criteria.

The tool MUST accept optional parameters for flexible searching and return formatted results optimized for AI understanding.

#### Scenario: Search by keyword

- GIVEN the bookstore has books with titles and descriptions
- WHEN user provides keyword "spring"
- THEN return all books where title or description contains "spring"
- AND format results with emoji and structured layout

#### Scenario: Filter by author

- GIVEN the bookstore has books by multiple authors
- WHEN user provides author_id=1
- THEN return only books by that author
- AND include author name in results

#### Scenario: Filter by category

- GIVEN the bookstore has books in multiple categories
- WHEN user provides category_id=2
- THEN return only books in that category
- AND include category name in results

#### Scenario: Filter by price range

- GIVEN the bookstore has books with various prices
- WHEN user provides min_price=300 and max_price=500
- THEN return only books within that price range
- AND display prices clearly

#### Scenario: Filter by stock availability

- GIVEN some books are in stock and some are out of stock
- WHEN user provides in_stock=true
- THEN return only books with stock > 0
- AND mark availability status with ✅ or ❌

#### Scenario: Combine multiple filters

- GIVEN user wants to find specific books
- WHEN user provides multiple parameters (e.g., keyword + category + price range)
- THEN apply all filters using AND logic
- AND return books matching all criteria

#### Scenario: No results found

- GIVEN user's search criteria
- WHEN no books match the criteria
- THEN return friendly message "📭 找不到符合條件的書籍"
- AND suggest to try different criteria

### Requirement: search_books Parameters

The tool SHALL accept the following optional parameters:

- `keyword` (string, default=""): Search term for title and description
- `author_id` (integer, optional): Filter by author ID
- `category_id` (integer, optional): Filter by category ID
- `min_price` (float, optional): Minimum price filter
- `max_price` (float, optional): Maximum price filter
- `in_stock` (boolean, optional): Filter by stock availability

### Requirement: search_books Return Format

The tool SHALL return a formatted string with the following structure:

📚 找到 {count} 本書籍：

📖 [{id}] {title}
作者：{author_name}
分類：{category_name}
價格：${price} | 庫存：{stock} {status}

[... more books ...]

Where:

- `{count}` is the number of books found
- `{status}` is ✅ for in stock, ❌ 缺貨 for out of stock
- Each book is separated by a newline for readability

### Requirement: search_books Tool Description

The tool's docstring SHALL clearly explain:

- What the tool does
- When to use it (e.g., "當使用者想找特定書籍或瀏覽書籍時使用")
- All parameters and their purposes
- Return value format
- Usage examples

This helps AI agents understand when and how to use the tool.
```

**重要提示**：

- 規格要詳細但清晰
- 每個 Scenario 描述一個具體的使用情境
- 使用 GIVEN/WHEN/THEN 格式
- 說明預期的行為和格式

驗證規格格式：

```bash
openspec validate add-search-books-tool
```

如果格式正確，會顯示：

```
✓ Spec format is valid
```

### 🤖 Step 4: 用 AI 實作程式碼

現在規格已經定義好了，讓 AI 根據規格生成程式碼！

**請 AI 實作**：

```
User：Please implement the search_books tool according to the OpenSpec specification in openspec/changes/add-search-books-tool/specs/mcp-tools/spec.md. Create a new file bookstore_mcp.py with the FastMCP server implementation.
```

或使用 slash command：

```
/openspec:apply add-search-books-tool
```

**AI 會做什麼**：

1. 讀取 `openspec/changes/add-search-books-tool/specs/mcp-tools/spec.md`
2. 理解所有 Requirements 和 Scenarios
3. 生成符合規格的 Python 程式碼
4. 在 `tasks.md` 中標記完成的任務

### ✅ Step 5: 測試與調整

#### 測試 1：使用 FastMCP Dev Mode

```bash
fastmcp dev bookstore_mcp.py
```

這會啟動一個開發伺服器，你可以：

- 查看 tool 列表
- 測試 tool 呼叫
- 檢查回傳格式

#### 測試 2：在瀏覽器中測試

FastMCP dev mode 會提供一個 Web UI，前往顯示的 URL（通常是 `http://localhost:3000`）。

嘗試不同的參數組合：

**測試案例 1：關鍵字搜尋**

```json
{
  "keyword": "春樹"
}
```

**測試案例 2：價格範圍**

```json
{
  "min_price": 300,
  "max_price": 400
}
```

**測試案例 3：有庫存的書**

```json
{
  "in_stock": true
}
```

**測試案例 4：組合條件**

```json
{
  "keyword": "小說",
  "category_id": 1,
  "in_stock": true
}
```

#### 測試 3：用 AI Agent 測試

可以配置 MCP server 並實際測試：

```
User：幫我找價格在 300 到 500 之間的書
```

AI 會呼叫：

```python
search_books(min_price=300, max_price=500)
```

#### 調整規格

如果測試中發現問題，使用 OpenSpec 工作流程修正：

> 📌 **說明**：因為 change proposal 已經在 Step 1 建立了（`openspec/changes/add-search-books-tool/`），所以只需要修改其中的規格檔案並重新生成程式碼，不需要重新建立 proposal。

1. **修改規格**：在 VS Code 中編輯 `openspec/changes/add-search-books-tool/specs/mcp-tools/spec.md`
2. **驗證規格**：在終端機中執行
   ```bash
   openspec validate add-search-books-tool
   ```
3. **重新實作**：在 Copilot Chat 中請 AI 根據更新後的規格重新實作
   ```
   請根據 openspec/changes/add-search-books-tool/specs/mcp-tools/spec.md 的更新規格重新實作 search_books tool
   ```
   或使用 slash command（如果 IDE 支援）：
   ```
   /openspec:apply add-search-books-tool
   ```
4. **再次測試**：重複 Step 5 的測試流程

> 💡 **Spec-driven 精髓**：規格是真相來源，程式碼由規格生成。當發現問題時，先修正規格，再重新生成程式碼。

### 📦 Step 6: 歸檔完成的 Change

測試通過後，歸檔這個 change：

```bash
openspec archive add-search-books-tool --yes
```

或請 AI 歸檔：

```
User：Please archive the add-search-books-tool change
```

**歸檔後會發生什麼**：

1. `openspec/changes/add-search-books-tool/` 移至 `openspec/archive/`
2. Spec delta 自動合併到 `openspec/specs/mcp-tools/spec.md`
3. 成為專案的正式規範文件

查看合併後的正式規格：

```bash
cat openspec/specs/mcp-tools/spec.md
```

你會看到 `search_books` 的規格已經成為正式文件的一部分！

### 🎉 Part 3 完成！

你已經完整走過一遍 OpenSpec 工作流程：

1. ✅ 建立 change proposal
2. ✅ 定義清晰的規格（spec delta）
3. ✅ 讓 AI 根據規格生成程式碼
4. ✅ 測試並調整
5. ✅ 歸檔 change，合併規格

### 📊 OpenSpec 工作流程總結

```
1. openspec init           → 初始化專案（已完成）
2. 創建 proposal           → 定義要做什麼變更
3. openspec list/show      → 檢視 changes
4. 撰寫 spec delta         → 定義詳細規格
5. openspec validate       → 驗證格式
6. AI 實作程式碼           → 根據 spec 生成
7. 測試與調整             → 迭代改善規格和程式碼
8. openspec archive        → 歸檔完成的 change
```

---

## Part 4: 練習與總結

現在輪到你了！請使用相同的流程，為剩餘 3 個 REST API 端點建立 MCP Tools。

### 📋 練習概述

需要完成：

1. `get_book_detail` Tool (from `GET /books/{id}`)
2. `update_stock` Tool (from `PATCH /books/{id}/stock`)
3. `get_inventory_report` Tool (from `GET /inventory/stats`)

### 🎯 實作流程（每個 tool）

按照以下步驟完成每個 tool：

```
1. 分析對應的 REST API OpenAPI 定義
   ↓
2. 建立 OpenSpec change proposal
   ↓
3. 在 spec delta 中定義 tool 規格
   ↓
4. 用 openspec validate 驗證格式
   ↓
5. 請 AI 根據 spec 實作程式碼
   ↓
6. 測試並歸檔 change
```

---

### 練習 1: get_book_detail Tool

#### 📖 REST API 分析

**端點**：`GET /books/{id}`

#### 🎯 設計目標

- 接受單一參數：`book_id` (必填)
- 回傳書籍的完整詳細資訊
- 處理找不到書籍的情況

#### 📝 規格設計框架（填寫提示）

```markdown
## ADDED Requirements

### Requirement: get_book_detail Tool

The MCP server SHALL provide a get_book_detail tool that retrieves detailed information about a specific book.

#### Scenario: Get existing book

- GIVEN a book with ID 1 exists
- WHEN user provides book_id=1
- THEN return complete book details including:
  - Title, ISBN, author info, category, price, stock
  - Description and publication date
  - Formatted for easy reading

#### Scenario: Book not found

- GIVEN no book with ID 999 exists
- WHEN user provides book_id=999
- THEN return error message "❌ 找不到 ID 為 999 的書籍"

### Requirement: get_book_detail Return Format

The tool SHALL return detailed information formatted as:

📖 {title}

📝 基本資訊
ISBN：{isbn}
作者：{author_name} ({author_country})
分類：{category_name}
出版日期：{publish_date}

💰 價格與庫存
價格：${price}
庫存：{stock} 本 ({status})

📄 簡介
{description}

👤 關於作者
{author_bio}
```

#### ✅ 實作檢查點

完成後，你的 tool 應該能夠：

- [ ] 接受 book_id 參數
- [ ] 回傳格式化的詳細資訊
- [ ] 包含 ISBN、作者、分類、價格、庫存
- [ ] 顯示書籍描述和作者簡介
- [ ] 友善處理找不到書籍的情況
- [ ] Docstring 說明「當使用者想了解特定書籍的詳細內容時使用」

#### 💡 提示

- 使用 `mock_db.get_book_by_id(book_id)` 取得書籍資料
- 使用 `mock_db.get_author_by_id(author_id)` 取得作者資料
- 使用 `mock_db.get_category_by_id(category_id)` 取得分類資料
- 參考 `bookstore_mcp_manual.py` 中的 `get_book_detail` 實作

---

### 練習 2: update_stock Tool

#### 📖 REST API 分析

**端點**：`PATCH /books/{id}/stock`

#### 🎯 設計目標

- 接受兩個參數：`book_id` (必填) 和 `quantity_change` (必填)
- 正數表示進貨，負數表示出貨
- 處理庫存不足的情況
- 顯示更新前後的庫存數量

#### 📝 規格設計框架（填寫提示）

```markdown
## ADDED Requirements

### Requirement: update_stock Tool

The MCP server SHALL provide an update_stock tool to modify book inventory.

The tool MUST support both increasing (restocking) and decreasing (selling) inventory quantities.

#### Scenario: Restock books (positive change)

- GIVEN a book has current stock of 10
- WHEN user provides book_id=1, quantity_change=5
- THEN increase stock to 15
- AND return success message showing old and new stock

#### Scenario: Sell books (negative change)

- GIVEN a book has current stock of 10
- WHEN user provides book_id=1, quantity_change=-3
- THEN decrease stock to 7
- AND return success message showing old and new stock

#### Scenario: Insufficient stock

- GIVEN a book has current stock of 5
- WHEN user provides book_id=1, quantity_change=-10
- THEN return error "❌ 錯誤：庫存不足，無法減少 10 本（目前庫存：5）"
- AND do not change stock

#### Scenario: Book not found

- GIVEN no book with ID 999 exists
- WHEN user provides book_id=999
- THEN return error message

### Requirement: update_stock Return Format

The tool SHALL return formatted result:

✅ 庫存更新成功

📖 書籍：{book_title}
{action}：{abs(quantity_change)} 本
原庫存：{old_stock} 本
現庫存：{new_stock} 本

Where {action} is "📥 進貨" for positive or "📤 出貨" for negative.
```

#### ✅ 實作檢查點

完成後，你的 tool 應該能夠：

- [ ] 接受 book_id 和 quantity_change 參數
- [ ] 處理正數（進貨）和負數（出貨）
- [ ] 在庫存不足時回傳友善錯誤訊息
- [ ] 顯示更新前後的庫存數量
- [ ] 顯示進貨或出貨的動作
- [ ] Docstring 包含使用範例（如何進貨和出貨）

#### 💡 提示

- 使用 `mock_db.get_book_by_id(book_id)` 取得當前庫存
- 使用 `mock_db.update_stock(book_id, quantity_change)` 更新庫存
- `update_stock` 回傳 None 表示庫存不足
- 使用 `abs()` 函數取得絕對值來顯示數量

---

### 練習 3: get_inventory_report Tool

#### 📖 REST API 分析

**端點**：`GET /inventory/stats`

#### 🎯 設計目標

- 無參數
- 回傳完整的庫存統計報告
- 包含總覽數據和低庫存警示
- 格式化為易讀的報表

#### 📝 規格設計框架（填寫提示）

```markdown
## ADDED Requirements

### Requirement: get_inventory_report Tool

The MCP server SHALL provide a get_inventory_report tool that generates a comprehensive inventory statistics report.

The tool is used when users want to check overall inventory status, identify low-stock items, or generate management reports.

#### Scenario: Generate full report

- GIVEN the bookstore has multiple books with varying stock levels
- WHEN user requests inventory report (no parameters needed)
- THEN return report containing:
  - Total number of book titles
  - Total stock quantity across all books
  - Total inventory value (price × stock)
  - Count of low-stock books (< 10 units)
  - Count of out-of-stock books
  - List of books needing restock

#### Scenario: Low stock alert

- GIVEN some books have stock < 10
- WHEN report is generated
- THEN clearly mark these books with 🟡 or 🔴
- AND sort by urgency (out of stock first)

#### Scenario: All stock healthy

- GIVEN all books have stock >= 10
- WHEN report is generated
- THEN show "✅ 所有書籍庫存充足！"

### Requirement: get_inventory_report Return Format

The tool SHALL return a formatted report:

# 📊 庫存統計報告

📈 總覽
書籍種類：{total_titles} 種
總庫存量：{total_stock} 本
庫存總值：${total_inventory_value:,.0f}

⚠️ 警示
低庫存（<10 本）：{low_stock_count} 種
完全缺貨：{out_of_stock_count} 種

🔔 需要補貨的書籍：
{urgency} [{id}] {title} - 剩餘 {stock} 本
[... more books ...]

Where {urgency} is 🔴 for out of stock or 🟡 for low stock.
```

#### ✅ 實作檢查點

完成後，你的 tool 應該能夠：

- [ ] 不需要參數即可執行
- [ ] 顯示書籍種類、總庫存、庫存總值
- [ ] 顯示低庫存和缺貨數量
- [ ] 列出需要補貨的書籍
- [ ] 使用 🔴 標示完全缺貨，🟡 標示低庫存
- [ ] 格式化數字（使用千分位逗號）
- [ ] Docstring 說明「當使用者詢問庫存狀況或需要補貨的書籍時使用」

#### 💡 提示

- 使用 `mock_db.get_inventory_stats()` 取得統計資料
- 使用 `f"${value:,.0f}"` 格式化金額（加入千分位逗號）
- 低庫存書籍清單已經包含在 stats 中
- stock == 0 用 🔴，stock < 10 但 > 0 用 🟡

---

### ❓ 常見問題 Q&A

**Q1: 如果 AI 生成的程式碼不符合規格怎麼辦？**

A: 先檢查規格是否寫得夠清楚。如果規格清楚但 AI 理解錯誤，可以：

- 提供更多 context
- 在 prompt 中引用具體的 Scenario
- 要求 AI 逐個 Scenario 實作並確認

**Q2: 規格要寫到多詳細？**

A: 遵循「清晰但不過度」原則：

- ✅ 描述所有重要的行為和邊界情況
- ✅ 使用具體的範例
- ❌ 不需要描述實作細節（如使用哪個函數）
- ❌ 不需要描述資料庫 schema

**Q3: 可以同時有多個 active changes 嗎？**

A: 可以！但建議：

- 一次專注於一個 change
- 確保 changes 之間不會衝突
- 定期歸檔完成的 changes

**Q4: 歸檔後發現問題怎麼辦？**

A: 建立新的 change 來修改：

- 建立 change proposal（如 `fix-search-books`）
- 在 spec delta 中使用 `MODIFIED Requirements`
- 實作並測試
- 歸檔，spec 會被更新

**Q5: OpenSpec 只能用於 MCP Server 嗎？**

A: 不是！OpenSpec 可以用於任何需要規格管理的專案：

- API 開發
- 前端元件
- CLI 工具
- 任何需要規格驅動的開發

---

## 附錄

### 參考答案

<details>
<summary>👁️ 點擊展開 - get_inventory_report 參考答案</summary>

```markdown
# MCP Tools Specification Delta

## ADDED Requirements

### Requirement: get_inventory_report Tool

The MCP server SHALL provide a get_inventory_report tool that generates a comprehensive inventory statistics report.

The tool is used when users want to check overall inventory status, identify books needing restock, generate management reports, or assess inventory health.

#### Scenario: Generate complete inventory report

- GIVEN the bookstore has multiple books with varying stock levels
- WHEN user requests inventory report (no parameters needed)
- THEN return comprehensive report containing:
  - Total number of book titles (種類)
  - Total stock quantity across all books (總庫存量)
  - Total inventory value (sum of price × stock for all books)
  - Count of low-stock books (stock < 10)
  - Count of out-of-stock books (stock = 0)
  - List of books needing restock with urgency indicators

#### Scenario: Low stock alert with books needing restock

- GIVEN some books have stock < 10
- WHEN report is generated
- THEN list all low-stock books with:
  - 🔴 indicator for out of stock (stock = 0) - highest urgency
  - 🟡 indicator for low stock (0 < stock < 10) - medium urgency
  - Book ID and title
  - Remaining stock quantity
- AND sort by urgency (out of stock first, then by stock level)

#### Scenario: All stock levels healthy

- GIVEN all books have stock >= 10
- WHEN report is generated
- THEN show all statistics normally
- AND show message "✅ 所有書籍庫存充足！" instead of restock list

### Requirement: get_inventory_report Parameters

The tool SHALL accept no parameters as it reports on the entire inventory.

### Requirement: get_inventory_report Return Format

The tool SHALL return a formatted report:

📊 庫存統計報告

📈 總覽
書籍種類：{total_titles} 種
總庫存量：{total_stock} 本
庫存總值：${total_inventory_value:,.0f}

⚠️ 警示
低庫存（<10 本）：{low_stock_count} 種
完全缺貨：{out_of_stock_count} 種

🔔 需要補貨的書籍：
{urgency} [{id}] {title} - 剩餘 {stock} 本
[... more books ...]

Where:

- `{urgency}` is 🔴 for stock = 0, 🟡 for 0 < stock < 10
- Format inventory value with thousands separator: `{value:,.0f}`
- Use double line separator (40 equals) under title
- Group information into clear sections with emoji headers

If all stock is healthy (no low-stock books):

📊 庫存統計報告

📈 總覽
書籍種類：{total_titles} 種
總庫存量：{total_stock} 本
庫存總值：${total_inventory_value:,.0f}

⚠️ 警示
低庫存（<10 本）：0 種
完全缺貨：0 種

✅ 所有書籍庫存充足！

### Requirement: get_inventory_report Tool Description

The tool's docstring SHALL clearly explain:

- What the tool does: "取得完整的庫存統計報告"
- When to use it: "當使用者詢問庫存狀況、統計資料、需要補貨的書籍時使用"
- No parameters needed
- Return value: "詳細的庫存統計報告"
- Mention that report includes:
  - Overall statistics (titles, stock, value)
  - Low stock alerts
  - List of books needing restock

This helps AI agents understand when this comprehensive report is more appropriate than individual book queries.
```

</details>

---

## 結語

恭喜你完成了這個專案！你現在已經掌握了使用 OpenSpec 進行 Specification-Driven Development 的完整流程。

記住核心理念：**Spec First, AI Implements**

人類負責思考和設計，AI 負責實作。這樣的分工讓我們能夠專注於真正重要的事情：設計出優秀的產品規格。

---

Happy Specification-Driven Development！🚀
