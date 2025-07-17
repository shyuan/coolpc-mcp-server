[![MseeP.ai Security Assessment Badge](https://mseep.net/pr/shyuan-coolpc-mcp-server-badge.png)](https://mseep.ai/app/shyuan-coolpc-mcp-server)

# CoolPC MCP Server

一個基於 Model Context Protocol (MCP) 的伺服器，提供台灣原價屋 (CoolPC) 電腦零組件價格查詢功能，讓 Claude Desktop 等 MCP 客戶端能夠透過 AI 協助生成電腦報價單。

## 功能特色

- 🔍 **智能搜尋**: 支援關鍵字、分類、價格範圍搜尋
- 📊 **產品分析**: 提供產品規格比較和價格分析  
- 💰 **報價生成**: 根據需求自動生成電腦配置報價單
- 🔥 **即時標記**: 顯示熱銷商品、降價商品、限時優惠等標記
- 🏷️ **分類瀏覽**: 完整的產品分類和統計資訊

## 專案結構

```
coolpc-mcp-server/
├── coolpc_parser.py             # Python 解析器
├── evaluate.html               # 範例 HTML 資料
├── product-sample.json         # 範例產品資料
├── src/
│   └── index.ts               # MCP Server 主程式
├── package.json               # Node.js 相依性
├── tsconfig.json              # TypeScript 設定
└── README.md                  # 說明文件
```

## 快速開始

### 1. 環境需求

- **Python 3.x** (用於資料解析)
- **Node.js 18+** (用於 MCP Server)
- **Claude Desktop** (作為 MCP 客戶端)

### 2. 安裝相依性

```bash
# 安裝 Node.js 相依性
npm install
```

### 3. 產生 product.json

從原價屋網站取得最新資料並產生產品資料檔：

```bash
# 方法一：直接下載最新資料並解析 (推薦)
python3 coolpc_parser.py --download --json product.json

# 方法二：使用本地 HTML 檔案
# 1. 開啟瀏覽器到 https://www.coolpc.com.tw/evaluate.php
# 2. 另存網頁為 evaluate.html
# 3. 執行解析器
python3 coolpc_parser.py evaluate.html --json product.json
```

### 4. 建置 MCP Server

```bash
# 開發模式 (即時編譯)
npm run dev

# 或建置生產版本
npm run build
npm start
```

## Claude Desktop 設定

在 Claude Desktop 中設定 MCP Server，讓 Claude 能夠使用電腦零組件查詢功能。

### 設定步驟

1. **開啟 Claude Desktop 設定檔**

   找到 Claude Desktop 的設定檔位置：
   - **macOS**: `~/Library/Application Support/Claude/claude_desktop_config.json`
   - **Windows**: `%APPDATA%\\Claude\\claude_desktop_config.json`

2. **新增 MCP Server 設定**

   在設定檔中新增以下設定：

   ```json
   {
     "mcpServers": {
       "coolpc": {
         "command": "node",
         "args": ["/path/to/coolpc-mcp-server/build/index.js"],
         "env": {}
       }
     }
   }
   ```

   > 請將 `/path/to/coolpc-mcp-server` 替換為專案的實際路徑

3. **重新啟動 Claude Desktop**

   儲存設定檔後，重新啟動 Claude Desktop 讓設定生效。

### 使用範例

設定完成後，您可以在 Claude Desktop 中使用以下指令：

```
請幫我查詢 Intel CPU 的價格範圍

幫我配電腦菜單，能順跑 4K 解析度 3A 大作，並且 CP 值要非常高，儘可能節省預算

請推薦一套預算 3 萬元的遊戲電腦配置

整理一套預算 15000 元內的準系統文書機

比較 NVIDIA RTX 4070 和 RTX 4060 的規格和價格

列出所有主機板的分類和數量

查詢 AM5 腳位的 CPU，價格由低到高排序

找出 6 核心的處理器

搜尋 RTX 4060 顯示卡，價格由低到高

找出 DDR5 32GB 的記憶體

查詢 1TB 的 M.2 SSD

找 AM5 腳位的 B650 晶片組主機板
```

## 可用工具

MCP Server 提供以下工具供 Claude 使用：

### `search_products`
搜尋產品，支援關鍵字、分類、價格範圍篩選
```typescript
search_products({
  keyword?: string,    // 搜尋關鍵字
  category?: string,   // 產品分類
  minPrice?: number,   // 最低價格
  maxPrice?: number    // 最高價格
})
```

### `get_product_by_model`
根據型號取得特定產品資訊
```typescript
get_product_by_model({
  model: string       // 產品型號
})
```

### `list_categories`
列出所有產品分類及統計資訊
```typescript
list_categories()
```

### `get_category_products`
取得特定分類的所有產品
```typescript
get_category_products({
  category: string    // 分類名稱
})
```

### `search_cpu`
專門搜尋 CPU 處理器，支援腳位、核心數篩選和價格排序
```typescript
search_cpu({
  socket?: string,     // CPU 腳位 (如 'AM5', '1700', '1851', 'AM4')
  cores?: number,      // 核心數量
  sort_by?: string,    // 排序方式 ('price_asc' | 'price_desc')
  limit?: number       // 結果數量限制 (預設: 10)
})
```

### `search_gpu`
專門搜尋顯示卡，支援晶片型號、記憶體容量篩選和價格排序
```typescript
search_gpu({
  chipset?: string,    // GPU 晶片 (如 'RTX 4060', 'RTX 4070', 'RX 7600')
  memory?: number,     // 記憶體容量 (GB)
  sort_by?: string,    // 排序方式 ('price_asc' | 'price_desc')
  limit?: number       // 結果數量限制 (預設: 10)
})
```

### `search_ram`
專門搜尋記憶體，支援類型、容量、頻率篩選和價格排序
```typescript
search_ram({
  type?: string,       // 記憶體類型 (如 'DDR4', 'DDR5')
  capacity?: number,   // 總容量 (GB)
  frequency?: number,  // 頻率 (MHz，如 3200, 4800, 5600)
  sort_by?: string,    // 排序方式 ('price_asc' | 'price_desc')
  limit?: number       // 結果數量限制 (預設: 10)
})
```

### `search_ssd`
專門搜尋固態硬碟，支援介面、容量篩選和價格排序
```typescript
search_ssd({
  interface?: string,  // 介面類型 (如 'M.2', 'SATA', 'NVMe', 'PCIe')
  capacity?: number,   // 容量 (GB)
  sort_by?: string,    // 排序方式 ('price_asc' | 'price_desc')
  limit?: number       // 結果數量限制 (預設: 10)
})
```

### `search_motherboard`
專門搜尋主機板，支援腳位、晶片組、尺寸規格篩選和價格排序
```typescript
search_motherboard({
  socket?: string,     // CPU 腳位 (如 'AM5', '1700', '1851', 'AM4')
  chipset?: string,    // 晶片組 (如 'B650', 'X670', 'Z790', 'B760')
  form_factor?: string,// 尺寸規格 (如 'ATX', 'mATX', 'ITX')
  sort_by?: string,    // 排序方式 ('price_asc' | 'price_desc')
  limit?: number       // 結果數量限制 (預設: 10)
})
```

## 開發指南

### 更新產品資料

定期更新產品資料以確保價格和庫存資訊準確：

```bash
# 方法一：直接下載最新資料 (推薦)
python3 coolpc_parser.py --download --json product.json

# 方法二：手動下載 HTML 檔案
# 1. 從 https://www.coolpc.com.tw/evaluate.php 下載最新 HTML
# 2. 解析資料
python3 coolpc_parser.py evaluate.html --json product.json

# 重新建置 MCP Server
npm run build
```

### 開發模式

```bash
# 開發模式 (自動重載)
npm run dev
```

### 除錯

檢查 Claude Desktop 的 MCP 連線狀態：
1. 開啟 Claude Desktop
2. 檢查左下角是否顯示 🔌 圖示
3. 點擊查看 MCP Server 連線狀態

## 故障排除

### 常見問題

**Q: Claude Desktop 無法連接到 MCP Server**
- 檢查設定檔路徑是否正確
- 確認 Node.js 和相依性已正確安裝
- 檢查 product.json 檔案是否存在

**Q: 搜尋結果為空**
- 確認 product.json 包含資料
- 檢查搜尋關鍵字是否正確
- 嘗試使用不同的搜尋條件

**Q: 產品資料過舊**
- 重新從原價屋網站取得 HTML 資料
- 執行解析器更新 product.json
- 重新啟動 MCP Server

### 日誌檢查

```bash
# 檢查 MCP Server 輸出
npm run dev

# 檢查解析器輸出
python3 coolpc_parser.py evaluate.html --summary
```

## 授權條款

本專案僅供學習和研究用途。使用時請遵守原價屋網站的使用條款。

## 貢獻

歡迎提交 Issue 和 Pull Request 來改善這個專案。

---

**注意**: 本工具僅提供價格參考，實際價格請以原價屋官網為準。
