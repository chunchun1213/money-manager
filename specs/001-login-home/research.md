# 技術研究：登入記帳主頁功能

**日期**: 2025-11-16  
**專案**: money-manager  
**功能**: 001-login-home

## 研究目標

解決實作計畫中標記為 NEEDS CLARIFICATION 的技術議題，為實作提供明確的技術決策和最佳實踐指引。

---

## 研究 1: Supabase Auth 整合最佳實踐

### 決策

使用 **Supabase Auth** 搭配 OAuth 2.0 進行社群登入，並啟用 Row Level Security (RLS) 保護使用者資料。

### 理由

1. **完整整合**: Supabase 提供 Auth、Database、Storage 一站式解決方案
2. **安全性**: 內建 JWT token 管理、RLS 政策、自動 token 更新
3. **多供應商支援**: 原生支援 Google、Facebook 等主流 OAuth 供應商
4. **Flutter 套件成熟**: `supabase_flutter` 套件維護良好，社群活躍

### 整合步驟

#### 1. Supabase 專案設定

```yaml
# pubspec.yaml
dependencies:
  supabase_flutter: ^2.0.0
```

#### 2. 初始化

```dart
// main.dart
await Supabase.initialize(
  url: 'YOUR_SUPABASE_URL',
  anonKey: 'YOUR_SUPABASE_ANON_KEY',
);
```

#### 3. OAuth 供應商配置

在 Supabase Dashboard:
- **Authentication > Providers > Google**
  - 啟用 Google provider
  - 輸入 Google Cloud Console 的 Client ID 和 Client Secret
  - 設定 Redirect URL: `io.supabase.{project-ref}://login-callback`

- **Authentication > Providers > Facebook**
  - 啟用 Facebook provider
  - 輸入 Facebook App 的 App ID 和 App Secret
  - 設定 Redirect URL: 同上

#### 4. RLS 政策範例

```sql
-- users 表格 RLS 政策
CREATE POLICY "Users can view own data"
  ON users FOR SELECT
  USING (auth.uid() = id);

CREATE POLICY "Users can update own data"
  ON users FOR UPDATE
  USING (auth.uid() = id);

-- sessions 表格 RLS 政策
CREATE POLICY "Users can view own sessions"
  ON sessions FOR SELECT
  USING (auth.uid() = user_id);

CREATE POLICY "Users can delete own sessions"
  ON sessions FOR DELETE
  USING (auth.uid() = user_id);
```

### 考慮的替代方案

- **Firebase Auth**: 功能強大但需額外設定 Google Cloud，學習曲線較陡
- **自建 Auth 系統**: 開發成本高，安全風險大，維護負擔重
- **Auth0**: 第三方服務，免費方案有用戶數限制，成本較高

**結論**: Supabase Auth 提供最佳的成本效益比和開發體驗。

---

## 研究 2: Riverpod 狀態管理模式

### 決策

使用 **Riverpod 2.x** 搭配 `AsyncNotifierProvider` 管理認證狀態，使用 `FutureProvider` 處理一次性非同步操作。

### 理由

1. **類型安全**: 編譯時期檢查，減少執行時期錯誤
2. **測試友善**: Provider 可輕鬆 Mock，單元測試簡單
3. **效能**: 只重建受影響的 Widget，避免不必要的渲染
4. **錯誤處理**: 內建 AsyncValue 處理 loading/data/error 狀態

### 架構設計

#### 1. 認證狀態 Provider

```dart
// auth_state_provider.dart
@riverpod
class AuthState extends _$AuthState {
  @override
  Future<User?> build() async {
    // 檢查現有會話
    return await ref.read(authRepositoryProvider).getCurrentUser();
  }

  Future<void> signInWithGoogle() async {
    state = const AsyncValue.loading();
    state = await AsyncValue.guard(() async {
      return await ref.read(authRepositoryProvider).signInWithGoogle();
    });
  }

  Future<void> signInWithFacebook() async {
    state = const AsyncValue.loading();
    state = await AsyncValue.guard(() async {
      return await ref.read(authRepositoryProvider).signInWithFacebook();
    });
  }

  Future<void> signOut() async {
    state = const AsyncValue.loading();
    state = await AsyncValue.guard(() async {
      await ref.read(authRepositoryProvider).signOut();
      return null;
    });
  }
}
```

