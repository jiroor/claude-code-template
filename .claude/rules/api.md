---
paths:
  - "src/api/**/*"
  - "src/app/api/**/*"
---

# API開発ルール

> API関連ファイルにのみ適用されるルールです。

## エンドポイント設計

| 規則 | 例 |
|------|-----|
| リソース名は複数形 | `/users`, `/posts` |
| ネストは2階層まで | `/users/:id/posts` |
| アクションはHTTPメソッドで表現 | `POST /users`（作成） |

## バリデーション

すべてのエンドポイントで入力バリデーションを実装:

```typescript
import { z } from 'zod';

const schema = z.object({
  name: z.string().min(1).max(100),
  email: z.string().email(),
});

const validated = schema.parse(body);
```

## エラーレスポンス形式

```json
{
  "error": {
    "code": "ERROR_CODE",
    "message": "エラーメッセージ",
    "details": []
  }
}
```

## 認証・認可

- 認証: Bearer トークンを使用
- 認可: 各エンドポイントの先頭で検証
