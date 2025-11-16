# 快速開始：登入記帳主頁功能

**功能**: 001-login-home  
**日期**: 2025-11-16  
**預估設定時間**: 30-45 分鐘

## 概述

本指南將協助您完成「登入記帳主頁功能」的開發環境設定，包括 Flutter SDK 安裝、Supabase 專案建立、OAuth 供應商設定、以及相依套件安裝。

---

## 前置需求

### 必要軟體

- **Flutter SDK**: 3.16.0 或更高版本
- **Dart SDK**: 3.2.0 或更高版本（隨 Flutter 安裝）
- **Git**: 用於版本控制

### 平台特定需求

#### iOS 開發
- **macOS**: 12.0 (Monterey) 或更高版本
- **Xcode**: 15.0 或更高版本
- **CocoaPods**: 1.11.0 或更高版本
- **Apple Developer 帳號**: 用於實機測試（免費帳號即可）

#### Android 開發
- **Android Studio**: 2023.1 或更高版本
- **JDK**: 11 或更高版本
- **Android SDK**: API Level 24 (Android 7.0) 或更高版本

### 第三方服務帳號

- **Supabase 帳號**: https://supabase.com（免費方案即可）
- **Google Cloud 帳號**: https://console.cloud.google.com（免費，需要信用卡）
- **Facebook 開發者帳號**: https://developers.facebook.com（免費）

---

## 步驟 1: 驗證 Flutter 環境

### 1.1 檢查 Flutter 版本

```bash
flutter --version
```

**預期輸出**:
```
Flutter 3.16.x • channel stable • ...
Dart 3.2.x • DevTools 2.x.x
```

如果版本過舊，執行更新：
```bash
flutter upgrade
```

### 1.2 執行環境檢查

```bash
flutter doctor
```

**確認以下項目都有 ✓**:
- Flutter SDK
- Xcode (macOS)
- Android toolchain
- Connected device（至少一個模擬器或實機）

如有 ✗ 或 ⚠️，請根據提示修復。

### 1.3 啟用必要平台（如需）

```bash
# 如果尚未啟用 iOS
flutter config --enable-ios

# 如果尚未啟用 Android
flutter config --enable-android
```

---

## 步驟 2: 複製專案並安裝相依套件

### 2.1 複製 Repository

```bash
git clone https://github.com/chunchun1213/money-manager.git
cd money-manager
```

### 2.2 切換到功能分支

```bash
git checkout 001-login-home
```

如果分支不存在，建立新分支：
```bash
git checkout -b 001-login-home
```

### 2.3 安裝相依套件

```bash
flutter pub get
```

**預期安裝的主要套件**:
- `flutter_riverpod` ^2.4.0
- `supabase_flutter` ^2.0.0
- `google_sign_in` ^6.1.0
- `flutter_facebook_auth` ^6.0.0
- `flutter_secure_storage` ^9.0.0
- `freezed` ^2.4.0
- `json_serializable` ^6.7.0

### 2.4 執行程式碼生成（如有使用 Freezed）

```bash
flutter pub run build_runner build --delete-conflicting-outputs
```

---

## 步驟 3: 設定 Supabase

### 3.1 建立 Supabase 專案

1. 前往 https://supabase.com 並登入
2. 點擊 **New Project**
3. 填寫專案資訊：
   - **Name**: `money-manager`
   - **Database Password**: 設定強密碼（記錄下來）
   - **Region**: 選擇最近的區域（如 `Northeast Asia (Tokyo)`）
   - **Pricing Plan**: Free
4. 點擊 **Create new project**（約需 2 分鐘）

### 3.2 取得 API 金鑰

1. 專案建立完成後，前往 **Settings** → **API**
2. 複製以下資訊：
   - **Project URL**: `https://xxxxx.supabase.co`
   - **Anon public key**: `eyJhbGci...`（長字串）

### 3.3 建立資料表

1. 前往 **SQL Editor**
2. 點擊 **New query**
3. 複製並執行以下 SQL：

