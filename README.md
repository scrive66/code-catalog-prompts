# Code Catalog Prompts

既存コードから「多数派の作法」を掘り出してカタログ化し、(1) Copilot instructions の材料にし、
(2) 多数派から外れたバグ候補の監査に使うための、VS Code / GitHub Copilot 向けプロンプト＆スキル集。

白紙からルールを書かない。「掘る → 選ぶ → 指示化する」ための道具です。

> レイアウトは [github/awesome-copilot](https://github.com/github/awesome-copilot) 形式に準拠しています
> （`prompts/` = プロンプトファイル、`skills/` = Agent Skills）。互換ツールからそのまま導入できます。

## Prompts

対象プロジェクトの `.github/prompts/` に置くと、Copilot Chat で `/<名前>` として呼べます。

| プロンプト | 説明 |
|---|---|
| [generate-code-catalog](prompts/generate-code-catalog.prompt.md) | 既存コードを調査し、証拠（実施率・path:line）付きの多数派カタログ `docs/code-catalog.md` を生成する（読み取り専用） |
| [audit-code-deviations](prompts/audit-code-deviations.prompt.md) | カタログの「確立した作法」と実コードを突き合わせ、チェックの不在＝バグ候補を監査し `docs/deviation-audit.md` を出す |
| [convert-catalog-to-instructions](prompts/convert-catalog-to-instructions.prompt.md) | カタログで `[採用]` 印を付けた項目だけを、根拠コメント付きの instructions 下書きに変換する |
| [interview-tacit-knowledge](prompts/interview-tacit-knowledge.prompt.md) | コードから読めない暗黙知だけを最大7問（選択肢つき）で聞き取り、ルール案と例外リストに整形する |
| [generate-reuse-index](prompts/generate-reuse-index.prompt.md) | 再利用可能な部品（関数・クラス・モジュール）を棚卸しし、シグネチャ・利用実例・再利用グレード付きの目録 `docs/reuse-index.md`（＋JSON版）を生成する。「書く前に見る」ための台帳で、instructions 用スニペットも出力する |

## Skills

| スキル | 説明 |
|---|---|
| [code-catalog](skills/code-catalog/SKILL.md) | 上記4工程（発掘→インタビュー→指示化→監査）を束ねる Agent Skill。要約手順を内蔵し、単体でも動作する |

## インストール

### A. スキルとして導入（推奨）

```bash
npx skills add scrive66/code-catalog-prompts
```

Agent Skills 対応ツール（GitHub Copilot、Claude Code、Codex、Cursor 等）に
`code-catalog` スキルが入ります。

### B. プロンプトだけ使う

使いたい `.prompt.md` を、対象プロジェクトの `.github/prompts/` にコピーしてください。
VS Code 拡張「Awesome Copilot」等の awesome-copilot 互換ツールから、
このリポジトリを指定して取り込むこともできます。

### C. 貼って使う

各ファイルの frontmatter（`---` で囲まれた部分）を取り除き、本文を Copilot Chat
（Agent モード）に貼っても動きます。対象範囲や出力先は本文の「入力」欄で変更できます。

## 回し方（推奨フロー）

1. **`/generate-code-catalog`** → `docs/code-catalog.md` ができる
2. **カタログを人間が検分**する。採用したい作法に `[採用]` と印を付ける。
   「分裂項目」は無理に統一せず、そのままにしておくのが正解
3. 必要に応じて **`/interview-tacit-knowledge`** で、コードから読めない例外・禁止事項を補う
   （出力される「登録済み例外」をカタログに追記しておくと、監査の誤検出が減る）
4. **`/convert-catalog-to-instructions`** → instructions の下書きができる。検分して `.github/` へ配置する
5. **`/audit-code-deviations`** → 逸脱監査レポートができる。優先度「高」から目視確認する。
   確認結果は次のどちらかに還流させる:
   - 本物のバグだった → 修正し、カタログの作法として確定
   - 意図的だった → 理由をカタログの「登録済み例外」に追記（次回監査で除外される）
6. コードベースが育ったら 1 を再実行してカタログを更新する（差分確認を推奨）

### 部品の再利用まで広げる場合

作法（カタログ）とは別に、部品そのものの再利用には **`/generate-reuse-index`** を使う。
生成された `docs/reuse-index.md` と、出力される instructions 用スニペットを
プロジェクトの instructions に貼ると、「新規実装の前に目録を見る」が常時ルールになる。
目録とカタログは独立に再生成してよい（目録=部品の棚、カタログ=作法の棚）。

## 設計上の約束（全プロンプト共通）

- **読み取り専用**。監査・生成系は調査対象コードを変更しない。指示化も下書きフォルダにしか書かない
- **証拠なしのルールを作らない**。全項目に実施率と代表例（path:line）を必須化し、
  採用率60%未満はルール化せず「分裂項目」に回す（80%以上が「確立した作法」）
- **意図的例外を守る**。逸脱はまず「説明可能か」を反証チェックしてから候補に残す
- **自動修正しない**。発見と報告までが仕事。直すかどうかは人間が決める

## 注意

- 母数が小さいプロジェクトでは実施率が暴れる。サンプル不足の項目は鵜呑みにしない
- 生成されたカタログは「今のコードの姿」の記録であり、正しさの証明ではない。
  多数派が間違っているプロジェクトでは、間違いもカタログ化される（だから検分工程がある）
- 大規模リポジトリでは対象範囲を絞るか、ファミリー単位で分割実行すること

## ライセンス

MIT
