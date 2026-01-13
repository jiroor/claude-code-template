---
description: GitHubイシューを分析して修正
argument-hint: <issue-number>
allowed-tools:
  - Bash(gh:*)
  - Read
  - Write
  - Grep
  - Glob
---

# Issue #$ARGUMENTS を修正

## 1. Issue詳細を取得

!`gh issue view $ARGUMENTS`

## 2. 修正を実装

上記のIssue内容を分析し、以下の方針で修正してください:

| 方針 | 説明 |
|------|------|
| 最小限の変更 | 問題解決に必要な変更のみ |
| 既存スタイルに準拠 | プロジェクトの規約に従う |
| テスト追加 | 必要に応じてテストを追加 |

## 3. 動作確認

修正後、テストとリントを実行してください。

## 4. コミット

修正が完了したら、コミットメッセージを提案してください。
形式: `fix: 説明 (#$ARGUMENTS)`

**注意**: コミットはユーザー確認後に実行してください。
