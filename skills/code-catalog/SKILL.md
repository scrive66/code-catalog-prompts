---
name: code-catalog
description: 'Use this skill when the user wants to mine implicit coding conventions from an existing codebase, generate Copilot instructions from real code instead of writing them from scratch, or find code that deviates from the codebase majority pattern and may hide undocumented bug risk. Trigger for requests like "extract our coding conventions", "make instructions from this codebase", "find inconsistent code", "コードの作法を掘り出して", "instructionsを既存コードから作って", "傾向の違うコードを探して". Do not trigger for routine feature implementation or single-file bug fixes.'
license: MIT
metadata:
  version: "1.0"
argument-hint: 'Optional: target scope or a specific family, e.g. "src/ only", "HTTP呼び出しだけ"'
---

# Code Catalog — 多数派パターンの発掘と逸脱監査

既存コードから「実際に守られている多数派パターン」を証拠付きで掘り出し、
(1) instructions の材料にし、(2) 多数派から外れた箇所をバグ候補として監査するスキル。
白紙からルールを書かせない。「掘る → 選ぶ → 指示化する」が原則。

詳細な実行手順は、このリポジトリの `prompts/` にある4つのプロンプトファイルが正本。
このスキル単体で導入された場合は、以下の要約手順で実行する。

## 絶対ルール（全工程共通）

1. **読み取り専用**。調査対象コードは変更しない。書き出すのは成果物ファイルだけ。
2. **全項目に証拠**。パターンには「該当数 / 母数」と代表例の `path:line` を2件以上付ける。
   数えられない主張は書かない。推測でルールを捏造しない。
3. **採用率で格付け**。80%以上=「確立した作法」、60〜79%=「傾向あり」、
   60%未満や流派が拮抗するものはルール化せず「分裂項目」に回す。
4. **意図的例外を犯罪扱いしない**。コメント・命名・設定・テストで意図が読める逸脱は
   「登録済み例外」候補として理由つきで記録する。
5. 母数3未満は格付けせず「サンプル不足」とする。不明なことは「不明」と書く。

## ワークフロー

### 工程1: カタログ生成（prompts/generate-code-catalog.prompt.md）
- 棚卸し: 言語・構成・既存の明文ルール（README、linter/CI設定、既存instructions）を先に分離収集
- 処理ファミリーの発見: 同種処理の集まり（HTTP呼び出し、ファイルIO、例外処理…）に分け母数を数える
- ファミリーごとに「チェック項目 × 実装」のマトリクスを作り、項目ごとの実施率を算出
- 横断作法（命名、分割、テスト配置等）も同様に数える
- 成果物: `docs/code-catalog.md`（明文ルール／ファミリー別カタログ／分裂項目／例外候補／逸脱候補予備／不明）

### 工程2: 暗黙知インタビュー（任意）（prompts/interview-tacit-knowledge.prompt.md）
コードから読めないことだけを最大7問聞く: 触ってはいけない領域、意図的な仕様、
過去の障害、検証手順、ドメイン用語。選択肢つきで、開発者が選ぶだけで済む形にする。

### 工程3: instructions化（prompts/convert-catalog-to-instructions.prompt.md）
人間がカタログで `[採用]` 印を付けた項目だけを指示ファイルの下書きに変換する。
- 検証可能な命令形だけ。各行に根拠（実施率・代表例）をコメントで併記
- ファイル種別に閉じるルールは `.github/instructions/*.instructions.md` 案に分け、`applyTo` を付ける
- 模範ファイルへの参照（「○○の書き方に倣う」）を活用する
- 成果物は `docs/instructions-draft/` の下書きまで。実ファイルへの配置は人間が行う

### 工程4: 逸脱監査（prompts/audit-code-deviations.prompt.md）
カタログの「確立した作法」だけを物差しに全実装を突き合わせ、欠落箇所を候補化する。
- 候補ごとに反証チェック（呼び出し側の肩代わり、専用テスト、共通ラッパの担保、履歴上の意図）を通し、
  説明が付くものは「説明可能な逸脱」に分ける
- 優先度は「差分の内容 × コードの重要度（外部入力・書き込み・認証・金額・安全）」で付ける
- 成果物: `docs/deviation-audit.md`。自動修正はしない

## 還流ルール
監査で見つけた逸脱は、本物なら作法としてカタログに確定し、意図的なら理由を
「登録済み例外」に追記する。この往復で暗黙知が目録に移っていく。

## やらないこと
- 実測していない一般論ベストプラクティスの説教
- formatter が決める領域への口出し（「設定に委ねられている」と記録するだけ）
- 逸脱候補の自動修正・自動Issue化
