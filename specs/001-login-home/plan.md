# 實作計畫：登入記帳主頁功能

**分支**: `001-login-home` | **日期**: 2025-11-16 | **規格**: [spec.md](./spec.md)  
**輸入**: 功能規格文件 `/specs/001-login-home/spec.md`

## 摘要

本功能實作 Flutter 記帳 APP 的登入和首頁功能，提供 Google 和 Facebook 社群帳號登入、會話管理、自動登入、登出功能，以及首頁佔位顯示。使用 Flutter 3.16+ 搭配 Riverpod 狀態管理和 Supabase Auth 進行帳號驗證與儲存。

## 技術背景

**語言/版本**: Flutter 3.16+, Dart 3.2+  
**主要相依套件**: 
- `flutter_riverpod` ^2.4.0 (狀態管理)
- `supabase_flutter` ^2.0.0 (後端認證與資料儲存)
- `google_sign_in` ^6.1.0 (Google 登入)
- `flutter_facebook_auth` ^6.0.0 (Facebook 登入)
- `flutter_secure_storage` ^9.0.0 (本地安全儲存)

**儲存方式**: 
- 本地: Flutter Secure Storage（會話 token、使用者偏好）
- 遠端: Supabase Database（使用者資料、會話記錄）

**測試框架**: 
- flutter_test（單元測試）
- integration_test（整合測試）
- mockito ^5.4.0（Mock 測試）

**目標平台**: iOS 15+, Android API 24+ (Android 7.0+)  
**專案類型**: Mobile (Flutter)  
**效能目標**: 
- 登入流程完成時間 < 30 秒
- 自動登入載入時間 < 3 秒
- UI 渲染幀率 ≥ 60 fps

**限制條件**: 
- 必須處理網路不穩定情況
- 必須支援 iOS 和 Android 雙平台
- 必須符合 Google 和 Facebook OAuth 最佳實踐

**規模範圍**: 
- 2 個主要頁面（登入頁、首頁）
- 3 個使用者故事
- 預期程式碼量約 1,500 - 2,000 行

## 憲章檢查

*閘門: 必須在 Phase 0 研究前通過。Phase 1 設計後重新檢查。*

### I. 程式碼品質 (NON-NEGOTIABLE)

- ✅ **編碼標準**: 使用 Flutter/Dart 官方 linting 規則 (`flutter_lints`)
- ✅ **文檔**: 為公開 API 和複雜邏輯提供 dartdoc 註解
- ✅ **單一職責**: 每個 Widget、Service、Provider 維持單一職責
- ✅ **格式化**: 使用 `dart format` 進行自動格式化，pre-commit hook 強制執行
- ✅ **靜態分析**: `dart analyze` 零錯誤要求
- ✅ **命名**: 使用具描述性的變數和函式名稱
- ✅ **常數**: 所有魔術數字和字串提取為命名常數

### II. 測試標準 (NON-NEGOTIABLE)

- ✅ **TDD 流程**: 先寫測試 → 取得批准 → 測試失敗（紅） → 實作（綠） → 重構
- ✅ **覆蓋率**: 業務邏輯最低 80% 覆蓋率，認證流程 100% 覆蓋率
- ✅ **回歸測試**: 每個 bug 修復必須包含回歸測試
- ✅ **測試特性**: 快速（<100ms 單元測試）、隔離、可重複、自我驗證、及時

**測試金字塔**:
- 70% 單元測試: AuthService, SessionManager, 狀態管理邏輯
- 20% 整合測試: Supabase 整合、OAuth 流程
- 10% E2E 測試: 完整登入到首頁流程

### III. 使用者體驗一致性

- ✅ **UI/UX 模式**: 遵循 Material Design 3 規範，使用 Figma 設計規格
- ✅ **即時回饋**: 登入過程顯示 loading 狀態，成功/錯誤顯示 SnackBar
- ✅ **無障礙**: 支援 Semantics，按鈕提供明確的語義標籤
- ✅ **錯誤處理**: 顯示使用者友善的錯誤訊息（「網路連線失敗，請重試」而非技術錯誤）
- ✅ **一致術語**: 統一使用「登入」而非混用「登錄」、「登陸」
- ✅ **跨平台**: iOS 和 Android 上行為一致，視覺符合平台規範

**輸入驗證**:
- 客戶端: 即時驗證並提供回饋
- 伺服器端: Supabase Auth 進行安全驗證

### IV. 效能需求

