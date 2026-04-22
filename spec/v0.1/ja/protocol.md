---
spec-id: ACE-001
version: 0.1.1
status: draft
created: 2026-04-14
updated: 2026-04-22
ai-context: claude-code
related-specs:
  - AMP v0.1 (github.com/Akari-OS/amp)
  - M2C v0.2 (github.com/Akari-OS/m2c)
language: ja
---

# ACE フレームワーク仕様 — v0.1（ドラフト）

> 原本: [../protocol.md](../protocol.md) — 英語版が正典

> **ACE** — Agent Context Engineering Framework（エージェント・コンテキスト・エンジニアリング・フレームワーク）。
> AI エージェントがリポジトリのコンテキストファイルを読み取り、尊重し、推論する方法を標準化する。
> 本ドキュメントは ACE v0.1（ドラフト）の規範的仕様書である。

---

## 0. 概要

AI コーディングエージェントはリポジトリのファイル・設定・自然言語指示を消費してコードと助言を生成する。
現在、すべてのエージェントフレームワーク（Claude Code、Cursor、Copilot、Windsurf、Cline、Aider、…）は
以下の3つの共通課題を互換性のない方法で再実装している:

1. **アクセス制御** — エージェントが読み取ってよいファイル
2. **コンテキストファイル形式** — 指示のエンコード方法（frontmatter、アクティベーション、スコーピング）
3. **バリデーション** — 古い・矛盾した・悪意のあるコンテキストの検出方法

ACE はこの3つに対して最小限でベンダー中立な契約を定義し、
さらに既存ファイル（`AGENTS.md`、`CLAUDE.md`、`.cursor/rules/*.mdc`、
`copilot-instructions.md`、Windsurf ルール）を書き換えなしに参加させる **アダプター層** を提供する。
ACE は既存のルートファイル規約を置き換えない。それらをラップする。

v0.1 は意図的に狭い範囲に絞っている: `.acp-ignore` + frontmatter スキーマ +
構造的 lint サーフェス + アダプターヒント。
セマンティック（LLM 駆動）lint、エンタープライズ由来証明機能（SBOM / SLSA Level 3 / 90日パッチ SLA）、
および署名付き適合性テストは後続バージョンに先送りする。

---

## 1. 適合性

### 1.1 キーワード

本ドキュメントにおいて **MUST**（しなければならない）、**MUST NOT**（してはならない）、
**REQUIRED**（必須）、**SHALL**（するものとする）、**SHALL NOT**（しないものとする）、
**SHOULD**（すべきである）、**SHOULD NOT**（すべきでない）、**RECOMMENDED**（推奨する）、
**MAY**（してもよい）、**OPTIONAL**（任意）という語は、すべて大文字で現れる場合に限り、
[RFC 2119] および [RFC 8174] の定義に従って解釈されるものとする。

[RFC 2119]: https://www.rfc-editor.org/info/rfc2119
[RFC 8174]: https://www.rfc-editor.org/info/rfc8174

### 1.2 アクター

- **ホストツール** — AI モデルに代わってファイルを読み取るエージェントフレームワークまたは IDE 拡張（例: Claude Code、Cursor、カスタム CLI エージェント）。
- **リンター** — リポジトリのコンテキストファイルを ACE ルールに照らして検証し、検知結果を出力する実装。
- **アダプター** — ACE ネイティブでないファイル形式（例: `AGENTS.md`）を読み取り、ACE の内部表現として公開するコンポーネント。
- **作成者** — コンテキストファイルを書き・維持管理する人間（または PR を作成する AI エージェント）。

### 1.3 適合性レベル

実装は以下のいずれかのレベルで ACE 適合を主張してもよい（MAY）。
すべての主張にはレベルと ACE 仕様バージョンを明記しなければならない（例: "ACE v0.1 L1"）。

- **L1 — アクセス制御**: `.acp-ignore`（§3）を尊重する。ほとんどのホストツールに十分。
- **L2 — Lint**: 加えて、構造的 lint サーフェス（§5）を実装し、SARIF 2.1.0 出力（§5.4）を生成する。
- **L3 — フル**: 加えて、少なくとも1つのアダプター（§6）を実装し、由来証明（§7）を出力する。

