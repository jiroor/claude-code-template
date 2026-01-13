---
paths:
  - "**/*.test.ts"
  - "**/*.test.tsx"
  - "**/*.spec.ts"
  - "**/*.spec.tsx"
  - "tests/**/*"
  - "__tests__/**/*"
---

# テスト規約

> テストファイルにのみ適用されるルールです。

## テスト構造

AAA パターン（Arrange-Act-Assert）に従う:

```typescript
it('should create a user with valid input', () => {
  // Arrange: 準備
  const input = { name: 'Test', email: 'test@example.com' };
  
  // Act: 実行
  const result = createUser(input);
  
  // Assert: 検証
  expect(result.name).toBe('Test');
});
```

## 命名規則

| 要素 | 規則 | 例 |
|------|------|-----|
| `describe` | テスト対象の名前 | `describe('UserService', ...)` |
| `it`/`test` | should + 動作 + when + 条件 | `it('should throw when email is invalid', ...)` |

## モック

| 対象 | 方針 |
|------|------|
| 外部API | 必ずモック |
| データベース | テスト用DBまたはインメモリ |
| 時刻 | 固定値を使用 |
| 環境変数 | テスト用の値を設定 |

## カバレッジ目標

| 種類 | 目標 |
|------|------|
| ステートメント | 80%以上 |
| ブランチ | 70%以上 |
| 重要なビジネスロジック | 100% |
