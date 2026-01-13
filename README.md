# Claude Code カスタマイズテンプレート

Claude Code の5つのカスタマイズ機能（CLAUDE.md, Rules, Skills, Subagents, Custom Commands）を体系的に整理した、すぐに使えるテンプレート集です。

## 特徴

- **実用的なサンプル**: コードレビュー、テスト実行、コミット作成など、実際のプロジェクトで使えるサンプルを収録
- **ベストプラクティス**: 二重管理を避け、Progressive Disclosure を活用した効率的な設計
- **すぐに試せる**: このリポジトリ自体が動作するサンプルとして機能
- **最新版対応**: Claude Code v2.1.4（2025年1月時点）に対応

## クイックスタート

### 1. リポジトリをクローン

```bash
git clone https://github.com/yourusername/claude-code-template.git
cd claude-code-template
```

### 2. Claude Code で開く

```bash
claude-code
```

リポジトリ自体にCLAUDE.mdと.claude/が含まれているため、すぐに動作を確認できます。

### 3. 自分のプロジェクトに適用

```bash
# プロジェクトルートで実行
cp /path/to/claude-code-template/CLAUDE.md ./
cp -r /path/to/claude-code-template/.claude ./
```

## ディレクトリ構成

```
claude-code-template/
├── README.md                           # このファイル
├── CLAUDE.md                           # プロジェクトメモリ（スリム化設計）
├── .claude/                            # Claude Code カスタマイズファイル
│   ├── rules/                          # ルールファイル
│   │   ├── _template.md                # 汎用テンプレート
│   │   ├── _template-with-paths.md     # paths指定ありテンプレート
│   │   ├── code-style.md               # コーディング規約
│   │   ├── prohibitions.md             # 禁止パターンと代替案
│   │   ├── api.md                      # API開発ルール（paths指定）
│   │   └── testing.md                  # テスト規約（paths指定）
│   ├── skills/_template/               # スキルテンプレート
│   │   ├── SKILL.md                    # メイン指示書
│   │   └── REFERENCE.md                # 参照ドキュメント
│   ├── agents/                         # サブエージェント
│   │   ├── _template.md                # 汎用テンプレート
│   │   ├── code-reviewer.md            # コードレビュー専門エージェント
│   │   └── test-runner.md              # テスト実行専門エージェント
│   └── commands/                       # カスタムコマンド
│       ├── _template.md                # 汎用テンプレート
│       ├── review.md                   # /project:review
│       ├── fix-issue.md                # /project:fix-issue <number>
│       ├── commit.md                   # /project:commit
│       └── test.md                     # /project:test [path]
└── docs/                               # ドキュメント
    ├── handoff.md                      # プロジェクト引き継ぎ文書
    ├── customization-guide.md          # 詳細カスタマイズガイド
    └── feature-relationships.md        # 機能間の関係性解説
```

## 設計方針

### CLAUDE.md はスリムに保つ

情報の二重管理を避けるため、既存ファイルを参照する設計：

| 情報 | 参照先 |
|------|--------|
| 技術スタック | `@package.json` の dependencies |
| コマンド | `@package.json` の scripts |
| フォーマット規約 | `@.prettierrc`, `@eslint.config.js` |
| 型設定 | `@tsconfig.json` |
| 詳細ルール | `.claude/rules/` |

### Progressive Disclosure

必要な情報を段階的に開示する構造：

```
CLAUDE.md（概要）
  ↓ 参照
.claude/rules/（詳細ルール）
  ↓ 使用
.claude/agents/（実行）
```

## 使い方

### 新しいルールを追加

```bash
cp .claude/rules/_template.md .claude/rules/my-rule.md
# my-rule.md を編集
```

### 新しいスキルを作成

```bash
cp -r .claude/skills/_template .claude/skills/my-skill
# SKILL.md と REFERENCE.md を編集
```

### 新しいサブエージェントを作成

```bash
cp .claude/agents/_template.md .claude/agents/my-agent.md
# my-agent.md を編集
```

### 新しいコマンドを作成

```bash
cp .claude/commands/_template.md .claude/commands/my-command.md
# my-command.md を編集
```

## サンプルの使い方

### コードレビュー

```bash
# Claude Code のCLIで実行
/project:review
```

`code-reviewer` サブエージェントが変更内容を分析し、セキュリティ・パフォーマンス・可読性の観点からレビューを実行します。

### テスト実行

```bash
/project:test
# または特定のパスを指定
/project:test src/components/
```

### コミット作成

```bash
/project:commit
```

変更内容を分析し、適切なコミットメッセージを生成してコミットを作成します。

## カスタマイズのヒント

### 1. プロジェクトに合わせてCLAUDE.mdを編集

- 技術スタック情報を更新
- ディレクトリ構成を実際の構造に合わせる
- プロジェクト固有のサブエージェント使い分けルールを追加

### 2. 不要なファイルを削除

`_template` で始まるファイルは雛形なので、使用後は削除してください。

### 3. paths指定でルールを限定

特定のファイルパターンにのみ適用されるルールを作成：

```yaml
---
paths:
  - "src/api/**/*"
---

# API開発ルール
...
```

### 4. サブエージェントにスキルを組み込む

```yaml
---
name: my-agent
skills: pdf-processing, docx
---
```

## 機能の使い分け

| 機能 | 発動方式 | 主な用途 | 適したケース |
|------|----------|----------|--------------|
| **CLAUDE.md** | 自動（常時） | プロジェクト全体の方針 | ビルドコマンド、コーディング規約 |
| **Rules** | 自動（条件付き可） | ファイル種別ごとのルール | API開発ルール、テスト規約 |
| **Skills** | モデル自動判断 | 専門知識の提供 | PDF処理、Excel操作 |
| **Subagents** | モデル自動判断/明示的 | 独立したタスク実行 | コードレビュー、大量ログ分析 |
| **Commands** | ユーザー明示的 | 定型プロンプト | /review, /commit |

## ドキュメント

- [handoff.md](docs/handoff.md) - プロジェクト引き継ぎ文書（設計判断、問題と解決策）
- [customization-guide.md](docs/customization-guide.md) - 詳細カスタマイズガイド（各機能の仕様とベストプラクティス）
- [feature-relationships.md](docs/feature-relationships.md) - 機能間の関係性解説（粒度、依存関係、レイヤー構造）

## 参考リンク

- [Claude Code ドキュメント](https://code.claude.com/docs)
- [メモリ管理（CLAUDE.md, Rules）](https://code.claude.com/docs/en/memory)
- [Agent Skills](https://code.claude.com/docs/en/skills)
- [Subagents](https://code.claude.com/docs/en/sub-agents)
- [Slash Commands](https://code.claude.com/docs/en/slash-commands)
- [Claude Code ベストプラクティス](https://www.anthropic.com/engineering/claude-code-best-practices)

## ライセンス

MIT License

## 貢献

Issue や Pull Request を歓迎します。