### 1.4 スコープ外（v0.1）

- セマンティック lint を実行する LLM の指定
- エンタープライズ向けパッチ SLA の規定
- 認証済み適合性テストスイート
- 英語・日本語以外の lint メッセージのローカライズ
- アクセス制御のランタイムなエージェント間ネゴシエーション（Layer 6 — オーケストレーションを参照）

---

## 2. アーキテクチャ — 7 層参照モデル

ACE は AI エージェントのコンテキストスタックにまたがる層状参照モデルとして理解できる。
ACE は Layer 2 と Layer 5 を規範的に定義する。その他の層は隣接プロトコルが担い、
ここでは位置づけの参照としてのみ列挙する。

```
┌──────────────────────────────────────────────────────┐
│ Layer 6: オーケストレーション                          │
│   Spec Kit / Harmony / Autopilot フレームワーク        │
├──────────────────────────────────────────────────────┤
│ Layer 5: アクセス制御         ⭐ §3 で定義              │
│   .acp-ignore / ケイパビリティ宣言 / 監査由来証明       │
├──────────────────────────────────────────────────────┤
│ Layer 4: メモリ                                       │
│   → AMP（Agent Memory Protocol）                     │
├──────────────────────────────────────────────────────┤
│ Layer 3: ツール                                       │
│   → MCP（Model Context Protocol）                    │
├──────────────────────────────────────────────────────┤
│ Layer 2: コンテキストファイル  ⭐ §4 / §5 で定義        │
│   frontmatter / lint / アダプター                     │
├──────────────────────────────────────────────────────┤
│ Layer 1: メディアセマンティクス                        │
│   → M2C（Media-to-Context）                          │
├──────────────────────────────────────────────────────┤
│ Layer 0: 生データ                                     │
│   Pool / ファイルシステム / DB                         │
└──────────────────────────────────────────────────────┘
```

- **Layer 2（コンテキストファイル）** は、すべての ACE コンテキストファイルに共通する frontmatter スキーマ・アクティベーションセマンティクス・ステータスライフサイクルを定義する。
- **Layer 5（アクセス制御）** は、`.acp-ignore`・監査契約・バイパス規律を定義する。
- Layer 1、3、4、6 は本ドキュメントで参照するが定義しない。

---

## 3. Layer 5 — アクセス制御（`.acp-ignore`）

### 3.1 ファイル形式

- リポジトリはリポジトリルートに `.acp-ignore` ファイルをちょうど1つ置いてもよい（MAY）。
- ファイルは UTF-8 でエンコードされなければならない（MUST）。
- 行末は LF または CRLF でよい（MAY）。実装は両方を受け入れなければならない（MUST）。
- 最大ファイルサイズ: 128 KiB。これを超えるファイルを実装は拒否しなければならず、`violation.acp-ignore-too-large` 検知を出力しなければならない（MUST）。

### 3.2 パターン構文

パターンは以下の明確化を加えた `.gitignore` glob セマンティクスに従う:

- `*` は `/` を含まない任意の文字列にマッチする
- `**` は `/` を含む任意の文字列にマッチする
- 先頭の `/` はリポルートへのアンカーとなる
- 末尾の `/` はマッチをディレクトリに限定する
- `#` は行コメントを開始する。インラインコメントはサポートされない
- 空白のみの行は無視される

以下のディレクティブプレフィックスを認識する（コメント外の1行に1つ）:

| プレフィックス | 意味 |
|---|---|
| （プレフィックスなし） | 暗黙の `deny:`（レガシー `.gitignore` 互換） |
| `deny: <パターン>` | パターンに一致するパスへの読み取りアクセスを拒否する |
| `allow: <パターン>` | パターンに対する事前の拒否（デフォルト拒否を含む）をオーバーライドする |
| `bypass: <パターン>` | 期限付きの拒否オーバーライド。`approver` + `expires` を含まなければならない（MUST） |

