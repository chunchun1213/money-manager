# 使用者資料 API 合約

**版本**: 1.0.0  
**日期**: 2025-11-16  
**平台**: Supabase Database (PostgreSQL)

## 概述

本文件定義使用者資料和偏好設定的 API 合約，基於 Supabase Database REST API。所有端點受 Row Level Security (RLS) 保護。

---

## 基礎設定

### Base URL

```
https://your-project.supabase.co/rest/v1
```

### Headers

```http
apikey: YOUR_SUPABASE_ANON_KEY
Authorization: Bearer USER_ACCESS_TOKEN
Content-Type: application/json
```

---

## 1. 取得使用者資料

### 端點

```
GET /users?id=eq.{userId}
```

### 請求

**路徑參數**: 無  
**查詢參數**:
- `id`: (required) 使用者 UUID，格式 `eq.{userId}`

**Headers**:
```http
Authorization: Bearer {access_token}
```

**範例**:
```dart
final response = await supabase
    .from('users')
    .select()
    .eq('id', userId)
    .single();
```

### 回應

#### 成功回應 (200 OK)

```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "email": "user@gmail.com",
  "display_name": "User Name",
  "photo_url": "https://lh3.googleusercontent.com/...",
  "auth_provider": "GOOGLE",
  "created_at": "2025-11-16T10:00:00.000Z",
  "updated_at": "2025-11-16T10:00:00.000Z"
}
```

**欄位說明**:
| 欄位 | 類型 | 描述 |
|-----|------|------|
| `id` | UUID | 使用者唯一識別碼 |
| `email` | String | 使用者電子郵件 |
| `display_name` | String? | 顯示名稱（可為 null） |
| `photo_url` | String? | 頭像 URL（可為 null） |
| `auth_provider` | String | 登入提供商 (GOOGLE/FACEBOOK) |
| `created_at` | ISO 8601 | 建立時間 |
| `updated_at` | ISO 8601 | 最後更新時間 |

#### 錯誤回應

| HTTP 狀態 | 描述 | 回應範例 |
|----------|------|---------|
| 401 | 未授權（Token 無效或過期） | `{"message":"JWT expired"}` |
| 403 | 禁止存取（嘗試存取他人資料） | `{"message":"Row level security policy violation"}` |
| 404 | 使用者不存在 | `{"message":"No rows found"}` |

---

## 2. 建立使用者資料

### 端點

```
POST /users
```

### 請求

**Headers**:
```http
Authorization: Bearer {access_token}
Content-Type: application/json
```

**Body**:
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "email": "user@gmail.com",
  "display_name": "User Name",
  "photo_url": "https://lh3.googleusercontent.com/...",
  "auth_provider": "GOOGLE"
}
```

**必填欄位**:
- `id`: 使用者 UUID（必須與 auth.users.id 一致）
- `email`: 電子郵件
- `auth_provider`: GOOGLE 或 FACEBOOK

**可選欄位**:
- `display_name`: 顯示名稱
- `photo_url`: 頭像 URL

**Dart 範例**:
```dart
final response = await supabase.from('users').insert({
  'id': userId,
  'email': email,
  'display_name': displayName,
  'photo_url': photoUrl,
  'auth_provider': authProvider.name.toUpperCase(),
}).select().single();
```

### 回應

#### 成功回應 (201 Created)

```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "email": "user@gmail.com",
  "display_name": "User Name",
  "photo_url": "https://lh3.googleusercontent.com/...",
  "auth_provider": "GOOGLE",
  "created_at": "2025-11-16T10:00:00.000Z",
  "updated_at": "2025-11-16T10:00:00.000Z"
}
```

#### 錯誤回應

| HTTP 狀態 | 描述 | 回應範例 |
|----------|------|---------|
| 400 | 請求格式錯誤或違反約束 | `{"message":"duplicate key value violates unique constraint"}` |
| 401 | 未授權 | `{"message":"JWT expired"}` |
| 403 | 禁止建立（ID 不符合當前使用者） | `{"message":"Row level security policy violation"}` |
| 409 | 衝突（使用者已存在） | `{"message":"Users already exists"}` |

---

## 3. 更新使用者資料

### 端點

```
PATCH /users?id=eq.{userId}
```

### 請求

**查詢參數**:
- `id`: (required) 使用者 UUID

**Headers**:
```http
Authorization: Bearer {access_token}
Content-Type: application/json
```

**Body**:
```json
{
  "display_name": "New Display Name",
  "photo_url": "https://new-avatar-url.com/..."
}
```

**可更新欄位**:
- `display_name`: 顯示名稱
- `photo_url`: 頭像 URL

**不可更新欄位**:
- `id`: 識別碼（不可變）
- `email`: 電子郵件（不可變，需透過 Supabase Auth 更新）
- `auth_provider`: 登入提供商（不可變）

**Dart 範例**:
```dart
final response = await supabase
    .from('users')
    .update({
      'display_name': newDisplayName,
      'photo_url': newPhotoUrl,
    })
    .eq('id', userId)
    .select()
    .single();
