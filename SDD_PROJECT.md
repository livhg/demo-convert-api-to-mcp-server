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
- ✅ 需要一個「橋接層」讓 AI 能夠理解和使用現有功能

**解決方案**：使用 OpenSpec 建立 MCP Server 規格，讓 AI 根據規格生成程式碼。

### 💡 核心理念

**Spec First, AI Implements**

```
人類（你）設計規格 + 技術決策
 → AI 生成程式碼
 → 測試 → 調整規格 → 再次實作
 → 完成
```

- **你的角色**：規格設計師 - 定義「要做什麼」和「怎麼做」
- **AI 的角色**：程式實作者 - 根據規格執行
- **OpenSpec**：管理規格版本、變更追蹤、自動合併

---

## 環境需求

### 必要軟體

- **Node.js** (v16+) - 用於安裝 OpenSpec
- **Python** (v3.11+) - 用於執行 REST API 和 MCP Server
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

2. **開啟整合終端機**

   - 點選 `Terminal` > `New Terminal`（或使用快捷鍵 `` Ctrl+` ``）
   - 終端機會自動在專案根目錄開啟

3. **確認 OpenSpec 已安裝**

在終端機中執行：

```bash
openspec --version

# 若尚未安裝，請先安裝
npm install -g @fission-ai/openspec
```

### 🚀 Step 2: 初始化專案

在 VS Code 的終端機中初始化 OpenSpec：

```bash
openspec init
```

### 💬 Step 3: 互動式設定

執行 `openspec init` 後，CLI 會進行互動式詢問：

```
? Which AI tools do you use?
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

### ⭐ Step 4: 初始化專案資訊

初始化完成後，OpenSpec 會顯示一段提示文字：

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
│   └── changes/         # 進行中的變更
├── AGENTS.md            # AI 工具的交接指南
└── project.md           # 專案整體資訊（剛才 AI 幫你填寫的）
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

### 📥 Step 6: 提取 OpenAPI 規範

我們需要 OpenAPI 規範讓 AI 能夠參考真實的 REST API 定義，確保生成的 MCP tool 與 API 完全一致。

**AI 會使用這個檔案來**：

- 自動理解端點的參數、類型、必填/選填
- 了解回應的資料結構
- 處理錯誤情況
- 確保 MCP tool 規格的準確性

**方法 1：使用 curl 下載**

```bash
curl http://localhost:8012/openapi.json > bookstore-openapi.json
```

**方法 2：從瀏覽器下載**

1. 前往 **http://localhost:8012/openapi.json**
2. 右鍵 → 「另存新檔」
3. 儲存為 `bookstore-openapi.json`

### 📊 Step 7: 分析 4 個選定的端點

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