#### 2. Repository Provider

```dart
// auth_repository_provider.dart
@riverpod
AuthRepository authRepository(AuthRepositoryRef ref) {
  return AuthRepositoryImpl(
    supabaseClient: Supabase.instance.client,
    localStorage: ref.read(localStorageProvider),
  );
}
```

#### 3. Widget 使用範例

```dart
// login_page.dart
class LoginPage extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final authState = ref.watch(authStateProvider);

    return authState.when(
      data: (user) {
        if (user != null) {
          // 已登入，導向首頁
          return HomePage();
        }
        return LoginForm();
      },
      loading: () => LoadingIndicator(),
      error: (err, stack) => ErrorWidget(error: err),
    );
  }
}
```

### 錯誤處理策略

```dart
// 在 Provider 中處理錯誤
state = await AsyncValue.guard(() async {
  try {
    return await repository.signIn();
  } on AuthException catch (e) {
    // 轉換為使用者友善的錯誤訊息
    throw _mapAuthExceptionToUserMessage(e);
  } on NetworkException {
    throw '網路連線失敗，請檢查網路設定';
  }
});

// 在 UI 中顯示錯誤
authState.when(
  error: (err, stack) {
    ScaffoldMessenger.of(context).showSnackBar(
      SnackBar(content: Text(err.toString())),
    );
  },
  // ...
);
```

### 考慮的替代方案

- **Provider (flutter_provider)**: 較舊的方案，缺乏編譯時期檢查
- **BLoC**: 較重量級，對於簡單狀態管理過於複雜
- **GetX**: 全域狀態，測試困難，不符合最佳實踐

**結論**: Riverpod 提供最佳的類型安全和測試體驗。

---

## 研究 3: 多裝置會話管理策略

### 決策

使用 **裝置識別碼 + Supabase Sessions 表格** 實作多裝置獨立會話，每個裝置維護獨立的 session token。

### 理由

1. **安全性**: 每個裝置有獨立的 token，單一裝置登出不影響其他裝置
2. **可追蹤**: 可記錄每個裝置的登入時間、最後活動時間
3. **靈活性**: 可實作單點登出（所有裝置）或單裝置登出

### 資料庫 Schema

```sql
-- sessions 表格
CREATE TABLE sessions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,
  device_id TEXT NOT NULL,
  device_name TEXT,
  device_type TEXT, -- 'ios', 'android'
  access_token TEXT NOT NULL,
  refresh_token TEXT NOT NULL,
  expires_at TIMESTAMPTZ NOT NULL,
  last_activity_at TIMESTAMPTZ DEFAULT NOW(),
  created_at TIMESTAMPTZ DEFAULT NOW(),
  UNIQUE(user_id, device_id)
);

-- 自動清理過期會話的觸發器
CREATE OR REPLACE FUNCTION delete_expired_sessions()
RETURNS TRIGGER AS $$
BEGIN
  DELETE FROM sessions WHERE expires_at < NOW();
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER cleanup_expired_sessions
  AFTER INSERT OR UPDATE ON sessions
  EXECUTE FUNCTION delete_expired_sessions();
```

### 裝置識別碼生成

```dart
// device_info_service.dart
class DeviceInfoService {
  Future<String> getDeviceId() async {
    final localStorage = await SharedPreferences.getInstance();
    
    // 嘗試從本地儲存取得
    String? deviceId = localStorage.getString('device_id');
    
    if (deviceId == null) {
      // 生成新的裝置 ID
      deviceId = const Uuid().v4();
      await localStorage.setString('device_id', deviceId);
    }
    
    return deviceId;
  }

  Future<Map<String, String>> getDeviceInfo() async {
    final deviceInfo = DeviceInfoPlugin();
    String deviceName = '';
    String deviceType = '';

    if (Platform.isIOS) {
      final iosInfo = await deviceInfo.iosInfo;
      deviceName = iosInfo.name ?? 'iOS Device';
      deviceType = 'ios';
    } else if (Platform.isAndroid) {
      final androidInfo = await deviceInfo.androidInfo;
      deviceName = androidInfo.model ?? 'Android Device';
      deviceType = 'android';
    }

    return {
      'deviceId': await getDeviceId(),
      'deviceName': deviceName,
      'deviceType': deviceType,
    };
  }
}
```