- ✅ **初始載入**: 冷啟動 < 2 秒到登入頁面
- ✅ **互動時間**: 點擊登入按鈕到 OAuth 頁面 < 1 秒
- ✅ **API 回應**: Supabase 認證 API < 500ms (P95)
- ✅ **畫面流暢度**: 維持 60fps，避免 UI 阻塞
- ✅ **資源使用**: APP 記憶體使用 < 100MB（閒置狀態）

**效能實踐**:
- 使用 FutureProvider 和 StreamProvider 避免不必要的重建
- 圖片資源使用適當尺寸和格式（PNG/WebP）
- 延遲載入非關鍵資源
- 使用 const Widget 減少重建

### V. 可維護性與簡潔性

- ✅ **簡單優先**: 實作當前需求，避免過度設計（YAGNI）
- ✅ **組合優於繼承**: 使用 Widget 組合和 Riverpod Provider 組合
- ✅ **最小相依**: 只使用必要的套件，定期檢查更新
- ✅ **外部化設定**: 環境變數（Supabase URL/Key）外部化
- ✅ **刪除未使用程式碼**: 不保留註解掉的程式碼

**文件需求**:
- README: 包含環境設定、套件安裝、執行指示
- 架構決策記錄 (ADR): 記錄重要技術選擇（為何使用 Riverpod、Supabase）
- API 文件: dartdoc 自動生成
- 註解: 解釋「為何」而非「做什麼」

### 閘門評估

**狀態**: ✅ 通過所有閘門

- 無違反憲章的設計決策
- 技術棧符合簡潔性原則
- 測試策略符合 TDD 要求
- 效能目標可衡量且合理

## 專案結構

### 文件結構（本功能）

```text
specs/001-login-home/
├── plan.md              # 本文件 (/speckit.plan 指令輸出)
├── spec.md              # 功能規格文件
├── research.md          # Phase 0 輸出（技術研究）
├── data-model.md        # Phase 1 輸出（資料模型）
├── quickstart.md        # Phase 1 輸出（快速開始指南）
├── contracts/           # Phase 1 輸出（API 合約）
│   ├── auth_api.md      # Supabase Auth API 合約
│   └── user_api.md      # 使用者資料 API 合約
├── checklists/          # 品質檢查清單
│   └── requirements.md  # 需求檢查清單
├── clarification-report-2025-11-16.md  # 澄清報告
└── tasks.md             # Phase 2 輸出 (/speckit.tasks 指令 - 未由 /speckit.plan 建立)
```

### 原始碼結構（Repository Root）

```text
lib/
├── main.dart                          # APP 入口
├── app.dart                           # APP Widget（路由、主題設定）
├── core/
│   ├── constants/
│   │   ├── design_tokens.dart         # 設計 Token（色彩、間距、文字樣式）
│   │   └── api_constants.dart         # API 常數（Supabase URL）
│   ├── theme/
│   │   └── app_theme.dart             # Material 3 主題設定
│   ├── router/
│   │   └── app_router.dart            # go_router 路由設定
│   └── utils/
│       ├── logger.dart                # 日誌工具
│       └── validators.dart            # 驗證工具
├── features/
│   └── auth/
│       ├── data/
│       │   ├── models/
│       │   │   ├── user_model.dart              # 使用者資料模型
│       │   │   ├── session_model.dart           # 會話資料模型
│       │   │   └── auth_provider_model.dart     # 登入提供商模型
│       │   ├── repositories/
│       │   │   ├── auth_repository.dart         # 認證資料庫操作
│       │   │   └── session_repository.dart      # 會話資料庫操作
│       │   └── datasources/
│       │       ├── supabase_auth_datasource.dart    # Supabase Auth 資料源
│       │       └── local_storage_datasource.dart    # 本地儲存資料源
│       ├── domain/
│       │   ├── entities/
│       │   │   ├── user.dart          # 使用者實體
│       │   │   └── session.dart       # 會話實體
│       │   ├── repositories/
│       │   │   └── auth_repository_interface.dart  # Repository 介面
│       │   └── usecases/
│       │       ├── sign_in_with_google.dart     # Google 登入用例
│       │       ├── sign_in_with_facebook.dart   # Facebook 登入用例
│       │       ├── sign_out.dart                # 登出用例
│       │       └── check_auth_status.dart       # 檢查認證狀態用例
│       └── presentation/
│           ├── providers/
│           │   ├── auth_state_provider.dart     # 認證狀態 Provider
│           │   └── session_provider.dart        # 會話狀態 Provider
│           ├── pages/
│           │   ├── login_page.dart              # 登入頁面
│           │   └── home_page.dart               # 首頁
│           └── widgets/
│               ├── google_sign_in_button.dart   # Google 登入按鈕
│               ├── facebook_sign_in_button.dart # Facebook 登入按鈕
│               └── under_construction_widget.dart # 施工中佔位元件
├── shared/
│   └── widgets/
│       ├── loading_overlay.dart       # Loading 遮罩元件
│       └── error_snackbar.dart        # 錯誤提示元件
└── firebase_options.dart              # Firebase 設定（如需使用 Firebase）

test/
├── unit/
│   ├── auth_repository_test.dart
│   ├── sign_in_with_google_test.dart
│   └── auth_state_provider_test.dart
├── integration/
│   ├── supabase_auth_integration_test.dart
│   └── session_persistence_test.dart
└── widget/
    ├── login_page_test.dart
    └── home_page_test.dart

integration_test/
└── login_flow_test.dart               # E2E 登入流程測試

design-assets/                         # Figma 設計資源
├── google_button.png
├── google_logo.png
├── facebook_button.png
├── facebook_logo.png
├── home_page.png
└── login_page.png
```