```sql
-- 建立 users 表格
CREATE TABLE users (
  id UUID PRIMARY KEY REFERENCES auth.users(id) ON DELETE CASCADE,
  email TEXT NOT NULL UNIQUE,
  display_name TEXT,
  photo_url TEXT,
  auth_provider TEXT NOT NULL CHECK (auth_provider IN ('GOOGLE', 'FACEBOOK')),
  created_at TIMESTAMPTZ DEFAULT NOW() NOT NULL,
  updated_at TIMESTAMPTZ DEFAULT NOW() NOT NULL
);

-- 建立 updated_at 自動更新函式
CREATE OR REPLACE FUNCTION update_updated_at_column()
RETURNS TRIGGER AS $$
BEGIN
  NEW.updated_at = NOW();
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- 建立觸發器
CREATE TRIGGER update_users_updated_at
  BEFORE UPDATE ON users
  FOR EACH ROW
  EXECUTE FUNCTION update_updated_at_column();

-- 啟用 RLS
ALTER TABLE users ENABLE ROW LEVEL SECURITY;

-- 建立 RLS 政策
CREATE POLICY "Users can view own data"
  ON users FOR SELECT
  USING (auth.uid() = id);

CREATE POLICY "Users can update own data"
  ON users FOR UPDATE
  USING (auth.uid() = id);

CREATE POLICY "Users can insert own data"
  ON users FOR INSERT
  WITH CHECK (auth.uid() = id);

-- 建立 sessions 表格
CREATE TABLE sessions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  device_id TEXT NOT NULL,
  device_name TEXT,
  device_type TEXT NOT NULL CHECK (device_type IN ('ios', 'android')),
  access_token TEXT NOT NULL,
  refresh_token TEXT NOT NULL,
  expires_at TIMESTAMPTZ NOT NULL CHECK (expires_at > created_at),
  last_activity_at TIMESTAMPTZ DEFAULT NOW() NOT NULL,
  created_at TIMESTAMPTZ DEFAULT NOW() NOT NULL,
  UNIQUE(user_id, device_id)
);

-- 建立索引
CREATE INDEX idx_sessions_user_id ON sessions(user_id);
CREATE INDEX idx_sessions_device_id ON sessions(device_id);
CREATE INDEX idx_sessions_expires_at ON sessions(expires_at);

-- 啟用 RLS
ALTER TABLE sessions ENABLE ROW LEVEL SECURITY;

-- 建立 RLS 政策
CREATE POLICY "Users can view own sessions"
  ON sessions FOR SELECT
  USING (auth.uid() = user_id);

CREATE POLICY "Users can insert own sessions"
  ON sessions FOR INSERT
  WITH CHECK (auth.uid() = user_id);

CREATE POLICY "Users can update own sessions"
  ON sessions FOR UPDATE
  USING (auth.uid() = user_id);

CREATE POLICY "Users can delete own sessions"
  ON sessions FOR DELETE
  USING (auth.uid() = user_id);

-- 建立 user_preferences 表格
CREATE TABLE user_preferences (
  user_id UUID PRIMARY KEY REFERENCES users(id) ON DELETE CASCADE,
  last_auth_provider TEXT CHECK (last_auth_provider IN ('GOOGLE', 'FACEBOOK')),
  updated_at TIMESTAMPTZ DEFAULT NOW() NOT NULL
);

-- 啟用 RLS
ALTER TABLE user_preferences ENABLE ROW LEVEL SECURITY;

-- 建立 RLS 政策
CREATE POLICY "Users can view own preferences"
  ON user_preferences FOR SELECT
  USING (auth.uid() = user_id);

CREATE POLICY "Users can insert own preferences"
  ON user_preferences FOR INSERT
  WITH CHECK (auth.uid() = user_id);

CREATE POLICY "Users can update own preferences"
  ON user_preferences FOR UPDATE
  USING (auth.uid() = user_id);
```

4. 點擊 **Run** 執行

---

## 步驟 4: 設定 Google OAuth

### 4.1 建立 Google Cloud 專案

1. 前往 https://console.cloud.google.com
2. 點擊專案下拉選單 → **New Project**
3. 輸入專案名稱：`Money Manager`
4. 點擊 **Create**

### 4.2 啟用 Google+ API

1. 在左側選單選擇 **APIs & Services** → **Library**
2. 搜尋 `Google+ API`
3. 點擊 **Enable**

### 4.3 建立 OAuth 2.0 憑證

