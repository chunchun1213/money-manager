# 認證 API 合約

**版本**: 1.0.0  
**日期**: 2025-11-16  
**平台**: Supabase Auth

## 概述

本文件定義登入記帳主頁功能的認證 API 合約，基於 Supabase Auth 服務。所有端點使用 OAuth 2.0 標準進行認證。

---

## 1. Google 登入

### 端點資訊

- **服務**: Supabase Auth
- **方法**: `signInWithOAuth`
- **OAuth 供應商**: Google
- **認證流程**: Authorization Code Flow

### 請求

```dart
final response = await supabase.auth.signInWithOAuth(
  OAuthProvider.google,
  redirectTo: 'io.supabase.moneymanager://login-callback',
);
```

**參數**:
- `provider`: `OAuthProvider.google`
- `redirectTo`: (可選) 自訂 Redirect URI
- `scopes`: (可選) 預設包含 `email`, `profile`

### 回應

#### 成功回應

```dart
AuthResponse {
  user: User {
    id: "550e8400-e29b-41d4-a716-446655440000",
    email: "user@gmail.com",
    userMetadata: {
      "name": "User Name",
      "avatar_url": "https://lh3.googleusercontent.com/...",
      "provider": "google"
    },
    appMetadata: {
      "provider": "google",
      "providers": ["google"]
    }
  },
  session: Session {
    accessToken: "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    refreshToken: "refresh_token_...",
    expiresIn: 3600,
    expiresAt: 1700000000
  }
}
```

#### 錯誤回應

| 錯誤碼 | HTTP 狀態 | 描述 | 使用者訊息 |
|-------|----------|------|-----------|
| `user_cancelled` | - | 使用者在 OAuth 頁面取消授權 | "登入已取消，請重試" |
| `invalid_grant` | 400 | 無效的授權碼或 token | "登入失敗，請重試" |
| `network_error` | - | 網路連線問題 | "網路連線失敗，請檢查網路設定" |
| `provider_error` | 500 | Google OAuth 服務錯誤 | "Google 登入暫時無法使用，請稍後再試" |

### 錯誤處理範例

```dart
try {
  final response = await supabase.auth.signInWithOAuth(
    OAuthProvider.google,
  );
  
  if (response.session != null) {
    // 登入成功
    return response;
  } else {
    throw AuthException('無法建立會話');
  }
} on AuthException catch (e) {
  if (e.message.contains('cancelled')) {
    throw UserCancelledException();
  } else if (e.message.contains('network')) {
    throw NetworkException();
  } else {
    throw GenericAuthException(e.message);
  }
} catch (e) {
  throw UnexpectedException(e.toString());
}
```

---

## 2. Facebook 登入

### 端點資訊

- **服務**: Supabase Auth
- **方法**: `signInWithOAuth`
- **OAuth 供應商**: Facebook
- **認證流程**: Authorization Code Flow

### 請求

```dart
final response = await supabase.auth.signInWithOAuth(
  OAuthProvider.facebook,
  redirectTo: 'io.supabase.moneymanager://login-callback',
);
```

**參數**:
- `provider`: `OAuthProvider.facebook`
- `redirectTo`: (可選) 自訂 Redirect URI
- `scopes`: (可選) 預設包含 `email`, `public_profile`

### 回應

#### 成功回應

```dart
AuthResponse {
  user: User {
    id: "660e8400-e29b-41d4-a716-446655440001",
    email: "user@facebook.com",
    userMetadata: {
      "name": "User Name",
      "avatar_url": "https://graph.facebook.com/.../picture",
      "provider": "facebook"
    },
    appMetadata: {
      "provider": "facebook",
      "providers": ["facebook"]
    }
  },
  session: Session {
    accessToken: "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    refreshToken: "refresh_token_...",
    expiresIn: 3600,
    expiresAt: 1700000000
  }
}
```

#### 錯誤回應

錯誤碼與 Google 登入相同，僅供應商名稱不同。

---

## 3. 登出