### 會話記錄流程

```dart
// session_repository.dart
class SessionRepository {
  Future<void> recordSession(User user, Session session) async {
    final deviceInfo = await _deviceInfoService.getDeviceInfo();
    
    await supabase.from('sessions').upsert({
      'user_id': user.id,
      'device_id': deviceInfo['deviceId'],
      'device_name': deviceInfo['deviceName'],
      'device_type': deviceInfo['deviceType'],
      'access_token': session.accessToken,
      'refresh_token': session.refreshToken,
      'expires_at': session.expiresAt.toIso8601String(),
      'last_activity_at': DateTime.now().toIso8601String(),
    });
  }

  Future<void> deleteCurrentSession() async {
    final deviceId = await _deviceInfoService.getDeviceId();
    
    await supabase
      .from('sessions')
      .delete()
      .eq('device_id', deviceId);
  }

  Future<void> deleteAllSessions(String userId) async {
    await supabase
      .from('sessions')
      .delete()
      .eq('user_id', userId);
  }
}
```

### 考慮的替代方案

- **單一會話模式**: 簡單但使用者體驗差，多裝置互踢
- **全域登出**: 安全但不靈活，使用者無法獨立管理裝置
- **無會話記錄**: 無法追蹤使用者活動，安全性問題

**結論**: 多裝置獨立會話提供最佳的安全性和使用者體驗平衡。

---

## 研究 4: Flutter 社群登入套件比較

### 決策

- **Google 登入**: 使用 `google_sign_in` ^6.1.0 搭配 Supabase Auth
- **Facebook 登入**: 使用 `flutter_facebook_auth` ^6.0.0 搭配 Supabase Auth

### 理由

| 評估項目 | google_sign_in | flutter_facebook_auth | Firebase Auth |
|---------|---------------|---------------------|---------------|
| 維護狀態 | ✅ Google 官方 | ✅ 社群活躍 | ✅ Google 官方 |
| 文件品質 | ✅ 完整 | ✅ 良好 | ✅ 完整 |
| 平台支援 | iOS, Android, Web | iOS, Android, Web | iOS, Android, Web |
| 相依性 | 輕量 | 輕量 | 重量（需整套 Firebase） |
| Supabase 整合 | ✅ 簡單 | ✅ 簡單 | ⚠️ 需額外設定 |
| 授權費用 | 免費 | 免費 | 免費（有用量限制） |

### 整合方式

#### Google 登入整合

```dart
// google_sign_in_service.dart
class GoogleSignInService {
  final GoogleSignIn _googleSignIn = GoogleSignIn(
    scopes: ['email', 'profile'],
  );

  Future<AuthResponse> signIn() async {
    final googleUser = await _googleSignIn.signIn();
    if (googleUser == null) {
      throw AuthException('使用者取消登入');
    }

    final googleAuth = await googleUser.authentication;
    final accessToken = googleAuth.accessToken;
    final idToken = googleAuth.idToken;

    if (accessToken == null || idToken == null) {
      throw AuthException('無法取得 Google Token');
    }

    // 使用 Supabase Auth 驗證 Google Token
    return await supabase.auth.signInWithIdToken(
      provider: OAuthProvider.google,
      idToken: idToken,
      accessToken: accessToken,
    );
  }

  Future<void> signOut() async {
    await _googleSignIn.signOut();
  }
}
```

#### Facebook 登入整合

