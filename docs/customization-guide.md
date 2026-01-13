# Claude Code カスタマイズ機能ガイド
## CLAUDE.md, Rules, Skills, Agents, Custom Commands の完全比較

**対応バージョン: Claude Code 2.1.4（2025年1月12日時点最新）**

---

## 概要

Claude Code には、Claudeの振る舞いをカスタマイズするための5つの主要な仕組みがあります。それぞれ異なる目的と責務を持ち、適切に使い分けることで最大の効果を発揮します。

| 機能 | 発動方式 | 主な用途 | スコープ |
|------|----------|----------|----------|
| **CLAUDE.md** | 自動（常時ロード） | プロジェクト全体の設定・規約 | プロジェクト/ユーザー |
| **Rules** | 自動（パス条件付き可） | ファイル種別ごとのルール | プロジェクト/ユーザー |
| **Skills** | モデル自動判断 | 専門的なワークフロー | プロジェクト/ユーザー/プラグイン |
| **Subagents** | モデル自動判断/明示的 | 独立したタスク実行 | プロジェクト/ユーザー/プラグイン |
| **Custom Commands** | ユーザー明示的（`/command`） | 定型プロンプト | プロジェクト/ユーザー |

---

## 1. CLAUDE.md（メモリファイル）

### 役割と責務

CLAUDE.mdは、Claude Codeが**セッション開始時に自動的に読み込む設定ファイル**です。プロジェクトの「記憶」として機能し、常にコンテキストに含まれます。

### 階層構造と優先順位

| 種類 | 場所 | 用途 |
|------|------|------|
| **エンタープライズポリシー** | `/Library/Application Support/ClaudeCode/CLAUDE.md` (macOS) | 組織全体のポリシー |
| **プロジェクトメモリ** | `./CLAUDE.md` または `./.claude/CLAUDE.md` | チーム共有の設定 |
| **ユーザーメモリ** | `~/.claude/CLAUDE.md` | 個人の全プロジェクト設定 |
| **ローカルメモリ** | `./CLAUDE.local.md` | 個人のプロジェクト固有設定（gitignore対象） |

### 最適な書き方

```markdown
# プロジェクト名

## 技術スタック
- TypeScript 5.3
- React 18
- PostgreSQL 15

## ビルドコマンド
- `npm run dev` - 開発サーバー起動
- `npm run test` - テスト実行
- `npm run lint` - リンター実行

## コーディング規約
- 2スペースインデント
- import文は destructure を使用
- コンポーネントは関数コンポーネントで作成

## 禁止事項
- `any` 型の使用禁止（代替: `unknown` を使用）
- console.log のコミット禁止
```

### ベストプラクティス

1. **簡潔に書く**: 長すぎるとコンテキストを圧迫
2. **Progressive Disclosure**: 詳細情報は別ファイルに分離し、`@path/to/file` で参照
3. **具体的に**: 「コードを適切にフォーマット」ではなく「2スペースインデント」
4. **リンターの代わりにしない**: スタイルルールはリンターに任せる
5. **否定だけで終わらない**: 「〜を使わない」だけでなく代替案も提示

### インポート機能

```markdown
# プロジェクト設定

@README.md の概要を参照
@package.json のnpmコマンドを参照

## 個人設定
@~/.claude/my-project-instructions.md
```

---

## 2. Rules（ルールディレクトリ）

### 役割と責務

Rulesは**CLAUDE.mdを分割してモジュール化するための仕組み**です。v2.0.64で導入されました。大きくなりがちなCLAUDE.mdを、トピックごとに分割した個別のMarkdownファイルとして管理できます。

### CLAUDE.md との違い

| 観点 | CLAUDE.md | Rules |
|------|-----------|-------|
| 構造 | 単一ファイル | 複数ファイルに分割可能 |
| 条件付き適用 | 不可 | `paths` で特定ファイルのみに適用可能 |
| 優先度 | 同じ | 同じ（プロジェクトレベル） |
| 主な用途 | プロジェクト全体の方針 | ドメイン別のルール |

### ディレクトリ構造

```
your-project/
├── .claude/
│   ├── CLAUDE.md           # メインのプロジェクト設定
│   └── rules/
│       ├── code-style.md   # コードスタイル
│       ├── testing.md      # テスト規約
│       ├── security.md     # セキュリティ要件
│       └── frontend/       # サブディレクトリも可
│           ├── react.md
│           └── styling.md
```

