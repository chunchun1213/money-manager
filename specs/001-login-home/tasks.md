# 任務清單：登入記帳主頁功能

**輸入**: 設計文件來自 `/specs/001-login-home/`  
**前置條件**: plan.md（必要）, spec.md（必要）, research.md, data-model.md, contracts/

**測試**: 本專案遵循 TDD 原則，測試任務已包含在內

**組織方式**: 任務依使用者故事分組，以便獨立實作和測試每個故事

## 格式: `[ID] [P?] [Story] 描述`

- **[P]**: 可平行執行（不同檔案，無相依性）
- **[Story]**: 此任務屬於哪個使用者故事（如 US1, US2, US3）
- 描述中包含確切的檔案路徑

## 路徑慣例

本專案採用 Flutter Feature-First 架構：
- **lib/**: 應用程式原始碼
- **test/**: 測試程式碼
- **integration_test/**: E2E 測試
- **design-assets/**: 設計資源

---

## Phase 1: 設定（專案初始化）

**目的**: 專案初始化和基礎結構

- [ ] T001 建立 Flutter 專案結構依照 plan.md 的架構設計
- [ ] T002 初始化 pubspec.yaml 並新增主要相依套件（riverpod, supabase_flutter, google_sign_in, flutter_facebook_auth, flutter_secure_storage, freezed）
- [ ] T003 [P] 設定 flutter_lints 和 analysis_options.yaml 進行程式碼品質檢查
- [ ] T004 [P] 建立 .env 檔案模板並配置環境變數載入（使用 flutter_dotenv）
- [ ] T005 [P] 設定 iOS Info.plist 包含 Google/Facebook OAuth 和 Supabase Deep Link 設定
- [ ] T006 [P] 設定 Android AndroidManifest.xml 和 strings.xml 包含 Facebook SDK 設定
- [ ] T007 建立 lib/core/constants/design_tokens.dart 定義色彩、間距、文字樣式常數
- [ ] T008 [P] 建立 lib/core/constants/api_constants.dart 定義 Supabase URL 和 Key
- [ ] T009 [P] 建立 lib/core/theme/app_theme.dart 實作 Material 3 主題
- [ ] T010 執行 flutter pub run build_runner build 生成 Freezed 程式碼

**檢查點**: 專案結構完成，相依套件已安裝，環境設定就緒

---

## Phase 2: 基礎建設（必要前置作業）

**目的**: 核心基礎設施，必須在任何使用者故事實作前完成

**⚠️ 關鍵**: 此階段完成前，無法開始任何使用者故事的工作

- [ ] T011 在 lib/main.dart 初始化 Supabase 客戶端並設定 ProviderScope
- [ ] T012 [P] 建立 lib/core/utils/logger.dart 實作日誌工具
- [ ] T013 [P] 建立 lib/shared/widgets/loading_overlay.dart 實作載入遮罩元件
- [ ] T014 [P] 建立 lib/shared/widgets/error_snackbar.dart 實作錯誤提示元件
- [ ] T015 建立 lib/features/auth/data/models/auth_provider_model.dart 定義 AuthProvider 列舉（GOOGLE/FACEBOOK）
- [ ] T016 [P] 建立 lib/features/auth/data/datasources/local_storage_datasource.dart 封裝 FlutterSecureStorage 和 SharedPreferences
- [ ] T017 執行 Supabase 資料庫遷移腳本建立 users, sessions, user_preferences 表格（參考 data-model.md）

**檢查點**: 基礎設施就緒 - 使用者故事實作現在可以平行開始

---

## Phase 3: 使用者故事 1 - 社群帳號登入 (優先級: P1) 🎯 MVP

**目標**: 使用者可以透過 Google 或 Facebook 帳號登入，系統記住登入狀態並支援自動登入

**獨立測試**: 點擊 Google/Facebook 登入按鈕 → 完成 OAuth 授權 → 自動導向首頁 → 關閉 APP 重新開啟 → 自動進入首頁（無需重新登入）

### 使用者故事 1 的測試 ✅

> **注意: 先寫這些測試，確保它們失敗後才開始實作**

- [ ] T018 [P] [US1] 建立 test/unit/auth/data/repositories/auth_repository_test.dart 測試 Google 和 Facebook 登入邏輯
- [ ] T019 [P] [US1] 建立 test/unit/auth/domain/usecases/sign_in_with_google_test.dart 測試 Google 登入用例
- [ ] T020 [P] [US1] 建立 test/unit/auth/domain/usecases/sign_in_with_facebook_test.dart 測試 Facebook 登入用例
- [ ] T021 [P] [US1] 建立 test/unit/auth/domain/usecases/check_auth_status_test.dart 測試認證狀態檢查用例
- [ ] T022 [P] [US1] 建立 test/unit/auth/presentation/providers/auth_state_provider_test.dart 測試認證狀態 Provider
- [ ] T023 [P] [US1] 建立 test/widget/auth/presentation/pages/login_page_test.dart 測試登入頁面 Widget
- [ ] T024 [P] [US1] 建立 test/integration/auth/supabase_auth_integration_test.dart 測試 Supabase Auth 整合
- [ ] T025 [US1] 建立 integration_test/login_flow_test.dart 測試完整登入到首頁的 E2E 流程

### 使用者故事 1 的實作

**資料層 (Data Layer)**

- [ ] T026 [P] [US1] 建立 lib/features/auth/data/models/user_model.dart 使用 Freezed 定義 User 資料模型（包含 fromJson/toJson）
- [ ] T027 [P] [US1] 建立 lib/features/auth/data/models/session_model.dart 使用 Freezed 定義 Session 資料模型（包含 isExpired/isExpiringSoon 擴充方法）
- [ ] T028 [US1] 建立 lib/features/auth/data/datasources/supabase_auth_datasource.dart 實作 Supabase OAuth 登入（signInWithGoogle, signInWithFacebook）
- [ ] T029 [US1] 建立 lib/features/auth/data/repositories/auth_repository_impl.dart 實作 AuthRepository（整合 Supabase 和本地儲存）
- [ ] T030 [US1] 建立 lib/features/auth/data/repositories/session_repository.dart 實作會話記錄和管理（recordSession, getCurrentSession）

**領域層 (Domain Layer)**

- [ ] T031 [P] [US1] 建立 lib/features/auth/domain/entities/user.dart 定義 User 實體
- [ ] T032 [P] [US1] 建立 lib/features/auth/domain/entities/session.dart 定義 Session 實體
- [ ] T033 [P] [US1] 建立 lib/features/auth/domain/entities/auth_state.dart 定義認證狀態（Unauthenticated, Authenticating, Authenticated, AuthenticationError）
- [ ] T034 [US1] 建立 lib/features/auth/domain/repositories/auth_repository_interface.dart 定義 AuthRepository 介面
- [ ] T035 [P] [US1] 建立 lib/features/auth/domain/usecases/sign_in_with_google.dart 實作 Google 登入用例
- [ ] T036 [P] [US1] 建立 lib/features/auth/domain/usecases/sign_in_with_facebook.dart 實作 Facebook 登入用例
- [ ] T037 [US1] 建立 lib/features/auth/domain/usecases/check_auth_status.dart 實作檢查認證狀態用例（含自動更新 token）

**展示層 (Presentation Layer)**

- [ ] T038 [US1] 建立 lib/features/auth/presentation/providers/auth_state_provider.dart 實作 Riverpod AsyncNotifierProvider 管理認證狀態
- [ ] T039 [US1] 建立 lib/features/auth/presentation/providers/auth_repository_provider.dart 提供 AuthRepository 實例
- [ ] T040 [P] [US1] 建立 lib/features/auth/presentation/widgets/google_sign_in_button.dart 實作 Google 登入按鈕（包含 Logo 和樣式）
- [ ] T041 [P] [US1] 建立 lib/features/auth/presentation/widgets/facebook_sign_in_button.dart 實作 Facebook 登入按鈕（包含 Logo 和樣式）
- [ ] T042 [US1] 建立 lib/features/auth/presentation/pages/login_page.dart 實作登入頁面（顯示 Google 和 Facebook 登入按鈕）
- [ ] T043 [US1] 更新 lib/main.dart 加入路由邏輯（檢查認證狀態決定顯示登入頁或首頁）

**整合與優化**

- [ ] T044 [US1] 實作錯誤處理（將 AuthException 轉換為使用者友善訊息）
- [ ] T045 [US1] 加入 loading 狀態顯示（登入過程顯示 LoadingOverlay）
- [ ] T046 [US1] 實作會話記錄到 Supabase sessions 表格（登入成功後記錄裝置資訊）
- [ ] T047 [US1] 實作記住上次登入提供商（儲存至 user_preferences 表格）
- [ ] T048 [US1] 執行所有使用者故事 1 的測試，確保通過

**檢查點**: 此時，使用者故事 1 應該完全功能正常且可獨立測試

---

## Phase 4: 使用者故事 2 - 登出功能 (優先級: P2)

**目標**: 已登入的使用者可以從首頁登出，清除登入狀態並返回登入頁面

**獨立測試**: 在首頁點擊登出按鈕 → 返回登入頁面 → 關閉 APP 重新開啟 → 顯示登入頁面（不自動登入）

### 使用者故事 2 的測試 ✅

- [ ] T049 [P] [US2] 建立 test/unit/auth/domain/usecases/sign_out_test.dart 測試登出用例
- [ ] T050 [P] [US2] 建立 test/widget/auth/presentation/pages/home_page_test.dart 測試首頁包含登出按鈕
- [ ] T051 [US2] 建立 test/integration/auth/sign_out_flow_test.dart 測試完整登出流程

### 使用者故事 2 的實作

- [ ] T052 [US2] 建立 lib/features/auth/domain/usecases/sign_out.dart 實作登出用例（清除 Supabase Auth 會話、刪除本地 token、保留 lastAuthProvider）
- [ ] T053 [US2] 在 lib/features/auth/presentation/providers/auth_state_provider.dart 加入 signOut 方法
- [ ] T054 [US2] 建立 lib/features/auth/presentation/pages/home_page.dart 實作首頁（包含登出按鈕和 Header）
- [ ] T055 [US2] 更新 lib/main.dart 路由邏輯支援登出後導向登入頁
- [ ] T056 [US2] 實作登出後清除會話記錄（從 Supabase Database sessions 表格刪除當前裝置的會話記錄）
- [ ] T057 [US2] 執行所有使用者故事 2 的測試，確保通過

**檢查點**: 此時，使用者故事 1 和 2 應該都能獨立運作

---

## Phase 5: 使用者故事 3 - 首頁佔位顯示 (優先級: P3)

**目標**: 使用者成功登入後進入首頁，首頁顯示「施工中」圖示和說明文字

**獨立測試**: 登入成功後檢查首頁是否顯示施工中圖示、說明文字和登出按鈕

### 使用者故事 3 的測試 ✅

- [ ] T058 [P] [US3] 建立 test/widget/auth/presentation/widgets/under_construction_widget_test.dart 測試施工中元件
- [ ] T059 [US3] 更新 test/widget/auth/presentation/pages/home_page_test.dart 驗證首頁包含施工中元件

### 使用者故事 3 的實作

- [ ] T060 [US3] 建立 lib/features/auth/presentation/widgets/under_construction_widget.dart 實作施工中元件（圖示 + 說明文字）
- [ ] T061 [US3] 更新 lib/features/auth/presentation/pages/home_page.dart 加入施工中元件（使用 design_tokens.dart 的樣式）
- [ ] T062 [US3] 實作首頁 Header 元件（背景色 #86efcc，右上角登出按鈕）
- [ ] T063 [US3] 執行所有使用者故事 3 的測試，確保通過

**檢查點**: 所有使用者故事現在應該都能獨立運作

---

## Phase 6: 精緻化與跨領域關注

**目的**: 影響多個使用者故事的改進

- [ ] T064 [P] 實作會話過期處理（自動更新 token 或導向登入頁並突出顯示上次使用的提供商）
- [ ] T065 [P] 加入網路錯誤處理（顯示友善錯誤訊息並提供重試選項）
- [ ] T066 [P] 實作授權取消處理（使用者在 OAuth 頁面取消時返回登入頁並顯示提示）
- [ ] T067 [P] 加入 Semantics 標籤支援無障礙功能（所有按鈕和互動元件）
- [ ] T068 [P] 優化 UI 效能（使用 const Widget、避免不必要的重建）
- [ ] T069 [P] 更新 README.md 包含專案設定和執行指示
- [ ] T070 [P] 建立 docs/architecture.md 記錄架構決策（為何選擇 Riverpod、Supabase、Clean Architecture）
- [ ] T071 執行 flutter analyze 確保無靜態分析錯誤
- [ ] T072 執行 flutter test --coverage 確保測試覆蓋率達標（業務邏輯 ≥ 80%，認證流程 = 100%）
- [ ] T073 執行 quickstart.md 中的驗證步驟確保環境設定正確
- [ ] T074 程式碼重構和清理（移除未使用的 import、統一命名風格）
- [ ] T075 安全性強化（確認 token 加密儲存、RLS 政策正確設定）
- [ ] T076 [P] 建立 integration_test/performance/login_performance_test.dart 驗證登入流程時間 < 30 秒（SC-001）
- [ ] T077 [P] 建立 integration_test/performance/auto_login_performance_test.dart 驗證自動登入時間 < 3 秒（SC-003）
- [ ] T078 [P] 建立 test/widget/responsive_design_test.dart 驗證登入頁和首頁在各螢幕尺寸（手機、平板）正確顯示（SC-006）

---

## 相依性與執行順序

### 階段相依性

- **設定 (Phase 1)**: 無相依性 - 可立即開始
- **基礎建設 (Phase 2)**: 依賴設定完成 - 阻塞所有使用者故事
- **使用者故事 (Phase 3-5)**: 全部依賴基礎建設階段完成
  - 使用者故事可以平行進行（如果有足夠人力）
  - 或依優先順序依序進行（P1 → P2 → P3）
- **精緻化 (Final Phase)**: 依賴所有期望的使用者故事完成

### 使用者故事相依性

- **使用者故事 1 (P1)**: 基礎建設 (Phase 2) 完成後即可開始 - 不依賴其他故事
- **使用者故事 2 (P2)**: 基礎建設 (Phase 2) 完成後即可開始 - 可與 US1 整合但應可獨立測試
- **使用者故事 3 (P3)**: 基礎建設 (Phase 2) 完成後即可開始 - 與 US2 輕度耦合（使用同一個首頁）但應可獨立測試

### 每個使用者故事內部

- 測試必須先寫並且失敗後才開始實作
- 模型先於服務
- 服務先於用例
- 用例先於 Providers
- Providers 先於 UI
- 核心實作先於整合
- 故事完成後才移至下一個優先級

### 平行執行機會

- 所有標記 [P] 的設定任務可以平行執行
- 所有標記 [P] 的基礎建設任務可以平行執行（在 Phase 2 內）
- 基礎建設階段完成後，所有使用者故事可以平行開始（如果團隊容量允許）
- 一個使用者故事內標記 [P] 的測試可以平行執行
- 一個使用者故事內標記 [P] 的模型可以平行執行
- 不同使用者故事可以由不同團隊成員平行處理

---

## 平行執行範例: 使用者故事 1

```bash
# 一起啟動使用者故事 1 的所有測試:
Task T018: "建立 test/unit/auth/data/repositories/auth_repository_test.dart"
Task T019: "建立 test/unit/auth/domain/usecases/sign_in_with_google_test.dart"
Task T020: "建立 test/unit/auth/domain/usecases/sign_in_with_facebook_test.dart"
Task T021: "建立 test/unit/auth/domain/usecases/check_auth_status_test.dart"
Task T022: "建立 test/unit/auth/presentation/providers/auth_state_provider_test.dart"
Task T023: "建立 test/widget/auth/presentation/pages/login_page_test.dart"
Task T024: "建立 test/integration/auth/supabase_auth_integration_test.dart"

# 一起啟動使用者故事 1 的所有模型:
Task T026: "建立 lib/features/auth/data/models/user_model.dart"
Task T027: "建立 lib/features/auth/data/models/session_model.dart"

# 一起啟動使用者故事 1 的領域實體:
Task T031: "建立 lib/features/auth/domain/entities/user.dart"
Task T032: "建立 lib/features/auth/domain/entities/session.dart"
Task T033: "建立 lib/features/auth/domain/entities/auth_state.dart"

# 一起啟動使用者故事 1 的用例:
Task T035: "建立 lib/features/auth/domain/usecases/sign_in_with_google.dart"
Task T036: "建立 lib/features/auth/domain/usecases/sign_in_with_facebook.dart"

# 一起啟動使用者故事 1 的 UI Widget:
Task T040: "建立 lib/features/auth/presentation/widgets/google_sign_in_button.dart"
Task T041: "建立 lib/features/auth/presentation/widgets/facebook_sign_in_button.dart"
```

---

## 實作策略

### MVP 優先（僅使用者故事 1）

1. 完成 Phase 1: 設定
2. 完成 Phase 2: 基礎建設（關鍵 - 阻塞所有故事）
3. 完成 Phase 3: 使用者故事 1
4. **停止並驗證**: 獨立測試使用者故事 1
5. 如果就緒，部署/展示

### 漸進式交付

1. 完成設定 + 基礎建設 → 基礎就緒
2. 加入使用者故事 1 → 獨立測試 → 部署/展示（MVP！）
3. 加入使用者故事 2 → 獨立測試 → 部署/展示
4. 加入使用者故事 3 → 獨立測試 → 部署/展示
5. 每個故事都增加價值而不會破壞先前的故事

### 平行團隊策略

多位開發者時：

1. 團隊一起完成設定 + 基礎建設
2. 基礎建設完成後：
   - 開發者 A: 使用者故事 1（社群帳號登入）
   - 開發者 B: 使用者故事 2（登出功能）
   - 開發者 C: 使用者故事 3（首頁佔位顯示）
3. 故事獨立完成並整合

---

## 任務統計總結

- **總任務數**: 78
- **使用者故事 1 任務數**: 31（包含 8 個測試任務）
- **使用者故事 2 任務數**: 9（包含 3 個測試任務）
- **使用者故事 3 任務數**: 6（包含 2 個測試任務）
- **設定任務數**: 10
- **基礎建設任務數**: 7
- **精緻化任務數**: 15（包含 3 個效能驗證測試任務）
- **可平行執行任務**: 約 43 個任務標記 [P]

---

## 格式驗證

✅ 所有任務遵循檢查清單格式：`- [ ] [TaskID] [P?] [Story?] 描述與檔案路徑`  
✅ 所有使用者故事任務包含 [US1]/[US2]/[US3] 標籤  
✅ 所有任務包含確切的檔案路徑  
✅ 測試任務在實作任務之前  
✅ 每個使用者故事都有獨立測試標準

---

## 建議的 MVP 範圍

**建議**: 實作到 Phase 3（使用者故事 1）作為 MVP

**理由**:
- 使用者故事 1 提供核心價值：社群帳號登入和自動登入
- 可獨立測試和驗證
- 為後續故事建立基礎
- 約 48 個任務（包含設定和基礎建設，不含 Phase 6 精緻化）
- 預估完成時間：2-3 週（單人）或 1-1.5 週（2-3 人平行）

**MVP 交付成果**:
- ✅ 使用者可以透過 Google 或 Facebook 登入
- ✅ 系統記住登入狀態
- ✅ 自動登入功能
- ✅ 完整的錯誤處理
- ✅ 全面的測試覆蓋

---

## 注意事項

- [P] 任務 = 不同檔案，無相依性
- [Story] 標籤將任務映射到特定使用者故事以便追蹤
- 每個使用者故事應該可獨立完成和測試
- 在實作前驗證測試失敗
- 每個任務或邏輯群組後提交
- 在任何檢查點停止以獨立驗證故事
- 避免：模糊任務、相同檔案衝突、破壞獨立性的跨故事相依性