1. 前往 **APIs & Services** → **Credentials**
2. 點擊 **Create Credentials** → **OAuth client ID**
3. 如果首次建立，點擊 **Configure consent screen**：
   - User Type: **External**
   - App name: `Money Manager`
   - User support email: 您的 email
   - Developer contact: 您的 email
   - 點擊 **Save and Continue**（其他欄位可略過）

4. 返回 **Credentials** 並再次點擊 **Create Credentials** → **OAuth client ID**

5. **為 iOS 建立憑證**:
   - Application type: **iOS**
   - Name: `Money Manager iOS`
   - Bundle ID: `com.yourcompany.moneymanager`（記錄此值）
   - 點擊 **Create**
   - 複製 **Client ID**（稍後使用）

6. **為 Android 建立憑證**:
   - Application type: **Android**
   - Name: `Money Manager Android`
   - Package name: `com.yourcompany.money_manager`
   - SHA-1 憑證指紋: 執行以下指令取得
     ```bash
     # macOS/Linux
     keytool -list -v -keystore ~/.android/debug.keystore -alias androiddebugkey -storepass android -keypass android

     # Windows
     keytool -list -v -keystore "%USERPROFILE%\.android\debug.keystore" -alias androiddebugkey -storepass android -keypass android
     ```
   - 複製 `SHA1` 值並貼上
   - 點擊 **Create**

7. **為 Web 建立憑證** (Supabase 需要):
   - Application type: **Web application**
   - Name: `Money Manager Web`
   - Authorized redirect URIs: 新增以下 URL
     ```
     https://xxxxx.supabase.co/auth/v1/callback
     ```
     (將 `xxxxx` 替換為您的 Supabase Project ID)
   - 點擊 **Create**
   - 複製 **Client ID** 和 **Client Secret**

### 4.4 在 Supabase 中設定 Google Provider

1. 前往 Supabase Dashboard → **Authentication** → **Providers**
2. 找到 **Google** 並點擊 **Enable**
3. 填入：
   - **Client ID**: 貼上步驟 4.3.7 取得的 Web Client ID
   - **Client Secret**: 貼上步驟 4.3.7 取得的 Web Client Secret
4. 點擊 **Save**

---

## 步驟 5: 設定 Facebook OAuth

### 5.1 建立 Facebook App

1. 前往 https://developers.facebook.com
2. 點擊右上角 **My Apps** → **Create App**
3. 選擇 **Consumer** → **Next**
4. 填寫：
   - **App name**: `Money Manager`
   - **App contact email**: 您的 email
5. 點擊 **Create app**

### 5.2 設定 Facebook Login

1. 在 Dashboard 點擊 **Add Product**
2. 找到 **Facebook Login** 並點擊 **Set Up**
3. 選擇 **iOS** 並點擊 **Next**（iOS 設定）:
   - Bundle ID: 輸入與步驟 4.3.5 相同的 Bundle ID
   - 點擊 **Save** → **Next**（其他步驟可略過）

4. 返回並選擇 **Android**（Android 設定）:
   - Package Name: `com.yourcompany.money_manager`
   - Default Activity Class Name: `com.yourcompany.money_manager.MainActivity`
   - Key Hashes: 執行以下指令取得
     ```bash
     # macOS/Linux
     keytool -exportcert -alias androiddebugkey -keystore ~/.android/debug.keystore | openssl sha1 -binary | openssl base64

     # Windows
     keytool -exportcert -alias androiddebugkey -keystore "%USERPROFILE%\.android\debug.keystore" | openssl sha1 -binary | openssl base64
     ```
   - 輸入密碼: `android`
   - 貼上產生的 hash
   - 點擊 **Save**

### 5.3 設定有效的 OAuth 重新導向 URI

1. 前往 **Facebook Login** → **Settings**
2. 在 **Valid OAuth Redirect URIs** 新增：
   ```
   https://xxxxx.supabase.co/auth/v1/callback
   ```
3. 點擊 **Save Changes**

### 5.4 取得 App ID 和 App Secret

1. 前往 Dashboard → **Settings** → **Basic**
2. 複製：
   - **App ID**
   - **App Secret**（點擊 Show 顯示）

### 5.5 在 Supabase 中設定 Facebook Provider

1. 前往 Supabase Dashboard → **Authentication** → **Providers**
2. 找到 **Facebook** 並點擊 **Enable**
3. 填入：
   - **Facebook client ID**: 貼上 App ID
   - **Facebook secret**: 貼上 App Secret