```

### 回應

#### 成功回應 (200 OK)

```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "email": "user@gmail.com",
  "display_name": "New Display Name",
  "photo_url": "https://new-avatar-url.com/...",
  "auth_provider": "GOOGLE",
  "created_at": "2025-11-16T10:00:00.000Z",
  "updated_at": "2025-11-16T10:30:00.000Z"
}
```

#### 錯誤回應

| HTTP 狀態 | 描述 | 回應範例 |
|----------|------|---------|
| 400 | 請求格式錯誤 | `{"message":"Invalid input"}` |
| 401 | 未授權 | `{"message":"JWT expired"}` |
| 403 | 禁止更新（嘗試更新他人資料） | `{"message":"Row level security policy violation"}` |
| 404 | 使用者不存在 | `{"message":"No rows found"}` |

---

## 4. 記錄上次使用的登入提供商

### 端點

```
POST /user_preferences
```

或

```
PATCH /user_preferences?user_id=eq.{userId}
```

### 請求

**Headers**:
```http
Authorization: Bearer {access_token}
Content-Type: application/json
```

**Body (POST - 新建)**:
```json
{
  "user_id": "550e8400-e29b-41d4-a716-446655440000",
  "last_auth_provider": "GOOGLE"
}
```

**Body (PATCH - 更新)**:
```json
{
  "last_auth_provider": "FACEBOOK"
}
```

**Dart 範例**:
```dart
// Upsert 模式（不存在則新建，存在則更新）
await supabase.from('user_preferences').upsert({
  'user_id': userId,
  'last_auth_provider': provider.name.toUpperCase(),
});
```

### 回應

#### 成功回應 (200 OK / 201 Created)

```json
{
  "user_id": "550e8400-e29b-41d4-a716-446655440000",
  "last_auth_provider": "GOOGLE",
  "updated_at": "2025-11-16T10:30:00.000Z"
}
```

---

## 5. 取得上次使用的登入提供商

### 端點

```
GET /user_preferences?user_id=eq.{userId}
```

### 請求

**查詢參數**:
- `user_id`: (required) 使用者 UUID

**Headers**:
```http
Authorization: Bearer {access_token}
```

**Dart 範例**:
```dart
final response = await supabase
    .from('user_preferences')
    .select('last_auth_provider')
    .eq('user_id', userId)
    .maybeSingle();

final lastProvider = response?['last_auth_provider'];
```

### 回應

#### 成功回應 (200 OK)

```json
{
  "user_id": "550e8400-e29b-41d4-a716-446655440000",
  "last_auth_provider": "GOOGLE",
  "updated_at": "2025-11-16T10:00:00.000Z"
}
```

#### 無資料回應 (200 OK with null)

```json
null
```

---

## 資料庫 Schema

### users 表格

```sql
CREATE TABLE users (
  id UUID PRIMARY KEY REFERENCES auth.users(id) ON DELETE CASCADE,
  email TEXT NOT NULL UNIQUE,
  display_name TEXT,
  photo_url TEXT,
  auth_provider TEXT NOT NULL CHECK (auth_provider IN ('GOOGLE', 'FACEBOOK')),
  created_at TIMESTAMPTZ DEFAULT NOW() NOT NULL,
  updated_at TIMESTAMPTZ DEFAULT NOW() NOT NULL
);
```

### user_preferences 表格

```sql
CREATE TABLE user_preferences (
  user_id UUID PRIMARY KEY REFERENCES users(id) ON DELETE CASCADE,
  last_auth_provider TEXT CHECK (last_auth_provider IN ('GOOGLE', 'FACEBOOK')),
  updated_at TIMESTAMPTZ DEFAULT NOW() NOT NULL
);
```

---

## Row Level Security (RLS) 政策

### users 表格政策

```sql
-- 使用者只能查看自己的資料
CREATE POLICY "Users can view own data"
  ON users FOR SELECT
  USING (auth.uid() = id);

