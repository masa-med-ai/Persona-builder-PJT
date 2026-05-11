# 02_Notion接続確認.md — Step 2

## ゴール

Notion MCP（`https://mcp.notion.com/sse`）が現在のセッションに接続されているかを確認します。**未接続でも Step 3 以降は進行可能** ですが、接続状況だけは利用者に明示します。

## 実行手順（Cowork 用）

### 1. 接続確認

Cowork は以下のいずれかで Notion MCP の有無を確認します。

- ツール一覧に `mcp__*__notion-search` または `mcp__*__notion-fetch` 等の Notion 系ツールが存在するか
- 利用者の MCP 設定で `https://mcp.notion.com/sse` が登録されているかを尋ねる

### 2. 接続済みの場合

利用者に以下を伝えて Step 3 へ進みます。

```
Notion MCP は接続済みです。
今回のフローでは Notion への保存は行いませんが、将来 persona.md を Notion ページ化したい場合に活用できます。
```

### 3. 未接続の場合

以下を伝えて **そのまま Step 3 へ進みます**（ブロックしません）。

```
Notion MCP は未接続です。今回のフローでは Notion への保存は行わないため、未接続のまま進めても問題ありません。
将来 persona.md を Notion ページ化したい場合は、Cowork の MCP 設定で `https://mcp.notion.com/sse` を追加してください。
```

### 4. PROGRESS.md 追記

Step 2 完了時に `PROGRESS.md` の該当箇所へ以下を追記します。

- Notion MCP 接続状況：接続済み / 未接続
- 確認日時

## 非ゴール

- Notion ワークスペース内の検索やページ作成は **今回のフローでは行いません**。接続有無の確認のみです。

---

次は Step 3（About-Me 簡単Q&A）に進みます。
