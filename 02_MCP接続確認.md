# 02_MCP接続確認.md — Step 2

## ゴール

ペルソナ作成中および作成後の文章支援に必要な MCP（Model Context Protocol サーバ）の接続状況を確認します。**未接続でも Step 3 以降は進行可能** ですが、PubMed MCP は文章作成・査読支援で重要なので、未接続時はインストール手順を案内します。

## 確認対象の MCP

| MCP | 用途 | 必須/任意 | 未接続時の対応 |
|---|---|---|---|
| **Notion MCP** | persona.md の Notion ページ化（任意・将来用） | 任意 | 案内のみ |
| **PubMed MCP** | 文献検索・引用検証・査読返信時のエビデンス確認 | 推奨 | DL/インストール案内 |

ペルソナ作成中（Step 3〜5）は **どちらの MCP も使用しません**。ペルソナ確定後の文章作成・査読・引用検証フェーズで PubMed MCP が活躍します。

## 実行手順（Cowork 用）

### 1. PubMed MCP の接続確認（優先）

Cowork は以下のいずれかで PubMed MCP の有無を確認します。

- ツール一覧に `mcp__*__search`（PubMed E-utility 連携）、`mcp__*__fetch`、`mcp__*__count`、`mcp__*__convert_ids` などの PubMed 系ツールが存在するか
- 利用者の MCP 設定で PubMed 関連のサーバが登録されているかを尋ねる

#### 1-A. 接続済みの場合

```
PubMed MCP は接続済みです。
ペルソナ確定後、引用文献の検索・PMID 検証・査読返信時のエビデンス確認等にお使いいただけます。
（検索時は MeSH で式を立案し、count で件数妥当性を確認してから実行する運用です。）
```

#### 1-B. 未接続の場合

以下を伝えてインストール案内を出します。**インストールは Cowork からは自動実行できません**（Claude Desktop / Cowork の MCP 設定画面でユーザー操作が必要）。

```
PubMed MCP は未接続です。文章作成・査読支援で頻繁に使うため、インストールを推奨します。

【インストール手順】
1. 以下から pubmed-mcp-bundle.mcpb をダウンロード
   https://github.com/masa-med-ai/pubmed-mcp-bundle
   （直接DL: https://github.com/masa-med-ai/pubmed-mcp-bundle/raw/main/pubmed-mcp-bundle.mcpb）
2. ダウンロードした .mcpb ファイルをダブルクリック、または
   Claude Desktop / Cowork の MCP 設定画面から「バンドルをインストール」で読み込み
3. インストール後、Claude Desktop / Cowork を再起動して接続を有効化
4. このセッションに戻ったら「Step 2 やり直し」と打ってください（接続確認のみ再実行します）

ペルソナ作成自体は PubMed なしでも完了できますので、後回しでも構いません。
その場合「Step 3 へ進む」と打って先に進めてください。
```

### 2. Notion MCP の接続確認

ツール一覧に `mcp__*__notion-search` または `mcp__*__notion-fetch` 等の Notion 系ツールが存在するかを確認します。

#### 2-A. 接続済みの場合

```
Notion MCP は接続済みです。
今回のフローでは Notion への保存は行いませんが、将来 persona.md を Notion ページ化したい場合に活用できます。
```

#### 2-B. 未接続の場合

```
Notion MCP は未接続です。今回のフローでは Notion への保存は行わないため、未接続のまま進めても問題ありません。
将来 persona.md を Notion ページ化したい場合は、Cowork の MCP 設定で `https://mcp.notion.com/sse` を追加してください。
```

### 3. ユーザーの選択肢提示

PubMed MCP の状態に応じて、利用者に以下のいずれかを促します。

- **両方接続済み** → 「Step 3 へ進みます」と利用者に案内
- **PubMed のみ未接続** → 「インストールしてから進める／このまま Step 3 へ進む」を選択させる
- **両方未接続** → Notion はそのまま、PubMed はインストール案内を出し、選択させる

利用者が「このまま進む」と選んだ場合はブロックせずに Step 3 へ進みます。

### 4. PROGRESS.md 追記

Step 2 完了時に以下を追記します。

- Notion MCP 接続状況：接続済み / 未接続
- PubMed MCP 接続状況：接続済み / 未接続 / インストール予定 / 未接続のまま進行
- 確認日時

## 非ゴール

- **Cowork による MCP の自動インストール**：.mcpb のインストールはユーザー操作が必要なため、Cowork は案内のみ。
- **業績調査（過去論文リスト抽出）**：PubMed MCP が接続済みでも、ペルソナ作成中（Step 3〜5）は本人の業績を機械的に列挙しません。PubMed は **ペルソナ確定後の文章支援** で使用するためのものです。
- **Notion ワークスペース検索・ページ作成**：今回のフローでは行いません。

---

次は Step 3（About-Me 簡単Q&A）に進みます。
