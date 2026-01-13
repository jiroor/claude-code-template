# コーディング規約（リンター補完）

> ESLint/Prettier でカバーされない規約を定義します。
> フォーマットや構文ルールはリンター設定を参照してください。

## 命名規則

### 変数・関数

| 種類 | 規則 | 例 |
|------|------|-----|
| 変数・関数 | camelCase | `getUserData`, `isValid` |
| 定数 | UPPER_SNAKE_CASE | `MAX_RETRY_COUNT` |
| ブール値 | `is`/`has`/`can` プレフィックス | `isLoading`, `hasPermission` |
| イベントハンドラ | `handle` + イベント名 | `handleClick`, `handleSubmit` |
| カスタムフック | `use` プレフィックス | `useAuth`, `useFetch` |

### ファイル・ディレクトリ

| 種類 | 規則 | 例 |
|------|------|-----|
| コンポーネント | PascalCase | `UserProfile.tsx` |
| ユーティリティ | camelCase | `formatDate.ts` |
| 定数ファイル | camelCase | `constants.ts` |
| 型定義ファイル | camelCase | `types.ts` |
| テストファイル | `*.test.ts` / `*.spec.ts` | `utils.test.ts` |

## コメント

- 「なぜ」を説明するコメントを優先（「何を」はコードで表現）
- TODO/FIXME には担当者と日付を記載
  ```typescript
  // TODO(username): 説明 - 2025-01-13
  ```
- JSDoc は公開API・エクスポート関数にのみ使用

## インポート順序

```typescript
// 1. Node.js 組み込みモジュール
import path from 'path';

// 2. 外部ライブラリ
import React from 'react';

// 3. 内部モジュール（絶対パス）
import { Button } from '@/components/ui';

// 4. 相対パス
import { helper } from './utils';

// 5. 型定義（type import）
import type { User } from '@/types';
```