### 端點資訊

- **服務**: Supabase Auth
- **方法**: `signOut`
- **作用範圍**: 全域（清除伺服器端會話）

### 請求

```dart
await supabase.auth.signOut();
```

**參數**: 無

### 回應

#### 成功回應

```dart
void // 無回應內容，成功完成
```

#### 錯誤回應

| 錯誤碼 | HTTP 狀態 | 描述 | 使用者訊息 |
|-------|----------|------|-----------|
| `network_error` | - | 網路連線問題 | "網路連線失敗，但本地登出已完成" |
| `session_not_found` | 404 | 會話不存在（已過期或已登出） | "您已成功登出" |

### 處理範例

```dart
try {
  await supabase.auth.signOut();
  // 清除本地儲存
  await localStorage.clearAll();
  // 成功登出
} on AuthException catch (e) {
  // 即使伺服器登出失敗，也要清除本地儲存
  await localStorage.clearAll();
  
  if (e.statusCode == '404') {
    // 會話不存在，視為成功
    return;
  } else {
    // 記錄錯誤但不阻止 UI 導向登入頁
    logger.error('Logout failed', error: e);
  }
}
```

---

## 4. 檢查認證狀態

### 端點資訊

- **服務**: Supabase Auth
- **方法**: `currentSession` (屬性)
- **用途**: 取得當前會話狀態

### 請求

```dart
final session = supabase.auth.currentSession;
```

**參數**: 無（只讀屬性）

### 回應

#### 有效會話

```dart
Session {
  accessToken: "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  refreshToken: "refresh_token_...",
  expiresIn: 3600,
  expiresAt: 1700000000,
  user: User { /* ... */ }
}
```

#### 無會話

```dart
null
```

### 使用範例

```dart
// APP 啟動時檢查
Future<AuthState> checkInitialAuthState() async {
  final session = supabase.auth.currentSession;
  
  if (session == null) {
    // 未登入
    final lastProvider = await localStorage.getLastAuthProvider();
    return Unauthenticated(lastUsedProvider: lastProvider);
  }
  
  if (session.isExpired) {
    // 會話已過期，嘗試更新
    try {
      final refreshed = await supabase.auth.refreshSession();
      return Authenticated(
        user: refreshed.user!,
        session: refreshed.session!,
      );
    } catch (e) {
      // 更新失敗，清除會話
      await supabase.auth.signOut();
      return Unauthenticated();
    }
  }
  
  // 會話有效
  return Authenticated(
    user: session.user,
    session: session,
  );
}
```

---

## 5. 更新會話 Token

### 端點資訊

- **服務**: Supabase Auth
- **方法**: `refreshSession`
- **用途**: 使用 Refresh Token 取得新的 Access Token

### 請求

```dart
final response = await supabase.auth.refreshSession();
```

**參數**: 無（自動使用當前的 Refresh Token）

### 回應

#### 成功回應

```dart
AuthResponse {
  session: Session {
    accessToken: "new_access_token...",
    refreshToken: "new_refresh_token...",
    expiresIn: 3600,
    expiresAt: 1700003600
  },
  user: User { /* ... */ }
}
```

#### 錯誤回應

| 錯誤碼 | HTTP 狀態 | 描述 | 處理方式 |
|-------|----------|------|---------|
| `invalid_grant` | 400 | Refresh Token 無效或已過期 | 清除會話，導向登入頁 |
| `network_error` | - | 網路連線問題 | 重試或提示使用者 |

### 自動更新策略

```dart
// 監聽認證狀態變化
supabase.auth.onAuthStateChange.listen((data) {
  final event = data.event;
  final session = data.session;
  
  if (event == AuthChangeEvent.tokenRefreshed) {
    // Token 已自動更新
    logger.info('Token refreshed automatically');
    
    // 更新本地儲存
    if (session != null) {
      localStorage.saveAccessToken(session.accessToken);
      localStorage.saveRefreshToken(session.refreshToken!);
    }
  } else if (event == AuthChangeEvent.signedOut) {
    // 會話已過期且無法更新
    logger.info('Session expired, user signed out');
  }
});
```

