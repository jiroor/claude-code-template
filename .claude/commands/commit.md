---
description: Conventional Commits形式でコミットを作成
allowed-tools:
  - Bash(git:*)
---

# コミット作成

## 変更内容を確認

!`git status`
!`git diff --cached`

ステージングされていない場合:
!`git diff`

## コミットメッセージ生成

変更内容を分析し、以下の形式でメッセージを生成してください:

```
<type>(<scope>): <subject>

<body>
```

### Type一覧

| Type | 用途 |
|------|------|
| `feat` | 新機能 |
| `fix` | バグ修正 |
| `docs` | ドキュメント |
| `style` | フォーマット |
| `refactor` | リファクタリング |
| `perf` | パフォーマンス改善 |
| `test` | テスト |
| `chore` | ビルド・ツール |

## 確認

生成したコミットメッセージを表示し、ユーザーの承認を待ってください。

**重要**: 明示的な承認なしにコミットを実行しないでください。