```dart
// facebook_sign_in_service.dart
class FacebookSignInService {
  Future<AuthResponse> signIn() async {
    final result = await FacebookAuth.instance.login();

    if (result.status == LoginStatus.success) {
      final accessToken = result.accessToken!.tokenString;

      // 使用 Supabase Auth 驗證 Facebook Token
      return await supabase.auth.signInWithIdToken(
        provider: OAuthProvider.facebook,
        idToken: accessToken,
      );
    } else if (result.status == LoginStatus.cancelled) {
      throw AuthException('使用者取消登入');
    } else {
      throw AuthException(result.message ?? '登入失敗');
    }
  }

  Future<void> signOut() async {
    await FacebookAuth.instance.logOut();
  }
}
```

### 平台設定

#### iOS (info.plist)

```xml
<!-- Google Sign-In -->
<key>GIDClientID</key>
<string>YOUR_GOOGLE_CLIENT_ID</string>

<key>CFBundleURLTypes</key>
<array>
  <dict>
    <key>CFBundleURLSchemes</key>
    <array>
      <string>com.googleusercontent.apps.YOUR_REVERSED_CLIENT_ID</string>
    </array>
  </dict>
</array>

<!-- Facebook Login -->
<key>CFBundleURLTypes</key>
<array>
  <dict>
    <key>CFBundleURLSchemes</key>
    <array>
      <string>fbYOUR_FACEBOOK_APP_ID</string>
    </array>
  </dict>
</array>

<key>FacebookAppID</key>
<string>YOUR_FACEBOOK_APP_ID</string>
<key>FacebookDisplayName</key>
<string>Money Manager</string>
```

#### Android (AndroidManifest.xml)

```xml
<!-- Google Sign-In 不需額外設定 -->

<!-- Facebook Login -->
<meta-data
    android:name="com.facebook.sdk.ApplicationId"
    android:value="@string/facebook_app_id"/>

<activity
    android:name="com.facebook.FacebookActivity"
    android:configChanges="keyboard|keyboardHidden|screenLayout|screenSize|orientation"
    android:label="@string/app_name" />
```

### 考慮的替代方案

- **Firebase Auth**: 功能完整但過於重量，需要完整 Firebase SDK
- **flutter_login**: 自建 UI 但需自行處理 OAuth 流程
- **Supabase Auth UI**: 網頁版 UI，不適合原生 APP

**結論**: `google_sign_in` 和 `flutter_facebook_auth` 提供最佳的輕量整合方案。

---

## 研究 5: 本地儲存安全方案

### 決策

使用 **flutter_secure_storage** 儲存敏感資料（token、使用者 ID），使用 **shared_preferences** 儲存非敏感偏好設定（上次登入提供商）。

### 理由

| 評估項目 | flutter_secure_storage | shared_preferences | Hive |
|---------|----------------------|-------------------|------|
| 安全性 | ✅ 加密儲存 | ❌ 明文儲存 | ⚠️ 可加密但需設定 |
| 平台整合 | iOS Keychain, Android Keystore | UserDefaults, SharedPreferences | 自定義檔案 |
| 效能 | 中等 | 快 | 快 |
| 使用複雜度 | 簡單 | 簡單 | 中等 |
| 資料大小限制 | 小量資料 | 小量資料 | 大量資料 |

### 平台安全性分析

#### iOS (Keychain)

- **安全等級**: 高
- **加密**: 硬體加密（Secure Enclave）
- **備份**: 可設定是否備份至 iCloud Keychain
- **限制**: 單一 App 存取，需授權才能跨 App 共享

#### Android (Keystore)

- **安全等級**: 高（Android 6.0+）
- **加密**: 硬體支援（Android 9.0+ 必須）
- **備份**: 預設不備份
- **限制**: API 23+ 才有完整支援

### 實作範例