4. 點擊 **Save**

---

## 步驟 6: 設定環境變數

### 6.1 建立 .env 檔案

在專案根目錄建立 `.env` 檔案：

```bash
cd /path/to/money-manager
touch .env
```

### 6.2 填入環境變數

編輯 `.env` 並填入以下內容：

```env
# Supabase
SUPABASE_URL=https://xxxxx.supabase.co
SUPABASE_ANON_KEY=eyJhbGci...

# Google OAuth (iOS)
GOOGLE_IOS_CLIENT_ID=xxxxx.apps.googleusercontent.com

# Facebook
FACEBOOK_APP_ID=1234567890
FACEBOOK_CLIENT_TOKEN=xxxxxxxxxxxxxxxxxxxxx
```

### 6.3 將 .env 加入 .gitignore

確認 `.gitignore` 包含：

```gitignore
.env
*.env
```

---

## 步驟 7: 設定 iOS 平台

### 7.1 編輯 Info.plist

開啟 `ios/Runner/Info.plist` 並新增：

```xml
<!-- Google Sign-In -->
<key>GIDClientID</key>
<string>YOUR_GOOGLE_IOS_CLIENT_ID</string>

<key>CFBundleURLTypes</key>
<array>
  <dict>
    <key>CFBundleURLSchemes</key>
    <array>
      <string>com.googleusercontent.apps.YOUR_REVERSED_CLIENT_ID</string>
    </array>
  </dict>
  <!-- Facebook Login -->
  <dict>
    <key>CFBundleURLSchemes</key>
    <array>
      <string>fbYOUR_FACEBOOK_APP_ID</string>
    </array>
  </dict>
  <!-- Supabase Deep Link -->
  <dict>
    <key>CFBundleURLSchemes</key>
    <array>
      <string>io.supabase.moneymanager</string>
    </array>
  </dict>
</array>

<!-- Facebook Login -->
<key>FacebookAppID</key>
<string>YOUR_FACEBOOK_APP_ID</string>
<key>FacebookDisplayName</key>
<string>Money Manager</string>

<!-- Supabase Universal Links -->
<key>com.apple.developer.associated-domains</key>
<array>
  <string>applinks:xxxxx.supabase.co</string>
</array>
```

**替換變數**:
- `YOUR_GOOGLE_IOS_CLIENT_ID`: 步驟 4.3.5 的 iOS Client ID
- `YOUR_REVERSED_CLIENT_ID`: 將 Client ID 反轉（如 `com.googleusercontent.apps.123-abc` → `123-abc.apps.googleusercontent.com`）
- `YOUR_FACEBOOK_APP_ID`: 步驟 5.4 的 App ID
- `xxxxx`: Supabase Project ID

### 7.2 安裝 CocoaPods 相依套件

```bash
cd ios
pod install
cd ..
```

---

## 步驟 8: 設定 Android 平台

### 8.1 編輯 AndroidManifest.xml

開啟 `android/app/src/main/AndroidManifest.xml` 並在 `<application>` 標籤內新增：

```xml
<!-- Facebook Login -->
<meta-data
    android:name="com.facebook.sdk.ApplicationId"
    android:value="@string/facebook_app_id"/>

<meta-data
    android:name="com.facebook.sdk.ClientToken"
    android:value="@string/facebook_client_token"/>

<activity
    android:name="com.facebook.FacebookActivity"
    android:configChanges="keyboard|keyboardHidden|screenLayout|screenSize|orientation"
    android:label="@string/app_name" />

<activity
    android:name="com.facebook.CustomTabActivity"
    android:exported="true">
    <intent-filter>
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />
        <data android:scheme="@string/fb_login_protocol_scheme" />
    </intent-filter>
</activity>

<!-- Supabase Deep Link -->
<intent-filter>
    <action android:name="android.intent.action.VIEW" />
    <category android:name="android.intent.category.DEFAULT" />
    <category android:name="android.intent.category.BROWSABLE" />
    <data
        android:scheme="io.supabase.moneymanager"
        android:host="login-callback" />
</intent-filter>
```

### 8.2 建立 strings.xml