[OpenSpec](https://github.com/Fission-AI/OpenSpec) 是一個 **Spec-Driven Development** 工具，幫助你：

1. 📋 **管理規格變更** - 用 change proposals 追蹤每個功能
2. 📦 **版本控制規格** - specs/ 是真相來源，changes/ 是開發中
3. 🤖 **AI 協作** - AI 讀取 spec 並生成程式碼
4. 🔄 **自動合併** - 完成後自動將 spec delta 合併到正式規格

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

請 AI 參考 OpenAPI 規範來建立準確的 proposal：

```
請參考 bookstore-openapi.json 中的 GET /books/search 端點，
創建一個 OpenSpec change proposal for adding search_books MCP tool。

要求：
1. 確保 MCP tool 的參數與 REST API 完全一致
2. 將 MCP tool 建立在 sdd_mcp.py (若檔案不存在則建立該檔，並完成 MCP Server 架構)
```

AI 會建立以下結構：

```
openspec/changes/add-search-books-mcp-tool/
├── proposal.md          # 變更說明
├── tasks.md            # 實作任務清單
└── specs/
    └── sdd-mcp-tools/
        └── spec.md     # Tool 規格 delta
```

> 💡 **關鍵**：AI 會自動讀取並理解 OpenAPI 規範，確保 proposal 和後續的 spec 設計與 REST API 保持一致，避免遺漏參數或錯誤處理。

### 🔍 Step 2: 檢視與驗證 Change

檢視所有 changes：

```bash
openspec list
```

查看詳細內容：

```bash
openspec show add-search-books-mcp-tool
```

### 📋 Step 3: 審查與確認

這是最關鍵的步驟！

請檢查 AI 產生的 Proposal 與 Spec 使否正確。

> 💡 Spec 是實作的藍圖，需要人類的領域知識和判斷來確保其正確性和完整性。AI 可以協助生成初稿，但最終確認必須由人類負責。

### ✅ Step 4: 驗證規格格式

```bash
openspec validate add-search-books-mcp-tool
```

如果格式正確，會顯示：

```
Change 'add-search-books-mcp-tool' is valid
```

### 🤖 Step 5: 用 AI 實作程式碼

現在規格已經定義好了，讓 AI 根據規格生成程式碼！

**請 AI 實作**：

```
Please implement the search_books tool according to the OpenSpec change proposal:
openspec/changes/add-search-books-mcp-tool/
```

**AI 會做什麼**：

1. 讀取 `openspec/changes/add-search-books-mcp-tool/specs/sdd-mcp-tools/spec.md`
2. 理解所有 Requirements 和 Scenarios
3. 生成符合規格的 Python 程式碼
4. 在 `tasks.md` 中標記完成的任務

### ✅ Step 6: 測試與調整

#### 測試 1：使用 FastMCP Dev Mode

```bash
fastmcp dev sdd_mcp.py
```

這會啟動一個開發伺服器，你可以：

- 查看 tool 列表
- 測試 tool 呼叫
- 檢查回傳格式

#### 測試 2：用 AI Agent 測試

可以配置 MCP server 並實際測試：

User：

```
幫我找價格在 300 到 500 之間的書
```

AI 會呼叫：

```python
search_books(min_price=300, max_price=500)
```

若需要 venv 內的 python：

```
python -c "import sys, json; print(json.dumps(sys.executable))"
```

#### 調整規格

如果測試中發現問題，使用 OpenSpec 工作流程修正：

1. **修改規格**：在 VS Code 中編輯 `openspec/changes/add-search-books-mcp-tool/specs/sdd-mcp-tools/spec.md`
2. **驗證規格**：在終端機中執行
   ```bash
   openspec validate add-search-books-mcp-tool
   ```
3. **重新實作**：在 Copilot Chat 中請 AI 根據更新後的規格重新實作

   ```
   請根據 openspec/changes/add-search-books-mcp-tool/specs/sdd-mcp-tools/spec.md 的更新規格重新實作 search_books tool
   ```

4. **再次測試**：重複 Step 6 的測試流程

> 💡 **Spec-driven 精髓**：規格是真相來源，程式碼由規格生成。當發現問題時，先修正規格，再重新生成程式碼。

### 📦 Step 7: 歸檔完成的 Change

測試通過後，歸檔這個 change：

```bash
openspec archive add-search-books-mcp-tool --yes
```

或請 AI 歸檔：

```
Please archive the add-search-books-mcp-tool change
```

**歸檔後會發生什麼**：

1. `openspec/changes/add-search-books-mcp-tool/` 移至 `openspec/archive/`
2. Spec delta 自動合併到 `openspec/specs/sdd-mcp-tools/spec.md`
3. 成為專案的正式規範文件

你會看到 `search_books` 的規格已經成為正式文件的一部分！

### 🎉 Part 3 完成！

你已經完整走過一遍 OpenSpec 工作流程：

1. ✅ 建立 change proposal
2. ✅ 定義清晰的規格
3. ✅ 讓 AI 根據規格生成程式碼
4. ✅ 測試並調整
5. ✅ 歸檔 change，合併規格

### 📊 OpenSpec 工作流程總結

```
1. openspec init           → 初始化專案（已完成）
2. 建立 proposal           → AI 參考 OpenAPI 規範建立
3. openspec list/show      → 檢視 changes
4. 撰寫 spec               → AI 參考 OpenAPI 規範撰寫
4. 審查與確認               → 由你做 Spec 的確認與把關
5. openspec validate       → 驗證格式
6. AI 實作程式碼            → 根據 spec 生成
7. 測試與調整               → 迭代改善規格和程式碼
8. openspec archive        → 歸檔完成的 change
```

---

## Part 4: 練習使用 SDD 做為開發流程

現在輪到你了！請使用相同的流程，為剩餘 3 個 REST API 端點建立 MCP Tools。

### 📋 練習概述

需要完成：

1. `get_book_detail` Tool (from `GET /books/{id}`)
2. `update_stock` Tool (from `PATCH /books/{id}/stock`)
3. `get_inventory_report` Tool (from `GET /inventory/stats`)

### 🎯 實作流程（每個 tool）

按照以下步驟完成每個 tool：

```
1. 建立 proposal
   （請 AI 參考 OpenAPI 規範建立準確的 proposal）
   ↓
2. 審查與確認
   ↓
3. 驗證格式
   ↓
4. 請 AI 實作程式碼
   ↓
5. 測試
   ↓
6. 歸檔 change
```

---

### 練習 1: get_book_detail Tool

#### 📝 建立 Proposal

請 AI 參考 OpenAPI 規範建立 proposal：

```
請參考 bookstore-openapi.json 中的 GET /books/{id} 端點，
創建一個 OpenSpec change proposal for adding get_book_detail MCP tool in sdd_mcp.py。
確保參數和錯誤處理與 REST API 一致。
```

#### 📖 參考資訊

**端點**：`GET /books/{id}`

#### 🎯 設計目標

- 接受單一參數：`book_id` (必填)
- 回傳書籍的完整詳細資訊
- 處理找不到書籍的情況

#### ✅ 實作檢查點

完成後，你的 tool 應該能夠：

- [ ] 接受 book_id 參數
- [ ] 回傳格式化的詳細資訊
- [ ] 包含 ISBN、作者、分類、價格、庫存
- [ ] 顯示書籍描述和作者簡介
- [ ] 友善處理找不到書籍的情況
- [ ] Docstring 說明「當使用者想了解特定書籍的詳細內容時使用」

---

### 練習 2: update_stock Tool

#### 📝 建立 Proposal

請 AI 參考 OpenAPI 規範建立 proposal：

```
請參考 bookstore-openapi.json 中的 PATCH /books/{id}/stock 端點，
創建一個 OpenSpec change proposal for adding update_stock MCP tool in sdd_mcp.py。
確保參數和錯誤處理與 REST API 一致。
```

#### 📖 參考資訊

**端點**：`PATCH /books/{id}/stock`

#### 🎯 設計目標

- 接受兩個參數：`book_id` (必填) 和 `quantity_change` (必填)
- 正數表示進貨，負數表示出貨
- 處理庫存不足的情況
- 顯示更新前後的庫存數量

#### ✅ 實作檢查點

完成後，你的 tool 應該能夠：

- [ ] 接受 book_id 和 quantity_change 參數
- [ ] 處理正數（進貨）和負數（出貨）
- [ ] 在庫存不足時回傳友善錯誤訊息
- [ ] 顯示更新前後的庫存數量
- [ ] 顯示進貨或出貨的動作
- [ ] Docstring 包含使用範例（如何進貨和出貨）

---

### 練習 3: get_inventory_report Tool

#### 📝 建立 Proposal

請 AI 參考 OpenAPI 規範建立 proposal：

```
請參考 bookstore-openapi.json 中的 GET /inventory/stats 端點，
創建一個 OpenSpec change proposal for adding get_inventory_report MCP tool in sdd_mcp.py。
確保回應結構與 REST API 一致。
```

#### 📖 參考資訊

**端點**：`GET /inventory/stats`

#### 🎯 設計目標

- 無參數
- 回傳完整的庫存統計報告
- 包含總覽數據和低庫存警示
- 格式化為易讀的報表

#### ✅ 實作檢查點

完成後，你的 tool 應該能夠：

- [ ] 不需要參數即可執行
- [ ] 顯示書籍種類、總庫存、庫存總值
- [ ] 顯示低庫存和缺貨數量
- [ ] 列出需要補貨的書籍
- [ ] 使用 🔴 標示完全缺貨，🟡 標示低庫存
- [ ] 格式化數字（使用千分位逗號）
- [ ] Docstring 說明「當使用者詢問庫存狀況或需要補貨的書籍時使用」

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

## 結語

恭喜你完成了這個專案！你現在已經掌握了使用 OpenSpec 進行 Specification-Driven Development 的完整流程。

記住核心理念：**Spec First, AI Implements**

人類負責思考和設計，AI 負責實作。這樣的分工讓我們能夠專注於真正重要的事情：設計出優秀的產品規格。

---

Happy Specification-Driven Development！🚀