---

## 6. 取得使用者資訊

### 端點資訊

- **服務**: Supabase Auth
- **方法**: `currentUser` (屬性)
- **用途**: 取得當前登入使用者的資訊

### 請求

```dart
final user = supabase.auth.currentUser;
```

**參數**: 無（只讀屬性）

### 回應

#### 已登入

```dart
User {
  id: "550e8400-e29b-41d4-a716-446655440000",
  email: "user@gmail.com",
  emailConfirmedAt: "2025-11-16T10:00:00Z",
  phone: null,
  createdAt: "2025-11-16T10:00:00Z",
  updatedAt: "2025-11-16T10:00:00Z",
  userMetadata: {
    "name": "User Name",
    "avatar_url": "https://...",
    "provider": "google"
  },
  appMetadata: {
    "provider": "google",
    "providers": ["google"]
  }
}
```

#### 未登入

```dart
null
```

---

## 錯誤碼總覽

| 錯誤碼 | HTTP 狀態 | 發生情境 | 建議處理 |
|-------|----------|---------|---------|
| `user_cancelled` | - | 使用者取消 OAuth 授權 | 返回登入頁，顯示提示 |
| `invalid_grant` | 400 | Token 無效或已過期 | 清除會話，重新登入 |
| `invalid_credentials` | 401 | 憑證無效 | 重新登入 |
| `network_error` | - | 網路連線失敗 | 顯示錯誤，提供重試 |
| `provider_error` | 500 | OAuth 供應商服務錯誤 | 顯示錯誤，稍後重試 |
| `rate_limit_exceeded` | 429 | 請求過於頻繁 | 延遲重試，顯示提示 |
| `session_not_found` | 404 | 會話不存在 | 視為登出成功 |

---

## 安全性考量

### 1. Token 儲存

- **Access Token**: 使用 `flutter_secure_storage` 加密儲存
- **Refresh Token**: 同樣加密儲存，永不暴露於 UI 或日誌
- **過期處理**: 定期檢查並自動更新即將過期的 token

### 2. HTTPS 強制

所有 API 請求必須使用 HTTPS，Supabase 預設強制 HTTPS。

### 3. CORS 設定

Supabase Dashboard 中設定允許的 Redirect URI:
- `io.supabase.moneymanager://login-callback`（生產環境）
- `io.supabase.moneymanager.dev://login-callback`（開發環境）

### 4. RLS (Row Level Security)

所有資料表啟用 RLS，確保使用者只能存取自己的資料。

---

## 測試

### Mock 資料

```dart
// test/mocks/mock_auth_response.dart
final mockGoogleAuthResponse = AuthResponse(
  user: User(
    id: 'test-user-id',
    email: 'test@gmail.com',
    userMetadata: {
      'name': 'Test User',
      'avatar_url': 'https://test.com/avatar.jpg',
      'provider': 'google',
    },
  ),
  session: Session(
    accessToken: 'mock_access_token',
    refreshToken: 'mock_refresh_token',
    expiresIn: 3600,
    expiresAt: DateTime.now().add(Duration(hours: 1)).millisecondsSinceEpoch ~/ 1000,
  ),
);
```

### 單元測試範例

```dart
// test/features/auth/data/repositories/auth_repository_test.dart
test('Google sign in returns user on success', () async {
  // Arrange
  when(mockSupabaseClient.auth.signInWithOAuth(any))
      .thenAnswer((_) async => mockGoogleAuthResponse);

  // Act
  final result = await repository.signInWithGoogle();

  // Assert
  expect(result.user, isNotNull);
  expect(result.user!.email, 'test@gmail.com');
  expect(result.session, isNotNull);
});
```

---

**版本**: 1.0.0  
**最後更新**: 2025-11-16  
**相關文件**: [使用者 API 合約](./user_api.md), [資料模型](../data-model.md)
