# Contributing to ACE

ACE (Agent Context Engineering Framework) への貢献を歓迎します。

## プロジェクト概要

ACE は AI エージェントの **context 層** を標準化するオープンプロトコル・フレームワーク。

- **v0.1** — draft（`spec/v0.1/protocol.md`）
- `.acp-ignore` / frontmatter / lint / adapter / audit の 5 本柱を定義

## 貢献の種類

- spec の改善提案（Issue または PR）
- 実装の追加報告（`IMPLEMENTATIONS.md` への追記 PR）
- 誤字・文法・翻訳改善
- 日本語翻訳（`spec/v0.1/ja/protocol.md`、v0.1.1 で追加）
- `.acp-ignore` デフォルトリストへの追加提案（特に言語・フレームワーク別の機密パターン）

## ライセンス & DCO

- Spec（prose）: **CC-BY 4.0**
- Reference 実装（code）: **Apache 2.0**
- Examples: **CC0 / MIT**（per-file）

本プロジェクトは **DCO（Developer Certificate of Origin）** を採用する。CLA は求めない。すべての commit に `Signed-off-by:` 行を含めること。

```bash
git commit -s -m "[仕様] .acp-ignore の bypass セクションを明確化"
```

DCO の本文: <https://developercertificate.org/>

## spec 変更プロセス

### v0.1 (draft) を変更する場合

draft 期間は積極的に改善 PR を受け付ける。ただし以下は Issue で先に議論：

- 新しい normative requirement（MUST / MUST NOT）の追加
- `.acp-ignore` 構文の破壊的変更
- frontmatter スキーマの破壊的変更

### v0.1 確定後

- 後方互換を保つ軽微な修正のみ（誤字・曖昧さ解消・例の追加）
- 破壊的変更は v0.2+ へ、delta spec 形式で起草

## RFC / 大規模変更

大きな提案（新セクション / 新機能 / 互換性影響あり）は **Issue に RFC テンプレで起票** → **7–10 日の FCP（Final Comment Period）** → merge のフローに従う。PEP / Rust RFC ハイブリッドを参考にしている。

RFC に含めるべき項目：

1. Motivation（なぜ必要か）
2. Design（具体案）
3. Alternatives considered（代替案と却下理由）
4. Backward compatibility（互換性影響）
5. Security implications（セキュリティ影響）

## PR フロー

1. 大きな変更は先に Issue で議論（上記 RFC）
2. fork & branch
3. `Signed-off-by:` 付きで commit
4. PR 作成（テンプレに従う）
5. review（最低 1 maintainer）
6. merge（squash preferred）

## コミット規約

AKARI エコシステム共通のプレフィックス：

- `[仕様]` spec ファイル変更
- `[ドキュメント]` README / CHANGELOG / ガイド
- `[バグ修正]` 誤字・リンク切れ・事実誤認等
- `[リファクタ]` 構成変更
- `[セキュリティ]` セキュリティ関連の修正

変数・関数名は英語、コメント・コミットメッセージは日本語 OK。

## 昇格条件（`draft → accepted → implemented`）

IETF 方式に倣う：

- `draft` → `accepted`: 自己レビュー + 最低 1 外部レビュアー
- `accepted` → `implemented`: **2 つの独立実装で相互運用成功**

## 関連

- [Code of Conduct](./CODE_OF_CONDUCT.md)
- [CHANGELOG](./CHANGELOG.md)
- [Implementations](./IMPLEMENTATIONS.md)
- [Trademarks](./TRADEMARKS.md)
- [AKARI-OS Vision](https://github.com/Akari-OS)