`bypass:` ブロックは、`bypass: <パターン>` 行から始まり、（a）空行（空白除去後ゼロ文字）または（b）新しいトップレベルディレクティブ（`deny:`、`allow:`、`bypass:`、もしくはインデントなしのベアパターン）が最初に現れた行で終了する。継続行は少なくともスペース1つまたはタブでインデントされなければならない（MUST）。実装は `approver:` と `expires:` の両方を含まない `bypass:` ブロックを拒否しなければならない（MUST）。

整形された `bypass:` ブロックの例:

```
bypass: debug/trace.log
  approver: @Akari-OS/maintainers
  expires: 2026-10-01
  reason: インシデント #42 の一時的なトレースアクセス

deny: internal/**
```

`bypass:` ブロック内で有効なフィールドは次のとおりである:

- `approver: <識別子>` — **必須（REQUIRED）**。識別子ハンドル（`@github-user`、メールアドレス、または不透明文字列）。実装はこれを監査ログに記録しなければならない（MUST）。
- `expires: <YYYY-MM-DD>` — **必須（REQUIRED）**。ISO 8601 日付。この日付を過ぎたバイパスは尊重してはならず（MUST NOT）、実装は `violation.acp-ignore-expired-bypass` 検知を出力しなければならない（MUST）。
- `reason: <自由テキスト>` — **任意（OPTIONAL）** だが推奨（RECOMMENDED）。1行。

### 3.3 デフォルト拒否リスト

準拠したホストツールは、`.acp-ignore` が存在しない場合でも以下のパターンを暗黙的に拒否として扱わなければならない（MUST）:

```
.env
.env.*
*.key
*.pem
*.p12
*.pfx
id_rsa*
id_ed25519*
*.gpg
.aws/credentials
.aws/config
.ssh/**
.netrc
.npmrc
.pypirc
secrets/**
credentials/**
private/**
```

このリストは v0.1 において固定されており、後続のマイナーバージョンでは拡張のみ許可される（削減は不可）。

### 3.4 allow オーバーライド

明示的な `allow: <パターン>` は、一致するパスに対してデフォルト拒否リストおよびリポ固有の `deny:` の両方をオーバーライドしなければならない（MUST）。
`allow:` はファイル内の順序によらず、すべての `deny:` ディレクティブより後に評価される。

根拠: デフォルト拒否は秘密情報を保護するが、隣接ファイルの安全性を損なわずに
真に公開のフィクスチャ（例: `.env.example`、`docs/public-keys/*.pem`）を除外できなければならない。

### 3.5 バイパスルール

バイパスは一時的なものである。永続的な `allow:` エントリを必要とせずに
インシデント対応や一時的なデバッグを可能にするために存在する。

- `bypass:` エントリの `expires` 日付は今日から **180日** 以内でなければならない（MUST）。実装はこれを超えるエントリを拒否しなければならない（MUST）。
- バイパスが尊重される場合、実装は `approver`・`expires`・`path`・`timestamp` を含む `info.acp-ignore-bypass-used` 監査イベントを出力しなければならない（MUST）。
- `expires` 日付を過ぎたバイパスは `deny:` として扱われなければならない（MUST）。古いバイパスエントリは削除すべきである（SHOULD）が、期限切れエントリはアクセスを黙認してはならない（MUST NOT）。

### 3.6 監査要件

ACE L1 実装は以下を行わなければならない（MUST）:

- 拒否されたパスへのすべての読み取り試行を、ファイル内容がツールのプロセス境界を越える**前に**ブロックする
- すべての拒否ヒットに対して `audit.log` エントリを出力する。エントリ形式:

  ```json
  {
    "timestamp": "2026-04-14T09:30:00Z",
    "severity": "violation",
    "rule": "acp-ignore-deny",
    "path": ".env.production",
    "tool": "host-tool-name/1.2.3",
    "actor": "不透明なエージェント識別子",
    "matched_pattern": ".env.*"
  }
  ```

- 監査ログをユーザーの要求に応じて公開する。形式は SARIF 2.1.0（§5.4）であるべきである（SHOULD）。