### 条件付きルール（パス指定）

特定のファイルパターンにのみ適用されるルールを定義できます：

```yaml
---
paths:
  - "src/api/**/*.ts"
---

# API開発ルール
- すべてのエンドポイントで入力バリデーションを実装
- 標準エラーレスポンス形式を使用
- OpenAPIドキュメントコメントを含める
```

**グロブパターンの例：**
- `src/api/**/*.ts` - src/api配下の全.tsファイル
- `**/*.test.ts` - 全テストファイル
- `src/components/**/*` - コンポーネント配下全体

### 配置場所

| 種類 | 場所 | 優先度 |
|------|------|--------|
| プロジェクトルール | `.claude/rules/` | 高 |
| ユーザールール | `~/.claude/rules/` | 低 |

### ベストプラクティス

1. **1ファイル1トピック**: testing.md, security.md など分離
2. **わかりやすいファイル名**: rules1.md ではなく api-validation.md
3. **条件付きルールは控えめに**: 本当に必要な場合のみ`paths`を使用
4. **500行以下に抑える**: 長すぎる場合は分割を検討
5. **シンボリックリンク活用**: 共有ルールを複数プロジェクトで使い回し

```bash
# 共有ルールをシンボリックリンク
ln -s ~/shared-claude-rules .claude/rules/shared
ln -s ~/company-standards/security.md .claude/rules/security.md
```

### CLAUDE.md と Rules の使い分け

**CLAUDE.md に書くべき内容：**
- プロジェクト全体に適用される方針
- オーケストレーションロジック（サブエージェントの使い分け等）
- 品質基準、協調プロトコル

**Rules に書くべき内容：**
- 特定ドメインのルール（API、フロントエンド、テスト等）
- ファイルタイプ固有のパターン
- チーム固有の規約（マージコンフリクト軽減にも有効）

---

## 3. Agent Skills（エージェントスキル）

### 役割と責務

Skillsは**Claudeが自動的に発見し、必要に応じて読み込む専門知識パッケージ**です。2025年10月に発表され、12月にはオープンスタンダードとして公開されました。

### 重要な変更（v2.1.3）

**Slash CommandsとSkillsが統合されました。** メンタルモデルが簡素化され、両者は同じ`Skill`ツールで扱われるようになりました（動作に変更なし）。

### Skillsの仕組み

1. Claude Codeは起動時に全Skillsの`name`と`description`をシステムプロンプトに読み込む
2. ユーザーのリクエストに基づき、Claudeが関連するSkillを自動判断
3. 必要なSkillの`SKILL.md`を読み込み、コンテキストに注入
4. **Progressive Disclosure**: 関連するファイルは必要に応じて追加読み込み

### ディレクトリ構造

```
my-skill/
├── SKILL.md           # 必須：メイン指示書
├── REFERENCE.md       # オプション：参照ドキュメント
├── EXAMPLES.md        # オプション：使用例
├── scripts/
│   └── helper.py      # オプション：実行スクリプト
└── templates/
    └── template.txt   # オプション：テンプレート
```

### SKILL.md の書き方

```yaml
---
name: pdf-processing
description: Extract text, fill forms, merge PDFs. Use when working with PDF files, forms, or document extraction.
allowed-tools: Read, Bash, Write  # オプション：ツール制限
---

# PDF Processing

## Instructions
1. PDFファイルを読み込む
2. テキストを抽出する
3. 必要に応じてフォームを処理する

## Examples
`python scripts/extract.py document.pdf`
```

### 配置場所

| 種類 | 場所 |
|------|------|
| 個人Skill | `~/.claude/skills/skill-name/` |
| プロジェクトSkill | `.claude/skills/skill-name/` |
| プラグインSkill | プラグインの `skills/` ディレクトリ |

### ベストプラクティス

1. **descriptionが最重要**: Claudeがスキルを発見するための鍵
2. **「何をするか」と「いつ使うか」を両方記載**
3. **フォーカスを絞る**: 1つのSkillは1つの機能に特化
4. **スクリプトは補助**: 複雑な処理はスクリプトに委譲

### 最近のアップデート（2025年12月）

