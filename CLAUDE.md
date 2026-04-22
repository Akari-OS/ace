# CLAUDE.md — ACE リポジトリ

> Akari-OS/ace の開発ルール（AI / 人間共通）。
> AKARI エコシステムの公開プロトコルリポ。非公開戦略・GTM メモは絶対に置かない。

## このリポの役割

ACE (Agent Context Engineering Framework) の **公開正典**。

- **spec**（`spec/v0.X/protocol.md`）— CC-BY 4.0
- **reference 実装**（将来追加、Rust）— Apache 2.0
- **governance**（README / CONTRIBUTING / CoC / TRADEMARKS / CHANGELOG）

非公開の戦略・競合分析・GTM・価格検討は `~/_project/akari-os/docs/strategy/` に置く。このリポには置かない。

## 扱わない範囲

- **実装コード** — v0.1 draft 段階では spec のみ。reference 実装は別 subdir or 別リポで将来追加
- **戦略メモ** — Hub（akari-os）に
- **AKARI 全体のビジョン** — `Akari-OS/.github` に
- **他プロトコル**（MCP / AMP / M2C）の詳細 — それぞれの公式リポに pointer

## spec 変更プロセス

### v0.1 (draft) 期間

- 積極的に改善 PR を受ける
- ただし normative（MUST / MUST NOT）追加は Issue で RFC 議論 → 7-10 日 FCP

### v0.1 確定後

- 後方互換の軽微修正のみ
- 破壊的変更は v0.2+ delta spec

## コミット規約

AKARI 共通プレフィックス：

- `[仕様]` spec 変更
- `[ドキュメント]` README / CHANGELOG / ガイド
- `[バグ修正]` 誤字・リンク切れ
- `[リファクタ]` 構成変更
- `[セキュリティ]` security 関連

**DCO 必須**: すべての commit に `Signed-off-by:` 行（`git commit -s`）。

変数・関数名は英語、コメント・コミットメッセージは日本語 OK。

## Claude Code 運用メモ

- **ファイル命名は英語 kebab-case**、日本語ファイル名は禁止（AKARI ドキュメント規則）
- **Rules File Backdoor 対策のドッグフード**: この repo 自身の context file（本 CLAUDE.md 含む）は ACE lint に掛ける
- **spec 変更と実装変更は別 PR で**: v0.1 安定化まではとくに spec 単独 PR を優先
- **他 AKARI リポへの言及は pointer のみ**: 内容転記禁止

## 関連

- 公開ビジョン: [Akari-OS/.github](https://github.com/Akari-OS)
- 兄弟プロトコル:
  - [Akari-OS/amp](https://github.com/Akari-OS/amp) — Agent Memory Protocol
  - [Akari-OS/m2c](https://github.com/Akari-OS/m2c) — Media to Context
- Hub（非公開、関係者のみ）: `akari-os`