```dart
// local_storage_service.dart
class LocalStorageService {
  final FlutterSecureStorage _secureStorage = FlutterSecureStorage(
    aOptions: AndroidOptions(
      encryptedSharedPreferences: true,
    ),
    iOptions: IOSOptions(
      accessibility: KeychainAccessibility.first_unlock,
    ),
  );
  
  final SharedPreferences _prefs;

  LocalStorageService(this._prefs);

  // === 敏感資料（使用 secure storage） ===
  
  Future<void> saveAccessToken(String token) async {
    await _secureStorage.write(key: 'access_token', value: token);
  }

  Future<String?> getAccessToken() async {
    return await _secureStorage.read(key: 'access_token');
  }

  Future<void> saveRefreshToken(String token) async {
    await _secureStorage.write(key: 'refresh_token', value: token);
  }

  Future<String?> getRefreshToken() async {
    return await _secureStorage.read(key: 'refresh_token');
  }

  Future<void> saveUserId(String userId) async {
    await _secureStorage.write(key: 'user_id', value: userId);
  }

  Future<String?> getUserId() async {
    return await _secureStorage.read(key: 'user_id');
  }

  // === 非敏感偏好（使用 shared preferences） ===

  Future<void> saveLastAuthProvider(String provider) async {
    await _prefs.setString('last_auth_provider', provider);
  }

  String? getLastAuthProvider() {
    return _prefs.getString('last_auth_provider');
  }

  Future<void> saveDeviceId(String deviceId) async {
    await _prefs.setString('device_id', deviceId);
  }

  String? getDeviceId() {
    return _prefs.getString('device_id');
  }

  // === 清除資料 ===

  Future<void> clearAll() async {
    await _secureStorage.deleteAll();
    await _prefs.clear();
  }
}
```

### 降級策略

如果 secure storage 在舊裝置上不可用（Android < 6.0）：

```dart
Future<void> _saveTokenSafely(String key, String value) async {
  try {
    await _secureStorage.write(key: key, value: value);
  } on PlatformException catch (e) {
    if (e.code == 'Not available') {
      // 降級到 shared preferences + 簡單加密
      final encrypted = await _simpleEncrypt(value);
      await _prefs.setString(key, encrypted);
    } else {
      rethrow;
    }
  }
}
```

### 考慮的替代方案

- **Hive + 加密**: 功能強大但設定複雜，對小量資料過度設計
- **SQLite + SQLCipher**: 重量級，適合大量資料而非簡單 key-value
- **僅使用 shared_preferences**: 不安全，token 會明文儲存

**結論**: `flutter_secure_storage` + `shared_preferences` 提供最佳的安全性和易用性平衡。

---

## 總結

### 關鍵決策總覽

| 技術議題 | 決策 | 主要理由 |
|---------|-----|---------|
| 後端認證 | Supabase Auth | 完整整合、安全性高、開發效率 |
| 狀態管理 | Riverpod 2.x | 類型安全、測試友善、效能 |
| 會話策略 | 多裝置獨立會話 | 安全性、使用者體驗、可追蹤 |
| Google 登入 | google_sign_in | 官方套件、穩定、輕量 |
| Facebook 登入 | flutter_facebook_auth | 社群活躍、文件完整、輕量 |
| 敏感資料儲存 | flutter_secure_storage | 平台原生加密、簡單 |
| 非敏感資料儲存 | shared_preferences | 快速、簡單、適合小量資料 |

### 風險與緩解

| 風險 | 機率 | 影響 | 緩解策略 |
|-----|-----|-----|---------|
| OAuth 供應商 API 變更 | 低 | 高 | 使用官方套件，定期更新相依套件 |
| Supabase 服務中斷 | 低 | 高 | 實作離線優雅降級，顯示友善錯誤訊息 |
| 舊裝置安全儲存不可用 | 中 | 中 | 實作降級策略，使用簡單加密 |
| 社群登入授權失敗 | 中 | 中 | 完整錯誤處理，提供重試機制 |

### 開發檢查清單

- [x] Supabase 專案建立與 OAuth 設定
- [x] Flutter 專案相依套件安裝
- [x] iOS/Android 平台設定（info.plist, AndroidManifest.xml）
- [ ] 實作 Repository 層（AuthRepository, SessionRepository）
- [ ] 實作 Riverpod Providers（AuthStateProvider）
- [ ] 實作 UI 層（LoginPage, HomePage, Widgets）
- [ ] 撰寫單元測試（Repository, UseCase, Provider）
- [ ] 撰寫整合測試（Supabase 整合、OAuth 流程）
- [ ] 撰寫 E2E 測試（完整登入流程）

---

**研究完成日期**: 2025-11-16  
**下一步**: 進入 Phase 1 - 根據研究結果實作資料模型和 API 合約