- **オープンスタンダード化**: クロスプラットフォーム互換性
- **Skills Directory**: Atlassian, Canva, Notion, Figmaなどパートナー製Skillsを提供
- **組織管理機能**: Team/Enterpriseプランでの一元管理

---

## 4. Subagents（サブエージェント）

### 役割と責務

Subagentsは**独立したコンテキストウィンドウを持つ専門的なAIアシスタント**です。メインの会話とは分離されたコンテキストで動作し、結果のみを返します。

### 組み込みSubagents

| 名前 | モデル | 用途 |
|------|--------|------|
| **Explore** | Haiku | 高速な読み取り専用検索・コード探索 |
| **Plan** | Sonnet | プランモードでの調査・分析 |
| **General-purpose** | Sonnet | 複雑なマルチステップタスク |

### カスタムSubagentの作成

```yaml
---
name: code-reviewer
description: Expert code review specialist. Use immediately after writing or modifying code.
tools: Read, Grep, Glob, Bash
model: sonnet  # または inherit, opus, haiku
permissionMode: default
skills: security-checklist  # オプション：自動読み込みするSkill
---

あなたはシニアコードレビュアーです。

レビュー時の手順:
1. git diff で変更を確認
2. セキュリティ問題をチェック
3. パフォーマンスを評価

## レビューチェックリスト
- コードの可読性
- エラーハンドリング
- テストカバレッジ
```

### 配置場所

| 種類 | 場所 | 優先度 |
|------|------|--------|
| プロジェクト | `.claude/agents/` | 最高 |
| CLI定義 | `--agents '{...}'` | 中 |
| ユーザー | `~/.claude/agents/` | 低 |

### Skills vs Subagents の使い分け

| 観点 | Skills | Subagents |
|------|--------|-----------|
| コンテキスト | メイン会話に注入 | 独立したコンテキスト |
| 用途 | 知識・ガイダンス | 独立したタスク実行 |
| 出力 | コンテキストに追加 | サマリーのみ返却 |
| 適したケース | 標準やベストプラクティスの適用 | 大量出力を伴う処理の隔離 |

### 最新機能（Claude Code 2.1.0）

- **ツール拒否後も継続**: ツール使用を拒否されても停止しない
- **再開可能**: `resume`パラメータで以前の会話を継続
- **Skillsの自動読み込み**: `skills`フィールドでSkillを指定可能

---

## 5. Custom Commands（カスタムコマンド）

### 役割と責務

Custom Commandsは**ユーザーが明示的に`/command`で呼び出す定型プロンプト**です。頻繁に使うワークフローをショートカット化します。

### 作成方法

```bash
# プロジェクトコマンド
mkdir -p .claude/commands
echo "このコードをパフォーマンス観点で分析してください" > .claude/commands/optimize.md

# 個人コマンド  
mkdir -p ~/.claude/commands
echo "セキュリティ脆弱性をチェックしてください" > ~/.claude/commands/security.md
```

### 高度な機能

#### 引数の使用

```markdown
# .claude/commands/fix-issue.md
---
argument-hint: [issue-number]
description: GitHubイシューを修正
---

GitHub Issue #$ARGUMENTS を分析し修正してください。
```

使用例: `/fix-issue 123`

#### Bashコマンドの実行

```markdown
---
allowed-tools: Bash(git:*)
description: コミット作成
---

## 現在の状態
!`git status`
!`git diff HEAD`

上記の変更に基づいてコミットを作成してください。
```

#### ファイル参照

```markdown
@src/utils/helpers.js を確認し、
@docs/ARCHITECTURE.md のガイドラインに従って改善してください。
```

### Skills vs Custom Commands の比較

| 観点 | Custom Commands | Skills |
|------|-----------------|--------|
| 発動 | 明示的（`/command`） | 自動（コンテキストベース） |
| 構造 | 単一の.mdファイル | ディレクトリ + 複数ファイル |
| 複雑さ | シンプルなプロンプト | 複雑なワークフロー |
| スクリプト | 組み込み不可 | 組み込み可 |

---

## 使い分けガイド

### フローチャート

