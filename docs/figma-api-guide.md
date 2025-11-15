# Figma API 使用指南

## 概述

本指南說明如何使用 Figma REST API 來獲取設計檔案資訊、匯出資源和生成前端規格書。

## 前置需求

- Figma 帳號
- Figma API Token
- curl 或任何 HTTP 客戶端

## 取得 Figma API Token

1. 前往 [Figma](https://www.figma.com/)
2. 登入您的帳號
3. 進入 **Settings** → **Account** → **Personal Access Tokens**
4. 點擊 **Generate new token**
5. 輸入描述（例如：「Money Manager 專案」）
6. 複製生成的 Token 並妥善保存

## 環境變數設定

建議將 Token 設定為環境變數：

```bash
# 在 ~/.zshrc 或 ~/.bash_profile 中加入
export FIGMA_TOKEN="your_figma_token_here"

# 重新載入設定
source ~/.zshrc
```

## 常用 API 端點

### 1. 獲取檔案資訊

```bash
curl -H "X-Figma-Token: $FIGMA_TOKEN" \
  "https://api.figma.com/v1/files/{file_key}"
```

### 2. 獲取檔案節點資訊

```bash
curl -H "X-Figma-Token: $FIGMA_TOKEN" \
  "https://api.figma.com/v1/files/{file_key}/nodes?ids={node_id}"
```

### 3. 匯出圖片

```bash
curl -H "X-Figma-Token: $FIGMA_TOKEN" \
  "https://api.figma.com/v1/images/{file_key}?ids={node_id}&format=png&scale=2"
```

支援格式：`png`, `jpg`, `svg`, `pdf`

### 4. 獲取樣式資訊

```bash
curl -H "X-Figma-Token: $FIGMA_TOKEN" \
  "https://api.figma.com/v1/files/{file_key}/styles"
```

## 從 URL 提取 File Key

Figma URL 格式：
```
https://www.figma.com/design/{file_key}/{file_name}?node-id={node_id}
```

例如：
```
https://www.figma.com/design/Mfp1UVqT4L2TrkEmhhXkol/家庭記帳APP?node-id=0-1
```

File Key: `Mfp1UVqT4L2TrkEmhhXkol`

## 實用腳本範例

### 批次匯出所有圖片

```bash
#!/bin/bash
FIGMA_TOKEN="your_token"
FILE_KEY="your_file_key"
OUTPUT_DIR="./design-assets"

mkdir -p "$OUTPUT_DIR"

# 獲取所有節點 ID
node_ids=$(curl -s -H "X-Figma-Token: $FIGMA_TOKEN" \
  "https://api.figma.com/v1/files/$FILE_KEY" | \
  jq -r '.document.children[].id')

# 匯出每個節點
for node_id in $node_ids; do
  image_url=$(curl -s -H "X-Figma-Token: $FIGMA_TOKEN" \
    "https://api.figma.com/v1/images/$FILE_KEY?ids=$node_id&format=png&scale=2" | \
    jq -r ".images[\"$node_id\"]")
  
  if [ "$image_url" != "null" ]; then
    curl -o "$OUTPUT_DIR/${node_id}.png" "$image_url"
    echo "已匯出: ${node_id}.png"
  fi
done
```

### 生成設計規格 JSON

```bash
#!/bin/bash
FIGMA_TOKEN="your_token"
FILE_KEY="your_file_key"
OUTPUT_FILE="design-spec.json"

curl -s -H "X-Figma-Token: $FIGMA_TOKEN" \
  "https://api.figma.com/v1/files/$FILE_KEY" | \
  jq '{
    name: .name,
    version: .version,
    lastModified: .lastModified,
    pages: [.document.children[] | {
      id: .id,
      name: .name,
      type: .type,
      frames: [.children[]? | {
        id: .id,
        name: .name,
        width: .absoluteBoundingBox.width,
        height: .absoluteBoundingBox.height
      }]
    }]
  }' > "$OUTPUT_FILE"

echo "規格已儲存至 $OUTPUT_FILE"
```

## API 限制與最佳實踐

### 速率限制
- 每分鐘最多 100 個請求
- 建議在批次操作間加入延遲

### 快取
- 使用 `version` 欄位判斷檔案是否更新
- 快取 API 回應以減少請求次數

### 錯誤處理

常見錯誤碼：
- `400` - 錯誤的請求參數
- `403` - Token 無效或權限不足
- `404` - 檔案不存在
- `429` - 超過速率限制

## 安全性注意事項

⚠️ **重要安全提醒**:
- **絕不**將 Token 提交到版本控制系統
- 使用環境變數或密鑰管理工具
- 定期輪換 API Token
- 限制 Token 的存取權限

## 相關資源

- [Figma API 官方文件](https://www.figma.com/developers/api)
- [Figma API 速率限制](https://www.figma.com/developers/api#ratelimiting)
- [Figma Community 外掛](https://www.figma.com/community/plugins)

## 版本資訊

- **建立日期**: 2025-11-15
- **最後更新**: 2025-11-15
- **API 版本**: v1

---

如有任何問題或需要協助,請參考 Figma 官方文件或專案的 issue tracker。