実装は、事後的にフィルタリングする前提でパスを黙って読み取っていながら L1 適合を主張してはならない（MUST NOT）。読み取り前ブロックが必須である。

### 3.7 `.gitignore` との関係

`.gitignore` はバージョン管理を管理する。`.acp-ignore` は AI エージェントの読み取りアクセスを管理する。
あるパスは一方にのみ現れることがある。両者は独立している。

準拠したツールは `.gitignore` が `.acp-ignore` を暗示すると仮定してはならず（MUST NOT）、その逆も同様である。
`.gitignore` の `node_modules/` エントリは AI エージェントが内部を読み取ることを防がない — それを防ぐのは一致する `.acp-ignore` エントリのみである。

---

## 4. Layer 2 — コンテキストファイル仕様

### 4.1 frontmatter スキーマ

ACE コンテキストファイル（例: `CLAUDE.md`、`AGENTS.md`、`.cursor/rules/*.mdc`、
`.github/instructions/*.instructions.md`）は YAML frontmatter ブロックで始めてもよい（MAY）。
存在する場合、frontmatter は有効な YAML 1.2 でなければならず、`---` フェンスで囲われていなければならない（MUST）。

フィールド一覧:

| フィールド | 要件 | 型 | 説明 |
|---|---|---|---|
| `spec-version` | REQUIRED | string | ACE 仕様バージョン（例: `"0.1"`） |
| `applies-to` | REQUIRED | string \| string[] | このファイルをスコープする glob パターン |
| `activation` | REQUIRED | enum | `always` / `auto-attached` / `agent-requested` / `manual` のいずれか（§4.2 参照） |
| `title` | RECOMMENDED | string | 短い人間可読なタイトル |
| `description` | RECOMMENDED | string | 最大 1024 文字。エージェント UI に表示される |
| `status` | RECOMMENDED | enum | `active` / `draft` / `deprecated`（§4.3 参照） |
| `updated` | RECOMMENDED | string | 最後の意味のある編集の ISO 8601 日付 |
| `deprecated-after` | OPTIONAL | string | ISO 8601 日付。この日付以降はファイルが陳腐化する |
| `language.comments` | OPTIONAL | string | BCP-47 タグ（例: `ja` / `en`） |
| `language.commits` | OPTIONAL | string | 同上。エージェントが作成したコミット向け |
| `output_language` | OPTIONAL | string | エージェント応答言語の優先設定 |
| `tone` | OPTIONAL | string | 自由テキスト（例: "terse"、"関西弁"） |
| `spec-id` | OPTIONAL | string | Spec-Kit / RFC 識別子へのリンク |
| `excludeAgent` | OPTIONAL | string[] | ベンダー拡張（Copilot）。除外するエージェントロール |
| `paths` | OPTIONAL | string[] | ベンダー拡張（Claude Skills） |

不明なフィールドはハードエラーを引き起こしてはならない（MUST NOT）。
実装はタイポを検出するために認識されないフィールドについて警告すべきである（SHOULD）。

### 4.2 アクティベーションモード

Cursor v2.2 および Copilot 2026-04 GA セマンティクスを包含するように選択された4つのモード:

| モード | ファイルのコンテンツがエージェントコンテキストにアタッチされるタイミング |
|---|---|
| `always` | 閲覧中のファイルに関わらず、すべてのインタラクション |
| `auto-attached` | `applies-to` に一致するファイルが開かれるまたは編集されるとき自動的に |
| `agent-requested` | エージェントがファイルを関連と判断したとき（`description` による）のみ |
| `manual` | ユーザーがファイルを明示的に参照したときのみ |

実装は少なくとも `always` と `auto-attached` を実装しなければならない（MUST）。
`agent-requested` と `manual` は推奨（RECOMMENDED）。

### 4.3 ステータスライフサイクル

```
draft → active → deprecated
```

- `draft`: 作業中。リンターによってデフォルトアクティベーションから除外されてもよい（MAY）
- `active`: エージェントが消費できる状態
- `deprecated`: 置き換えられたか削除予定。ホストツールはアクティベーション時に警告すべきである（SHOULD）