**結構決策**: 採用 Feature-First 架構結合 Clean Architecture 原則。`features/auth/` 包含完整的認證功能，分為三層：
- **Data**: 處理資料來源（Supabase、本地儲存）
- **Domain**: 定義業務邏輯和實體（獨立於框架）
- **Presentation**: UI 和狀態管理（Riverpod Providers、Pages、Widgets）

此結構便於測試、維護和未來擴充新功能。

## 複雜度追蹤

*僅在憲章檢查有違規需要說明時填寫*

| 違規項目 | 為何需要 | 更簡單替代方案被拒絕的原因 |
|---------|---------|---------------------------|
| 無違規 | - | - |

## Phase 0: 概述與研究

### 待研究項目

基於技術背景的 NEEDS CLARIFICATION 項目：

1. **Supabase Auth 整合最佳實踐**
   - 研究內容: Supabase Auth 與 Flutter 的整合方式、OAuth 設定、RLS (Row Level Security) 政策
   - 研究原因: 需要確保安全且符合最佳實踐的認證流程

2. **Riverpod 狀態管理模式**
   - 研究內容: Riverpod 2.x 最佳實踐、Provider 組合模式、錯誤處理策略
   - 研究原因: 選擇適合的狀態管理模式以確保可維護性

3. **多裝置會話管理策略**
   - 研究內容: 如何在 Supabase 中實作獨立會話、裝置識別碼生成
   - 研究原因: 需支援多裝置同時登入，每個裝置獨立會話

4. **Flutter 社群登入套件比較**
   - 研究內容: `google_sign_in` vs Firebase Auth、`flutter_facebook_auth` 最佳實踐
   - 研究原因: 選擇穩定且維護良好的套件

5. **本地儲存安全方案**
   - 研究內容: `flutter_secure_storage` 在不同平台的安全性、備用方案（shared_preferences）
   - 研究原因: 確保會話 token 和使用者偏好安全儲存

### 研究任務分派

**任務 1**: Supabase Auth 整合研究
- 輸出: Supabase Auth 設定步驟、OAuth 供應商配置、RLS 政策範例
- 參考: Supabase 官方文件、Flutter Supabase 套件文件

**任務 2**: Riverpod 狀態管理模式研究
- 輸出: Riverpod Provider 架構設計、AsyncNotifier 使用模式、錯誤狀態處理
- 參考: Riverpod 官方文件、社群最佳實踐

**任務 3**: 多裝置會話管理研究
- 輸出: 資料庫 schema 設計、裝置識別碼生成策略、會話清理政策
- 參考: Supabase Auth 文件、JWT 最佳實踐

**任務 4**: 社群登入套件評估
- 輸出: 套件比較表、推薦套件清單、整合範例
- 參考: pub.dev 評分、GitHub Issues、社群回饋

**任務 5**: 本地儲存安全評估
- 輸出: 平台安全性分析、推薦儲存方案、降級策略
- 參考: Flutter 安全最佳實踐、平台文件

**輸出**: 所有研究結果整合至 `research.md`，格式為：
- **決策**: 選擇的方案
- **理由**: 為何選擇此方案
- **考慮的替代方案**: 其他評估過的選項

## Phase 1: 設計與合約