```
質問: どの機能を使うべきか？

「常にこの情報が必要」
   ├── 全体に適用 → CLAUDE.md
   │      ├── プロジェクト設定・規約
   │      ├── ビルドコマンド
   │      └── コーディングスタイル
   │
   └── 特定ファイルのみに適用 → Rules
          ├── API固有のルール（paths: src/api/**）
          ├── テスト規約（paths: **/*.test.ts）
          └── フロントエンドパターン

「Claudeが自動で判断して使ってほしい」
   ├── 「独立したコンテキストで実行」→ Subagents
   │      ├── テスト実行
   │      ├── 大量のログ分析
   │      └── コードレビュー
   │
   └── 「メイン会話にガイダンス注入」→ Skills
          ├── PDF/Excel処理
          ├── ブランドガイドライン
          └── 複雑なワークフロー

「明示的に呼び出したい」→ Custom Commands
   ├── 定型プロンプト
   ├── 頻繁に使うショートカット
   └── シンプルな操作
```

### 具体例

| シナリオ | 推奨機能 | 理由 |
|----------|----------|------|
| プロジェクトのビルドコマンド | CLAUDE.md | 常に参照が必要 |
| API固有のバリデーションルール | Rules（paths指定） | 特定ファイルにのみ適用 |
| PDF帳票の自動生成 | Skills | 専門知識とスクリプトが必要 |
| コードレビュー | Subagents | 独立コンテキストで詳細分析 |
| `git commit` のショートカット | Custom Commands | 明示的に呼び出したい |
| チームのコーディング規約 | CLAUDE.md + Rules | 基本はCLAUDE.md、詳細はRules |
| 並列タスク実行 | Subagents | 最大10並列で実行可能 |

---

## 最新アップデート情報

### Claude Code 2.1.4（2025年1月9日）

- `CLAUDE_CODE_DISABLE_BACKGROUND_TASKS` 環境変数でバックグラウンドタスク機能を無効化可能
- OAuth トークンリフレッシュの修正

### Claude Code 2.1.3（2025年1月8日）

- **Slash CommandsとSkillsの統合**: メンタルモデルの簡素化
- リリースチャンネル（stable/latest）の切り替えが `/config` で可能に
- 到達不能なパーミッションルールの検出と警告
- VSCode: パーミッション設定の保存先を選択可能なUIを追加

### Claude Code 2.1.2（2025年1月8日）

- ドラッグ＆ドロップした画像にソースパスメタデータを追加
- ファイルパスのクリック可能なハイパーリンク（iTerm等）
- Windows Package Manager（winget）サポート
- Shift+Tab でプランモードから「auto-accept edits」を素早く選択

### Claude Code 2.1.0（2025年1月7日）

**1,096コミットを含む大型アップデート**

- **Skillsホットリロード**: `~/.claude/skills` や `.claude/skills` の変更が即座に反映
- **フォークコンテキスト**: `context: fork` でSkill/コマンドを独立サブエージェントで実行
- **言語設定**: `language: "japanese"` で応答言語を指定可能
- **ワイルドカードツール権限**: `Bash(npm *)`, `Bash(*-h*)` などパターン指定
- **セッションテレポート**: `/teleport` でclaude.ai/codeへセッションを転送
- **Shift+Enter**: iTerm2, Kitty, Ghostty, WezTermで設定不要で動作
- **Ctrl+B統合**: エージェントとシェルコマンドを同時にバックグラウンド化

### Agent Skills オープンスタンダード（2025年12月18日）

- クロスプラットフォーム互換性
- パートナーSkills Directory（Atlassian, Canva, Notion, Figma等）
- 組織管理機能（Team/Enterprise）

### .claude/rules/ ディレクトリ（v2.0.64〜）

- CLAUDE.mdのモジュール化
- `paths` フィールドによる条件付きルール
- サブディレクトリ対応
- シンボリックリンクサポート

---

## 参考リンク

- [Claude Code ドキュメント](https://code.claude.com/docs)
- [メモリ管理（CLAUDE.md, Rules）](https://code.claude.com/docs/en/memory)
- [Agent Skills](https://code.claude.com/docs/en/skills)
- [Subagents](https://code.claude.com/docs/en/sub-agents)
- [Slash Commands](https://code.claude.com/docs/en/slash-commands)
- [Agent Skills エンジニアリングブログ](https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills)
- [Anthropic Skills リポジトリ](https://github.com/anthropics/skills)
- [Claude Code ベストプラクティス](https://www.anthropic.com/engineering/claude-code-best-practices)
- [Changelog](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)