リンターは `status: deprecated` のファイルがセッションでアクティベートされた場合に
`warn.deprecated-context-active` 検知を出力すべきである（SHOULD）。

### 4.4 陳腐化検出

コンテキストファイルが **陳腐化** しているとは、以下のいずれかを満たす場合である:

- `status: deprecated`
- `deprecated-after` が過去の日付
- `updated` が 180 日以上前 かつ `activation: always` を使用している

陳腐化したファイルは `warn.stale-context` として報告すべきである（SHOULD）。
根拠: MITRE / NAACL Findings 2024 は、誤ったドキュメントが欠落したドキュメントより
LLM のパフォーマンスを低下させることを示した。陳腐化検出は装飾的なものではなく、安全上重要である。

---

## 5. Lint ルール

### 5.1 構造的 lint（L2 で REQUIRED）

以下のルールを実装しなければならない（MUST）:

| ルール ID | 深刻度 | 説明 |
|---|---|---|
| `ace-fm-required-field` | error | 必須 frontmatter フィールドが欠落している |
| `ace-fm-invalid-enum` | error | `activation` / `status` が許可された列挙値外 |
| `ace-fm-invalid-date` | error | `updated` / `deprecated-after` が ISO 8601 でない |
| `ace-fm-applies-to-empty` | error | `applies-to` がどのファイルにも解決されない |
| `ace-acp-ignore-malformed` | error | `.acp-ignore` の構文エラー |
| `ace-acp-ignore-bypass-no-approver` | error | `bypass:` に `approver` がない |
| `ace-acp-ignore-bypass-no-expires` | error | `bypass:` に `expires` がない |
| `ace-acp-ignore-bypass-too-long` | error | `bypass:` の `expires` が 180 日超 |
| `ace-acp-ignore-expired-bypass` | warn | `bypass:` の `expires` 日付が過去 |
| `ace-stale-context` | warn | §4.4 に従いファイルが陳腐化している |
| `ace-deprecated-active` | warn | `status: deprecated` だがアクティベートされている |
| `ace-unicode-tag-found` | error | 不可視 Unicode タグ文字を検出（§8.2） |
| `ace-bidi-override-found` | error | BiDi オーバーライド文字を検出（§8.2） |
| `ace-zwsp-in-instruction` | warn | 指示行内のゼロ幅スペース |
| `ace-unknown-frontmatter-field` | info | 認識されない（タイポの可能性がある）フィールド |
| `ace-linkrot` | warn | 欠落しているファイルへの内部リンク |

### 5.2 セマンティック lint（OPTIONAL、ドラフト）

セマンティック lint は LLM を使用して以下を検出する:

- 複数のコンテキストファイル間の矛盾
- 曖昧な指示（「時々」「場合によっては」「依存する…」）
- 用語のドリフト（同じ概念が一貫して名付けられていない）
- 存在しないパッケージへの依存

セマンティック lint は v0.1 では OPTIONAL である。実装する場合:

- キーストローク毎に実行してはならない（MUST NOT）（コスト）
- PR 時に実行すべきである（SHOULD）
- 各検知に信頼度スコア（0.0〜1.0）を割り当てなければならない（MUST）
- ユーザーが設定可能な信頼度閾値以下の検知を抑制できるようにしなければならない（MUST）

詳細なルールセットと信頼度キャリブレーションは v0.1 のスコープ外であり、v0.2 で安定化する。

### 5.3 深刻度モデル

| 深刻度 | セマンティクス |
|---|---|
| `error` | 正確性またはセキュリティ違反。デフォルトで CI をブロックする |
| `warn` | 品質上の問題。PR レビューに表示されるが、デフォルトではブロックしない |
| `info` | 観察。冗長出力にのみ表示される |

実装はルールごとの深刻度オーバーライドを設定で許可しなければならない（MUST）。

### 5.4 出力形式（SARIF 2.1.0）

リンター出力は [SARIF 2.1.0] として表現できなければならない（MUST）。
最小限必要な SARIF フィールド:

- `runs[].tool.driver.name` = ACE 実装名
- `runs[].tool.driver.semanticVersion` = 実装バージョン
- `runs[].tool.driver.informationUri` = 実装 URL
- `runs[].results[].ruleId` = ACE ルール ID（例: `ace-fm-required-field`）
- `runs[].results[].level` = `error` / `warning` / `note`
- `runs[].results[].message.text`
- `runs[].results[].locations[].physicalLocation.artifactLocation.uri`
- `runs[].results[].locations[].physicalLocation.region.startLine`

実装は人間向けフォーマット（pretty / JSON / GitHub Actions アノテーション）を追加で出力してもよい（MAY）。
SARIF が規範的な交換フォーマットである。

[SARIF 2.1.0]: https://docs.oasis-open.org/sarif/sarif/v2.1.0/

---

## 6. アダプター層

アダプターは ACE ネイティブでないコンテキストファイルを読み取り、ACE の内部表現として公開する。
これにより、リポジトリは既存ファイルを書き換えることなく ACE バリデーションを採用できる。

### 6.1 `AGENTS.md`

- リポルートの `AGENTS.md` は、frontmatter ブロックでオーバーライドされない限り、
  `activation: always`、`applies-to: ["**/*"]`、`spec-version: "0.1"` として読み取られなければならない（MUST）。
- ネストした `AGENTS.md`（例: `packages/foo/AGENTS.md`）はそのディレクトリサブツリーにスコープされた `applies-to` を継承する。

### 6.2 `CLAUDE.md`

- Claude Code の `CLAUDE.md` は、frontmatter でオーバーライドされない限り、
  `activation: always`、`applies-to: ["**/*"]` として読み取られなければならない（MUST）。
- より深くネストした `CLAUDE.md` はそのサブツリーにスコープされる（`AGENTS.md` セマンティクスのミラー）。

### 6.3 Cursor `.cursor/rules/*.mdc`

- 3つの Cursor frontmatter フィールドは以下のように ACE にマッピングされる:

| Cursor フィールド | ACE フィールド |
|---|---|
| `description` | `description` |
| `globs` | `applies-to` |
| `alwaysApply: true` | `activation: always` |
| `alwaysApply: false` + glob | `activation: auto-attached` |
| （description のみ、globs なし、`alwaysApply` なし） | `activation: agent-requested` |

### 6.4 Copilot `.github/instructions/*.instructions.md`

- `applyTo` glob は `applies-to` にマッピングされる。
- Personal > Repo > Org のマージ順序は保持される。ACE は各スコープを、
  出力時に明示的な優先度メタデータを持つ別個のコンテキストファイルとして扱う。

### 6.5 Windsurf ルール（`.windsurfrules`）

- アクティベーションモード（`always_on` / `manual` / `model_decision` / `glob`）は
  ACE モード（`always` / `manual` / `agent-requested` / `auto-attached`）に 1:1 でマッピングされる。
- ファイルあたり 6,000 文字・アクティブバジェット 12,000 文字の制限は ACE では参考情報。
  lint はこれらを警告として表示してもよい（MAY）が、超過したファイルを拒否してはならない（MUST NOT）。

### 6.6 アダプター適合性

アダプターは ACE 準拠であるとは、ソース形式で有効な任意のソースファイルが与えられたとき、
アダプターが ACE リンターが余分なエラーなしに検証できる ACE 内部表現を出力することである。

---

## 7. 監査と由来証明

### 7.1 SARIF 統合

L2 実装は SARIF 2.1.0 を出力しなければならない（MUST）。これにより以下への直接取り込みが可能になる:

- GitHub Code Scanning（Advanced Security）
- GitLab SAST レポート統合
- Azure DevOps Advanced Security
- 任意の汎用 SARIF ビューアー

### 7.2 Sigstore 署名

実装は [Sigstore] (`cosign sign-blob`) を使用して `.acp-ignore` と
信頼のソースとして意図されたコンテキストファイルに署名すべきである（SHOULD）。
署名の検証は v0.1 では OPTIONAL であり、エンタープライズ展開では RECOMMENDED である。