**前置條件**: `research.md` 完成

### 1. 資料模型生成

從功能規格提取實體 → 輸出至 `data-model.md`:

#### 使用者 (User)
- **欄位**: 
  - `id` (String): Supabase Auth UID
  - `email` (String): 電子郵件
  - `displayName` (String?): 顯示名稱
  - `photoUrl` (String?): 頭像 URL
  - `authProvider` (AuthProvider): 登入提供商（GOOGLE/FACEBOOK）
  - `createdAt` (DateTime): 建立時間
  - `updatedAt` (DateTime): 更新時間
- **關聯**: 一對多 Session（一個使用者可有多個會話）
- **驗證規則**: 
  - `email` 必須符合 email 格式
  - `authProvider` 必須為 GOOGLE 或 FACEBOOK

#### 會話 (Session)
- **欄位**:
  - `id` (String): 會話 ID
  - `userId` (String): 關聯的使用者 ID（外鍵）
  - `deviceId` (String): 裝置識別碼
  - `accessToken` (String): 存取 Token
  - `refreshToken` (String): 更新 Token
  - `expiresAt` (DateTime): 過期時間
  - `lastActivityAt` (DateTime): 最後活動時間
  - `createdAt` (DateTime): 建立時間
- **關聯**: 多對一 User
- **驗證規則**:
  - `expiresAt` 必須大於當前時間
  - `deviceId` 不可為空

#### 登入提供商 (AuthProvider)
- **類型**: Enum
- **值**: `GOOGLE`, `FACEBOOK`
- **用途**: 標識使用者使用的登入方式，用於會話過期後記住偏好

**狀態轉換**:
- 使用者: 未登入 → 登入中 → 已登入 → 登出
- 會話: 有效 → 即將過期 → 已過期

### 2. API 合約生成

從功能需求生成端點 → 輸出至 `/contracts/`:

#### `contracts/auth_api.md` - 認證 API

```markdown
# 認證 API 合約

## 1. Google 登入
- **端點**: Supabase Auth - `signInWithOAuth`
- **方法**: OAuth 2.0
- **供應商**: Google
- **請求**: 
  ```dart
  await supabase.auth.signInWithOAuth(OAuthProvider.google)
  ```
- **回應**: 
  - 成功: `AuthResponse` 包含 `user`, `session`
  - 失敗: `AuthException` 包含錯誤訊息
- **錯誤碼**:
  - `user_cancelled`: 使用者取消授權
  - `network_error`: 網路錯誤
  - `invalid_credentials`: 無效憑證

## 2. Facebook 登入
- **端點**: Supabase Auth - `signInWithOAuth`
- **方法**: OAuth 2.0
- **供應商**: Facebook
- **請求**: 
  ```dart
  await supabase.auth.signInWithOAuth(OAuthProvider.facebook)
  ```
- **回應**: 同 Google 登入

## 3. 登出
- **端點**: Supabase Auth - `signOut`
- **方法**: POST
- **請求**: 
  ```dart
  await supabase.auth.signOut()
  ```
- **回應**: 
  - 成功: `void`
  - 失敗: `AuthException`

## 4. 檢查認證狀態
- **端點**: Supabase Auth - `currentSession`
- **方法**: GET
- **請求**: 
  ```dart
  final session = supabase.auth.currentSession
  ```
- **回應**: `Session?`（null 表示未登入）
```

#### `contracts/user_api.md` - 使用者資料 API

```markdown
# 使用者資料 API 合約

## 1. 取得使用者資料
- **端點**: `/rest/v1/users`
- **方法**: GET
- **查詢**: `?id=eq.{userId}`
- **回應**: 
  ```json
  {
    "id": "uuid",
    "email": "user@example.com",
    "display_name": "User Name",
    "photo_url": "https://...",
    "auth_provider": "GOOGLE",
    "created_at": "2025-11-16T...",
    "updated_at": "2025-11-16T..."
  }
  ```

## 2. 建立/更新使用者資料
- **端點**: `/rest/v1/users`
- **方法**: POST/PATCH
- **請求 Body**: 
  ```json
  {
    "id": "uuid",
    "email": "user@example.com",
    "display_name": "User Name",
    "photo_url": "https://...",
    "auth_provider": "GOOGLE"
  }
  ```
- **回應**: 同取得使用者資料

## 3. 記錄最後使用的登入提供商
- **端點**: `/rest/v1/user_preferences`
- **方法**: POST/PATCH
- **請求 Body**:
  ```json
  {
    "user_id": "uuid",
    "last_auth_provider": "GOOGLE"
  }
  ```
```