建立 `android/app/src/main/res/values/strings.xml`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <string name="app_name">Money Manager</string>
    <string name="facebook_app_id">YOUR_FACEBOOK_APP_ID</string>
    <string name="facebook_client_token">YOUR_FACEBOOK_CLIENT_TOKEN</string>
    <string name="fb_login_protocol_scheme">fbYOUR_FACEBOOK_APP_ID</string>
</resources>
```

**替換變數**:
- `YOUR_FACEBOOK_APP_ID`: 步驟 5.4 的 App ID
- `YOUR_FACEBOOK_CLIENT_TOKEN`: 步驟 5.4 的 Client Token（在 Settings → Advanced → Client Token）

---

## 步驟 9: 初始化 Supabase

### 9.1 編輯 main.dart

開啟 `lib/main.dart` 並在 `main()` 函式中初始化：

```dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:supabase_flutter/supabase_flutter.dart';
import 'package:flutter_dotenv/flutter_dotenv.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();

  // 載入環境變數
  await dotenv.load(fileName: ".env");

  // 初始化 Supabase
  await Supabase.initialize(
    url: dotenv.env['SUPABASE_URL']!,
    anonKey: dotenv.env['SUPABASE_ANON_KEY']!,
  );

  runApp(
    const ProviderScope(
      child: MyApp(),
    ),
  );
}
```

---

## 步驟 10: 執行測試

### 10.1 執行單元測試

```bash
flutter test
```

### 10.2 執行應用程式

```bash
# iOS 模擬器
flutter run -d ios

# Android 模擬器
flutter run -d android

# 實機（先連接裝置）
flutter run
```

### 10.3 測試登入流程

1. 開啟 APP，應該會看到登入頁面
2. 點擊 **Google 登入**
3. 完成 Google OAuth 授權
4. 應該會導向首頁並顯示「施工中」訊息
5. 點擊 **登出** 按鈕
6. 應該會返回登入頁面

---

## 常見問題排除

### Q1: Google 登入失敗 - "Sign in canceled"

**原因**: iOS Client ID 未正確設定在 Info.plist

**解決方法**:
1. 檢查 Info.plist 的 `GIDClientID` 是否正確
2. 確認 `CFBundleURLSchemes` 包含反轉的 Client ID

### Q2: Facebook 登入失敗 - "App not set up"

**原因**: Facebook App 尚未發布或設定不完整

**解決方法**:
1. 前往 Facebook App Dashboard
2. 在 Settings → Basic 檢查 App Status 是否為 Live
3. 如為開發模式，確認測試帳號已加入 Roles → Test Users

### Q3: Supabase 連線錯誤 - "Invalid API key"

**原因**: SUPABASE_ANON_KEY 錯誤或過期

**解決方法**:
1. 前往 Supabase Dashboard → Settings → API
2. 重新複製 `anon public` key
3. 更新 `.env` 檔案
4. 重新執行 `flutter run`

### Q4: Android Build 失敗 - "Duplicate class"

**原因**: 相依套件版本衝突

**解決方法**:
```bash
cd android
./gradlew clean
cd ..
flutter clean
flutter pub get
flutter run
```

### Q5: iOS Build 失敗 - "CocoaPods could not find compatible versions"

**原因**: Pod 相依性衝突

**解決方法**:
```bash
cd ios
pod repo update
pod install --repo-update
cd ..
flutter run
```

---

## 下一步

環境設定完成後，您可以：

1. **閱讀 [plan.md](./plan.md)**: 了解完整的技術架構和實作計畫
2. **閱讀 [data-model.md](./data-model.md)**: 了解資料模型設計
3. **閱讀 [contracts/](./contracts/)**: 了解 API 合約規範
4. **開始實作**: 根據 `tasks.md`（待建立）的任務清單開始開發
5. **執行測試**: 確保所有功能符合規格要求

---

## 參考資源

- [Flutter 官方文件](https://flutter.dev/docs)
- [Supabase Flutter 套件](https://pub.dev/packages/supabase_flutter)
- [Google Sign-In for Flutter](https://pub.dev/packages/google_sign_in)
- [Flutter Facebook Auth](https://pub.dev/packages/flutter_facebook_auth)
- [Riverpod 文件](https://riverpod.dev)

---

**文件版本**: 1.0.0  
**最後更新**: 2025-11-16  
**維護者**: Money Manager 開發團隊