署名されたファイルは `.sig` シブリングと透明性ログエントリを生成する。
リンターはエンタープライズモードで動作している場合に `warn.unsigned-context` 検知を表示してもよい（MAY）。

[Sigstore]: https://www.sigstore.dev/

### 7.3 SBOM 生成

実装はすべてのコンテキストファイルとそのハッシュをリストする [SPDX] または [CycloneDX] の
SBOM を生成してもよい（MAY）。これは v0.1 では OPTIONAL であり、
EU CRA 2027 要件との整合のために v0.3 では RECOMMENDED となる。

[SPDX]: https://spdx.dev/
[CycloneDX]: https://cyclonedx.org/

---

## 8. セキュリティ上の考慮事項

### 8.1 Rules File Backdoor 軽減策

研究（Pillar Security 2025、arXiv:2601.17548）は、コンテキストファイルが
AI エージェントへの隠れた指示注入に悪用され得ることを実証している（適応的攻撃の成功率は 85% 超）。
一般的なベクター:

- 不可視 Unicode（ゼロ幅スペース U+200B、Unicode タグ文字 U+E0000〜U+E007F、BiDi オーバーライド U+202E）
- ホモグリフ置換
- フェンスされたコードブロック内のコメントに偽装した指示

ACE の軽減策（v0.1 で規範的）:

- `ace-unicode-tag-found`（error）: `lang` タグブロック外の U+E0000〜U+E007F 範囲の文字を含むファイルを拒否する
- `ace-bidi-override-found`（error）: U+202E、U+202D、U+2066、U+2067、U+2068 を含むファイルを拒否する
- `ace-zwsp-in-instruction`（warn）: 指示行内（コードブロック外の検出）の U+200B / U+200C / U+200D / U+FEFF をフラグする

リンターは検知が表示されたときにこれらの文字をユーザーに可視化する `--render-invisible` / 相当する可視化モードを提供しなければならない（MUST）。

### 8.2 サプライチェーンの安全性

- サードパーティソース（例: 共有ルールパック）からインポートされたコンテキストファイルはハッシュでピン留めすべきである（SHOULD）。
- リンターはエンタープライズモードで動作している場合、署名検証に失敗したコンテキストファイルの自動適用を拒否すべきである（SHOULD）。

### 8.3 エージェントが作成したコンテキスト

AI エージェントがコンテキストファイルを変更する場合（例: Claude Code セッションが `CLAUDE.md` を編集）、
その変更は帰属可能でなければならない（MUST）。実装はすべきである（SHOULD）:

- コミットに `Signed-off-by:` を要求する（DCO に従う）
- コミットメタデータにエージェント識別子（`tool-name/version` + モデル ID）を記録する
- エージェントが作成した変更を PR レビューで明示的に表示する

### 8.4 `.acp-ignore` 自身

`.acp-ignore` は信頼のアンカーである。実装は以下を行わなければならない（MUST）:

- 任意のディレクティブを尊重する前に構文を検証する
- パース失敗時にはファイル全体を拒否し、部分的に尊重しない
- 明示的な人間の承認なしに `.acp-ignore` を自動編集することを拒否する

---

## 9. バージョニングポリシー

ACE は仕様レベルで [Semantic Versioning 2.0.0] に従う。

- **MAJOR**（1.0 → 2.0）: frontmatter スキーマ、`.acp-ignore` 構文、または適合性レベルへの後方互換性のない変更
- **MINOR**（0.1 → 0.2）: 後方互換性のある追加（新しい lint ルール、新しいアダプター、新しい任意 frontmatter フィールド）
- **PATCH**（0.1.0 → 0.1.1）: 編集上の明確化のみ

ドラフトバージョン（例: `0.1.0-draft`）は安定版ステータスに達するまで非互換変更を行ってもよい（MAY）。
一度安定版と宣言されたマイナーバージョンは不変である。

**`draft` から安定版への昇格** には以下が必要:

- 自己レビューと少なくとも1人の外部レビュアー
- 安定版 `draft → implemented` のために: 相互運用を実証する2つの独立した実装（IETF 慣例）

