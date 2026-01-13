# Claude Code カスタマイズ機能の調査と設計判断

## 目的

Claude Code の5つのカスタマイズ機能（CLAUDE.md, Rules, Skills, Subagents, Custom Commands）について、役割・責務・使い分けを整理し、実用的なテンプレートを作成した。

---

## 1. 調査結果サマリー

### 5つの機能の比較

| 機能 | 発動方式 | 主な用途 | 粒度 |
|------|----------|----------|------|
| **CLAUDE.md** | 自動（常時ロード） | プロジェクト全体の方針 | 最大 |
| **Rules** | 自動（パス条件付き可） | ファイル種別ごとのルール | 大〜中 |
| **Skills** | モデル自動判断 | 専門的なワークフロー | 中 |
| **Subagents** | モデル自動判断/明示的 | 独立したタスク実行 | 小 |
| **Custom Commands** | ユーザー明示的（`/command`） | 定型プロンプト | 小 |

### 使う/使われる関係

```
CLAUDE.md（規定する）
    ↓ 方針を定める
Rules（参照される）
    ↓ 詳細ルールを提供

         Skills
        (知識)
          ▲
          │ skills: フィールドで読み込み
          │ （完全な内容が注入される）
          │
    Subagents ────────► Commands
    (実行者)              (エントリ)
          │                   │
          └───────呼び出される──┘
```

**依存関係の詳細:**
- **CLAUDE.md**: 全機能の振る舞いを規定（呼び出しはできない）
- **Rules**: 参照されるのみ（静的な規約）
- **Subagents → Skills**: Subagentsは`skills:`フィールドで明示的にSkillsを読み込む（自動継承はされない）
- **Commands → Subagents/Skills**: CommandsからSubagentsやSkillsを呼び出し・参照可能

### レイヤー構造

| レイヤー | 機能 | 役割 |
|----------|------|------|
| Layer 0 | CLAUDE.md + Rules | 方針・規約（常時適用） |
| Layer 1 | Skills | 専門知識（動的ロード） |
| Layer 2 | Subagents + Commands | 実行（タスク単位） |

---

## 2. 設計上の問題と解決策

### 問題1: CLAUDE.md が肥大化・二重管理になりやすい

**問題点:**
- 技術スタックを直接記述 → `package.json` と二重管理
- コマンドを直接記述 → `package.json` の scripts と二重管理
- コーディング規約を直接記述 → リンター設定と二重管理
- 禁止事項を直接記述 → CLAUDE.md が肥大化

**解決策:**

| 項目 | 解決方法 |
|------|----------|
| 技術スタック | `@package.json` を参照 |
| コマンド | `@package.json` の scripts を参照 |
| コーディング規約 | リンター設定（`@eslint.config.js` 等）を参照 |
| 禁止事項 | `.claude/rules/prohibitions.md` に分離 |
| 詳細なルール | `.claude/rules/` に分離 |

**改善後の CLAUDE.md:**
```markdown
## 技術スタック
@package.json の `dependencies` および `devDependencies` を参照してください。

## コマンド
@package.json の `scripts` を参照してください。

## コーディング規約
リンター・フォーマッター設定を参照:
- @eslint.config.js
- @prettier.config.js
- @tsconfig.json

上記でカバーされない規約は `.claude/rules/` を参照してください。
```

---

### 問題2: 組み込みエージェントの記載方法

**問題点:**
- 直接記述 → アップデートで陳腐化するリスク
- 記述しない → 存在に気づきにくい

**解決策: 「存在を示唆 + 動的参照」方式**

```markdown
## サブエージェントの使い分け

### プロジェクト固有

| タスク | サブエージェント | 説明 |
|--------|------------------|------|
| コードレビュー | `code-reviewer` | 品質・セキュリティ・パフォーマンスを評価 |
| テスト実行 | `test-runner` | テスト実行と結果分析 |

### 組み込みエージェント

Claude Code には組み込みのサブエージェントがあります。
大量のファイル探索や調査タスクには、これらの使用を検討してください。

利用可能な組み込みエージェントは `/agents` コマンドで確認できます。
```

**効果:**
- 組み込みエージェントの存在には気づける
- 具体的なリストは常に最新のものを参照できる
- アップデートがあっても陳腐化しない

---

### 問題3: テンプレートの汎用性

**問題点:**
- 具体的なサンプルだけだと、新しいファイル作成時に応用しにくい

**解決策: `_template` ファイルを各ディレクトリに配置**

```
.claude/
├── rules/
│   ├── _template.md              # 汎用テンプレート
│   └── _template-with-paths.md   # paths指定ありテンプレート
├── skills/_template/
│   ├── SKILL.md
│   └── REFERENCE.md
├── agents/_template.md
└── commands/_template.md
```

新しいファイル作成時は `_template` をコピーして編集する。

---

## 3. 最新バージョン情報（2.1.4時点）

### Claude Code 2.1.3 の重要な変更

**Slash Commands と Skills の統合**
- メンタルモデルが簡素化
- 両者は同じ `Skill` ツールで扱われる（動作に変更なし）

### .claude/rules/ ディレクトリ（v2.0.64〜）

- CLAUDE.md をモジュール化する仕組み
- `paths` フィールドで特定ファイルパターンにのみ適用可能
- CLAUDE.md と同じ優先度で自動ロード
- サブディレクトリ対応

**条件付きルールの例:**
```yaml
---
paths:
  - "src/api/**/*.ts"
---

# API開発ルール
- すべてのエンドポイントで入力バリデーションを実装
```

---

## 4. テンプレート構成

作成したテンプレートの構成:

```
templates/
├── README.md                           # 使い方ガイド
├── CLAUDE.md                           # プロジェクトメモリ（スリム化済み）
└── .claude/
    ├── rules/
    │   ├── _template.md                # 汎用テンプレート
    │   ├── _template-with-paths.md     # paths指定ありテンプレート
    │   ├── code-style.md               # コーディング規約（リンター補完）
    │   ├── prohibitions.md             # 禁止パターンと代替案
    │   ├── api.md                      # API開発ルール（paths指定）
    │   └── testing.md                  # テスト規約（paths指定）
    │
    ├── skills/_template/
    │   ├── SKILL.md                    # メイン指示書
    │   └── REFERENCE.md                # 参照ドキュメント
    │
    ├── agents/
    │   ├── _template.md                # 汎用テンプレート
    │   ├── code-reviewer.md            # コードレビュー用
    │   └── test-runner.md              # テスト実行用
    │
    └── commands/
        ├── _template.md                # 汎用テンプレート
        ├── review.md                   # /project:review
        ├── fix-issue.md                # /project:fix-issue <number>
        ├── commit.md                   # /project:commit
        └── test.md                     # /project:test [path]
```

---

## 5. 使い分けフローチャート

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

---

## 6. 参考リンク

- [Claude Code ドキュメント](https://code.claude.com/docs)
- [メモリ管理（CLAUDE.md, Rules）](https://code.claude.com/docs/en/memory)
- [Agent Skills](https://code.claude.com/docs/en/skills)
- [Subagents](https://code.claude.com/docs/en/sub-agents)
- [Slash Commands](https://code.claude.com/docs/en/slash-commands)
- [Claude Code ベストプラクティス](https://www.anthropic.com/engineering/claude-code-best-practices)

---

## 7. 添付ファイル

このドキュメントと一緒に以下のファイルがあります:

- `claude-code-templates.zip`: 上記テンプレート一式
- `claude_code_customization_report.md`: 詳細な調査レポート
- `claude_code_feature_relationships.md`: 機能間の関係性の詳細分析
