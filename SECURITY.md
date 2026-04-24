# Security Policy — ACE (Agent Context Engineering)

ACE コミュニティはセキュリティを真剣に受け止めている。仕様書の誤記、脆弱なセキュリティガイダンス、
または examples / reference-impl 配下のセキュリティ懸念を発見したら、
責任ある方法で報告してほしい。

---

## 脆弱性の報告方法

### ⚠️ 公開 Issue を立てないこと

公開 Issue に脆弱性情報を書くと、修正される前に悪用される可能性がある。
必ず**プライベートな方法**で報告してほしい。

### 報告チャネル

**優先**: 該当リポジトリの Security タブ →「Report a vulnerability」を使う。
GitHub 上で暗号化された報告が送れて、追跡もしやすい。
（例: `https://github.com/Akari-OS/ace/security/advisories/new`）

**代替**: メールで報告 → security@akari-oss.app

---

## 報告に含めてほしい情報

- 影響を受ける仕様セクション / バージョン
- セキュリティ懸念の種類（例: `.acp-ignore` 回避リスク、frontmatter 検証漏れ、
  prompt injection リスク、rules file backdoor パターン等）
- 具体的な問題シナリオ（最小限の PoC があると助かる）
- 想定される影響範囲（spec の採用者 / reference impl ユーザー / エンドユーザー等）
- 可能であれば修正案（仕様改訂案）

---

## 対応プロセス

1. **受領確認** — 72 時間以内に受領を確認する
2. **初期評価** — 1 週間以内に影響の初期評価を返す
3. **修正開発** — 重大度に応じて仕様改訂 / reference impl fix を開発
4. **リリース** — 修正版をリリース（spec バージョン bump + reference impl patch）
5. **公開情報開示** — 修正リリース後、Security Advisory / Release Notes として公開

---

## 重大度の目安

| レベル | 例 | 対応速度 |
|---|---|---|
| **Critical** | `.acp-ignore` が回避可能、prompt injection の確実な exploit パターン、認可バイパス | 即日〜数日 |
| **High** | frontmatter 検証の不備、rules file backdoor の検出漏れ、ガイダンス誤記による実装バグ | 1〜2 週間 |
| **Medium** | 仕様の曖昧性、examples 配下の不十分な安全警告 | 2〜4 週間 |
| **Low** | タイポ、曖昧な用語の明確化、ベストプラクティス追加 | 次回リリース時 |

---

## スコープ

本ポリシーは `github.com/Akari-OS/ace` に適用される。

### スコープ内

- `spec/` — ACE 公式仕様（`.md` form）
- `examples/` — reference examples と anti-patterns
- `reference-impl/` — reference implementation（あれば）
- ドキュメント全体の誤記・矛盾

### スコープ外

- サードパーティの ACE adapter / 実装
- fork / 古いブランチ
- 未公開の開発版

---

## 謝辞

責任ある開示をしてくれた研究者には、修正リリース時のリリースノートにお名前を記載する
（希望しない場合は匿名）。

**ありがとう**。あなたの報告が ACE をより安全で信頼できる仕様にする。