[Semantic Versioning 2.0.0]: https://semver.org/spec/v2.0.0.html

---

## 10. 変更履歴

### v0.1.1（2026-04-22）

編集上のパッチ。5件の Open issues のうち3件を解消し、2件を v0.2 に先送り（付録 B 参照）。

### v0.1.0-draft（2026-04-14）

初回公開ドラフト。以前のバージョンなし。

---

## 11. 参考文献

### 規範的

- [RFC 2119] — RFC においてリクエスト要件レベルを示すためのキーワード
- [RFC 8174] — RFC 2119 キーワードの大文字・小文字の曖昧さ
- [SARIF 2.1.0] — OASIS 静的解析結果交換フォーマット
- [Semantic Versioning 2.0.0]

### 参考情報

- [AGENTS.md](https://agents.md) — AAIF クロスベンダーのルートファイル仕様
- [MCP](https://modelcontextprotocol.io) — Model Context Protocol
- [AMP](https://github.com/Akari-OS/amp) — Agent Memory Protocol
- [M2C](https://github.com/Akari-OS/m2c) — Media to Context
- [Cursor Rules](https://docs.cursor.com/context/rules-for-ai)
- [Claude Code Skills](https://github.com/anthropics/claude-skills)
- [Copilot Instructions](https://docs.github.com/copilot/customizing-copilot/about-customizing-github-copilot-chat-responses)
- [Windsurf Rules](https://docs.windsurf.com/windsurf/cascade/memories)
- [Cline Memory Bank](https://docs.cline.bot/prompting/custom-instructions)
- [Vale](https://vale.sh/) — エンジンレスのリンターアーキテクチャ
- [Ruff](https://docs.astral.sh/ruff/) — Rust ベースのリンターのパフォーマンス参考
- [Sigstore](https://www.sigstore.dev/)
- [robots.txt](https://www.robotstxt.org/)
- [MITRE NAACL Findings 2024] — 「誤ったドキュメントは欠落したドキュメントより有害」
- [Pillar Security, Rules File Backdoor, 2025]
- [arXiv:2601.17548] — エージェントコンテキストファイルへの適応的攻撃

---

## 12. 付録 A — 最小適合例

以下のファイルを持つリポジトリは最小限の L1 準拠 ACE コンシューマーである:

```
my-repo/
├── .acp-ignore
├── AGENTS.md
└── docs/
    └── README.md
```

`.acp-ignore` の内容:

```gitignore
# デフォルト拒否リストはリンターが提供する。リポ固有の
# エントリのみここに記載する。
deny: internal/**
allow: .env.example
```

`AGENTS.md` の冒頭:

```yaml
---
spec-version: "0.1"
applies-to: "**/*"
activation: always
title: リポ規約
status: active
updated: 2026-04-14
---
```

## 13. 付録 B — v0.2 以降の検討事項

v0.1.0-draft に列挙されていた全5件の課題は、v0.1.1 編集パッチにて解決または明示的に先送りされた。

### v0.1.1 で解決済み

| 課題 | 解決内容 |
|---|---|
| `bypass:` ブロック終了文法 | §3.2 にてインデント継続モデルをプロセス記述＋例示で規定 |
| 複数 `allow:` エントリ競合時の評価順序 | §3.4 にて「最も具体的パターン優先、同スコア時はファイル末尾優先」を規定 |
| 日本語翻訳 | `spec/v0.1/ja/protocol.md` を追加 |

### v0.2 へ先送り

- **エージェントごとのアクセス制御** — パス単位ではなくエージェント識別子（`@claude-code`、`@cursor`）を主キーとするアクセス制御は、ケイパビリティ言語を新設せずには表現困難なため v0.2 以降に先送りする。
- **セマンティック lint の信頼度スコア正規化** — 異なる LLM ベンダー間でスコアを比較可能にする正規化手法は、十分なベンチマークデータが揃うまで規定できない。v0.2 でセマンティック lint ルールセットとともに安定化する。

課題のトラッキング: <https://github.com/Akari-OS/ace/issues>