### 3. Agent 背景更新

執行腳本更新 agent 背景：

```bash
cd /Users/chunchun/文件/speckit/money-manager
.specify/scripts/bash/update-agent-context.sh copilot
```

此腳本會：
- 偵測當前使用的 AI agent（Copilot）
- 更新 `.github/copilot-instructions.md`
- 新增本次計畫使用的技術（Flutter, Riverpod, Supabase）
- 保留手動新增的內容（在標記之間）

### 4. 快速開始指南

輸出至 `quickstart.md`:

```markdown
# 快速開始：登入記帳主頁功能

## 前置需求

- Flutter SDK 3.16+
- Dart 3.2+
- iOS: Xcode 15+, CocoaPods
- Android: Android Studio, JDK 11+
- Supabase 帳號

## 環境設定

1. **安裝 Flutter**
   ```bash
   flutter doctor
   ```

2. **複製專案並安裝相依套件**
   ```bash
   git clone https://github.com/chunchun1213/money-manager.git
   cd money-manager
   git checkout 001-login-home
   flutter pub get
   ```

3. **設定 Supabase**
   - 建立 Supabase 專案: https://supabase.com
   - 啟用 Google OAuth 供應商（需要 Google Cloud Client ID）
   - 啟用 Facebook OAuth 供應商（需要 Facebook App ID）
   - 複製 Supabase URL 和 Anon Key

4. **設定環境變數**
   建立 `.env` 檔案：
   ```
   SUPABASE_URL=https://your-project.supabase.co
   SUPABASE_ANON_KEY=your-anon-key
   ```

5. **執行 APP**
   ```bash
   # iOS
   flutter run -d ios

   # Android
   flutter run -d android
   ```

## 測試

```bash
# 單元測試
flutter test

# 整合測試
flutter test integration_test/
```

## 常見問題

**Q: Google 登入失敗怎麼辦？**  
A: 確認 Google Cloud Console 已正確設定 OAuth 2.0 Client ID，並將 Redirect URI 新增至授權清單。

**Q: Supabase 連線錯誤？**  
A: 檢查 `.env` 檔案的 URL 和 Key 是否正確，確認網路連線正常。
```

**輸出**: `data-model.md`, `/contracts/*`, `quickstart.md`, `.github/copilot-instructions.md` 已更新

### 憲章重新檢查

**Phase 1 設計後評估**: ✅ 通過

- 資料模型符合 YAGNI 原則，只包含必要欄位
- API 合約使用標準 REST 模式
- 無新增的複雜度或違規項目

## Phase 2 規劃

**注意**: 本階段由 `/speckit.tasks` 指令執行，不在 `/speckit.plan` 範圍內。

Phase 2 將：
1. 將實作工作分解為可追蹤的任務
2. 建立 `tasks.md` 包含所有開發任務
3. 估算每個任務的時間和優先級
4. 定義任務相依性和里程碑

## 總結與產出

### 完成的產出

- ✅ `plan.md` - 本實作計畫文件
- ⏳ `research.md` - 待 Phase 0 完成
- ⏳ `data-model.md` - 待 Phase 1 完成
- ⏳ `contracts/auth_api.md` - 待 Phase 1 完成
- ⏳ `contracts/user_api.md` - 待 Phase 1 完成
- ⏳ `quickstart.md` - 待 Phase 1 完成
- ⏳ `.github/copilot-instructions.md` - 待 Phase 1 更新

### 下一步

1. 執行 Phase 0 研究任務，產生 `research.md`
2. 執行 Phase 1 設計任務，產生資料模型和 API 合約
3. 執行 `/speckit.tasks` 建立任務清單
4. 開始 TDD 實作流程

### 關鍵決策記錄

- **狀態管理**: 選擇 Riverpod（理由：類型安全、測試友善、效能優秀）
- **後端服務**: 選擇 Supabase（理由：整合 Auth、Database、即時功能，快速開發）
- **架構模式**: Feature-First + Clean Architecture（理由：可維護性、可測試性、擴充性）
- **設計系統**: Material 3（理由：現代、符合 Flutter 最佳實踐、跨平台一致）

---

**分支**: `001-login-home`  
**專案**: money-manager  
**計畫路徑**: `/Users/chunchun/文件/speckit/money-manager/specs/001-login-home/plan.md`