-- 使用者只能更新自己的資料
CREATE POLICY "Users can update own data"
  ON users FOR UPDATE
  USING (auth.uid() = id);

-- 使用者只能插入自己的資料
CREATE POLICY "Users can insert own data"
  ON users FOR INSERT
  WITH CHECK (auth.uid() = id);
```

### user_preferences 表格政策

```sql
-- 使用者只能查看自己的偏好
CREATE POLICY "Users can view own preferences"
  ON user_preferences FOR SELECT
  USING (auth.uid() = user_id);

-- 使用者可以插入自己的偏好
CREATE POLICY "Users can insert own preferences"
  ON user_preferences FOR INSERT
  WITH CHECK (auth.uid() = user_id);

-- 使用者可以更新自己的偏好
CREATE POLICY "Users can update own preferences"
  ON user_preferences FOR UPDATE
  USING (auth.uid() = user_id);
```

---

## 批次操作

### 取得多個使用者資料（僅限管理功能，未來擴充）

```dart
final response = await supabase
    .from('users')
    .select()
    .in_('id', [userId1, userId2, userId3]);
```

**注意**: 此操作受 RLS 限制，使用者只能查詢自己的資料。管理員功能需額外配置。

---

## 錯誤處理

### 通用錯誤處理

```dart
try {
  final response = await supabase
      .from('users')
      .select()
      .eq('id', userId)
      .single();
  
  return UserModel.fromJson(response);
} on PostgrestException catch (e) {
  if (e.code == '23505') {
    // Unique constraint violation
    throw DuplicateUserException();
  } else if (e.code == 'PGRST116') {
    // No rows found
    throw UserNotFoundException();
  } else {
    throw DatabaseException(e.message);
  }
} catch (e) {
  throw UnexpectedException(e.toString());
}
```

### 常見 PostgreSQL 錯誤碼

| 錯誤碼 | 描述 | 處理方式 |
|-------|------|---------|
| `23505` | Unique constraint violation | 提示使用者資料已存在 |
| `23503` | Foreign key violation | 檢查參考資料是否存在 |
| `23514` | Check constraint violation | 驗證輸入值是否合法 |
| `PGRST116` | No rows found | 資料不存在，返回 null 或錯誤 |
| `PGRST301` | Row level security violation | 權限不足，禁止存取 |

---

## 測試

### Mock 資料

```dart
// test/mocks/mock_user_data.dart
final mockUserData = {
  'id': 'test-user-id',
  'email': 'test@gmail.com',
  'display_name': 'Test User',
  'photo_url': 'https://test.com/avatar.jpg',
  'auth_provider': 'GOOGLE',
  'created_at': '2025-11-16T10:00:00.000Z',
  'updated_at': '2025-11-16T10:00:00.000Z',
};
```

### 單元測試範例

```dart
// test/features/auth/data/repositories/user_repository_test.dart
test('getUserData returns user on success', () async {
  // Arrange
  when(mockSupabaseClient.from('users'))
      .thenReturn(mockQueryBuilder);
  when(mockQueryBuilder.select())
      .thenReturn(mockFilterBuilder);
  when(mockFilterBuilder.eq('id', any))
      .thenReturn(mockFilterBuilder);
  when(mockFilterBuilder.single())
      .thenAnswer((_) async => mockUserData);

  // Act
  final result = await repository.getUserData(userId);

  // Assert
  expect(result.id, 'test-user-id');
  expect(result.email, 'test@gmail.com');
  expect(result.authProvider, AuthProvider.google);
});
```

---

## 效能考量

### 索引

```sql
-- Email 查詢索引（用於登入查找）
CREATE INDEX idx_users_email ON users(email);

-- Auth Provider 索引（用於統計分析）
CREATE INDEX idx_users_auth_provider ON users(auth_provider);
```

### 快取策略

- **使用者資料**: 登入後快取至記憶體，僅在更新時重新載入
- **偏好設定**: 快取至本地儲存，減少 API 呼叫

---

**版本**: 1.0.0  
**最後更新**: 2025-11-16  
**相關文件**: [認證 API 合約](./auth_api.md), [資料模型](../data-model.md)
