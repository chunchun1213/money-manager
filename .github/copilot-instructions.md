# money-manager Development Guidelines

Auto-generated from all feature plans. Last updated: 2025-11-16

## Active Technologies

- **Flutter** 3.16+ (Mobile App Framework)
- **Dart** 3.2+ (Programming Language)
- **Riverpod** 2.4+ (State Management)
- **Supabase** 2.0+ (Backend Service - Auth, Database)
- **Google Sign-In** 6.1+ (Google OAuth)
- **Flutter Facebook Auth** 6.0+ (Facebook OAuth)
- **Flutter Secure Storage** 9.0+ (Local Encrypted Storage)
- **Freezed** 2.4+ (Code Generation for Models)

## Project Structure

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
│       │   ├── models/                # 資料模型（User, Session, AuthProvider）
│       │   ├── repositories/          # Repository 實作
│       │   └── datasources/           # 資料源（Supabase, 本地儲存）
│       ├── domain/
│       │   ├── entities/              # 業務實體
│       │   ├── repositories/          # Repository 介面
│       │   └── usecases/              # 使用案例（登入、登出、檢查狀態）
│       └── presentation/
│           ├── providers/             # Riverpod Providers
│           ├── pages/                 # 頁面（登入頁、首頁）
│           └── widgets/               # 元件（登入按鈕、施工中元件）
├── shared/
│   └── widgets/                       # 共用元件
└── firebase_options.dart              # Firebase 設定（如需使用）

test/
├── unit/                              # 單元測試
├── integration/                       # 整合測試
└── widget/                            # Widget 測試

design-assets/                         # Figma 設計資源
```

## Commands

### 測試指令

```bash
# 執行所有測試
flutter test

# 執行單元測試
flutter test test/unit/

# 執行整合測試
flutter test test/integration/

# 執行 E2E 測試
flutter test integration_test/

# 測試覆蓋率報告
flutter test --coverage
```

### 開發指令

```bash
# 執行程式碼生成（Freezed, JSON Serializable）
flutter pub run build_runner build --delete-conflicting-outputs

# 執行 APP (iOS)
flutter run -d ios

# 執行 APP (Android)
flutter run -d android

# 清理建構產物
flutter clean
```

### 程式碼品質

```bash
# 執行 Linter
flutter analyze

# 格式化程式碼
dart format lib/ test/

# 檢查相依套件更新
flutter pub outdated
```

## Code Style

### Dart 編碼規範

- 遵循 [Effective Dart](https://dart.dev/guides/language/effective-dart) 指南
- 使用 `flutter_lints` 套件進行靜態分析
- 所有公開 API 必須有 dartdoc 註解
- 優先使用 `const` 建構子減少重建
- 使用 `Freezed` 建立不可變資料類別

### 命名慣例

- **檔案**: `snake_case.dart`（如 `auth_repository.dart`）
- **類別**: `PascalCase`（如 `AuthRepository`）
- **變數/函式**: `camelCase`（如 `signInWithGoogle`）
- **常數**: `lowerCamelCase`（如 `primaryColor`）
- **私有成員**: `_camelCase`（如 `_handleError`）

### 架構原則

- **Feature-First**: 依功能組織程式碼，而非技術層級
- **Clean Architecture**: 分離 Data, Domain, Presentation 層
- **SOLID 原則**: 單一職責、開放封閉、依賴反轉
- **DRY**: 避免重複程式碼，提取共用邏輯

### 設計規範

- **色彩**: 使用 `design_tokens.dart` 定義的色彩常數
- **間距**: 使用 8px 基礎網格系統（spacing-xs/sm/md/lg/xl）
- **文字**: 遵循 Material 3 Typography 規範
- **無障礙**: 所有 Widget 提供 Semantics 標籤

## Recent Changes

- 001-login-home: 新增登入記帳主頁功能（Flutter + Supabase + OAuth）

<!-- MANUAL ADDITIONS START -->
<!-- MANUAL ADDITIONS END -->
