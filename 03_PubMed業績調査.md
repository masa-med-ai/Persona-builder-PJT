# 03_PubMed業績調査.md — Step 3: PubMed業績調査

このファイルは Step 3 で Cowork が参照します。Step 3 のゴールは、**対象者を First Author とする論文** を網羅的に取得し、主要論文・MeSH頻出語・共著者・主要雑誌を抽出して `outputs/publications.md` に保存することです。

PubMed検索は **First Author 検索 → count → search** の3段階を踏みます（CLAUDE.md ルール#2-A）。MeSH は検索式の絞り込みではなく、**検索後の集計（研究テーマ抽出）** に使います。

---

## 3-A. 検索式立案（First Author ベース）

### 基本形（最初に試す）

```
"Last F"[1au]
```

`[1au]` は PubMed の First Author 専用フィールドタグです。これで対象者が筆頭著者の論文のみが返ります。

#### 例：三澤将史先生の場合

```
"Misawa M"[1au]
```

### 表記揺れ対応

著者名のイニシャル表記が論文によって異なる場合があります（例: ミドルネーム入れる/入れない）。Step 2 のフォームで「著者名表記の揺れ」が記入されていれば OR 連結します。

```
("Misawa M"[1au] OR "Misawa MA"[1au])
```

### 同名著者問題が出た場合の補助

count 結果が想定より多い（例: 数百件）または明らかに別人の論文が混入している場合のみ、以下を併用します。

| 補助条件 | 検索式追加 |
|---|---|
| 所属で絞る | `AND "Showa University"[Affiliation]` |
| ORCID で絞る | `AND "0000-0002-XXXX-XXXX"[Author - ORCID]`（最も確実） |
| 出版年で絞る | `AND ("2010"[Date - Publication] : "3000"[Date - Publication])` |

### 検索式の3パターン用意

Cowork は以下の3パターンを立案し、利用者と合意してから count に進みます。

| パターン | 検索式 | 想定ヒット数 | 採否の判断 |
|---|---|---|---|
| **基本** | `"Last F"[1au]` | 10〜200件 | 業績数として妥当ならこれを採用 |
| **表記揺れ** | `("Last F"[1au] OR "Last FM"[1au])` | 基本+α | 別表記の論文が複数想定される場合 |
| **絞り込み** | 基本 + 所属/ORCID/出版年 | 基本より少 | 同名著者問題が出た場合のみ |

### MeSH の役割（検索後の集計）

MeSH は **検索後の集計** で使います。具体的には:

- 取得した全論文の MeSH terms を集計し、頻出 Top 10 を抽出 → 対象者の研究テーマ可視化（3-E参照）
- 検索式の絞り込みには **使わない**（First Author検索で既に対象者の業績が絞れているため、テーマで切ると業績の一部が漏れる）

---

## 3-B. count による検証（必須）

### 実行ツール

```
mcp__e5502dd0-...__count
  query: <立案した検索式>
```

### 判定基準

| ヒット数 | 判定 | 対処 |
|---|---|---|
| 0件 | 著者名表記が違う | 代表論文 PMID を `fetch` で確認し、PubMed上の正しい表記に修正 |
| 1〜5件 | 業績が少ない or 表記が違う | 表記揺れパターン（OR連結）で再検索 |
| 6〜100件 | 適切 | search に進む |
| 101〜300件 | やや多い | First Author なら通常許容範囲。同名著者疑いがあれば代表論文の有無を確認 |
| 301件以上 | 同名著者混入の可能性大 | ORCID か所属で絞り込む |

### 利用者への確認

count 結果を以下のように提示し、進めるか確認:

```
検索式: "Misawa M"[1au]
ヒット数: 47件

このまま search に進んでよいですか?
- 念のため絞る場合: 所属追加 → "Misawa M"[1au] AND "Showa University"[Affiliation] → 推定 35件前後
- 同名著者の確認をする場合: 代表論文 PMID 29360439 が結果に含まれるか確認
```

---

## 3-C. search による論文取得

### 実行ツール

```
mcp__e5502dd0-...__search
  query: <count で確定した検索式>
  retmax: 200  # 多めに取って後で絞る
  sort: pub_date  # 出版年降順
```

### 取得後の処理

1. 全 PMID リストを取得
2. `fetch_batch` で書誌情報（タイトル・著者・雑誌・年・MeSH・abstract）を一括取得
3. 結果を以下の構造で整理

```yaml
publications:
  - pmid: "29360439"
    first_author: "Misawa M"
    last_author: "Mori Y"
    journal: "Gastroenterology"
    year: 2018
    title: "Artificial Intelligence-Assisted Polyp Detection..."
    mesh_terms: ["Colonoscopy", "Artificial Intelligence", "Colonic Polyps"]
    abstract: "..."
    url: "https://pubmed.ncbi.nlm.nih.gov/29360439/"
```

---

## 3-D. 主要論文の選定（Visualizer フォーム）

取得した論文すべてを persona.md に載せると冗長です。**主要論文10〜20本** に絞ります。

### 選定基準（自動推定）

Cowork が以下のスコアで自動ランキングし、上位20本を初期候補として提示します。

このバンドルでは検索段階で First Author に絞っているため、すべての論文が「対象者が First Author」です。スコアは impact / 被引用 / 新しさ の3軸で付けます。

| 観点 | 重み |
|---|---|
| 雑誌の impact 想定（Nature/NEJM/Lancet系= +3, 専門トップ誌= +2, 一般誌= +1） | 加点 |
| 被引用数（`get_citation_counts` で取得） | 100超= +2, 50超= +1 |
| 最近5年以内 | +1点 |

### 利用者への提示（Visualizer）

```
20本の主要論文候補をリストで提示します。残す論文にチェックを入れてください。
（チェックボックス + 並び替え可能）

[✓] 2018  Misawa M  Gastroenterology  AI-Assisted Polyp Detection...   被引用321
[✓] 2021  Misawa M  Endoscopy         Real-time CADe...                 被引用187
[ ] 2015  Misawa M  J Gastroenterol   Conventional polypectomy...       被引用23
...
```

### 確定後の処理

選定された論文を `outputs/publications.md` に書き込む（後述のテンプレ参照）。

---

## 3-E. MeSH 頻出語・共著者・主要雑誌の集計

選定された論文（または検索ヒット全件）を対象に、以下を集計。

### MeSH 頻出語 Top 10

```python
# 擬似コード
mesh_counter = Counter()
for paper in papers:
    for term in paper.mesh_terms:
        mesh_counter[term] += 1
top10 = mesh_counter.most_common(10)
```

→ persona.md の「MeSH 頻出語 Top 10」テーブルへ。

### 共著者 Top 10

```python
coauthor_counter = Counter()
for paper in papers:
    for author in paper.all_authors:
        if author != target_author:
            coauthor_counter[author] += 1
top10 = coauthor_counter.most_common(10)
```

→ persona.md の「主要共著者」テーブルへ。

### 主要投稿雑誌

```python
journal_counter = Counter(p.journal for p in papers)
```

→ persona.md の「主要投稿雑誌」テーブルへ。

---

## 3-F. 研究テーマの自己定義（要約生成）

集計結果から、Cowork が研究テーマを3〜5文で要約します。

### 生成テンプレ

```
{対象者名}は、{頻出MeSH 1位}を中心に、{頻出MeSH 2-3位}との接点で
{過去N年間に M本}の論文を発表している。主たる発表媒体は{主要雑誌1-2位}で、
研究の中核は{1st author論文の傾向から推定したテーマ}にある。
最近の{直近2-3年}の論文では、{新しいMeSH語の出現傾向}へのシフトが見られる。
```

利用者に提示し「このテーマ要約で合っていますか？」と確認。修正があれば反映してから persona.md へ書き込み。

---

## 3-G. outputs/publications.md への保存

### テンプレート

```markdown
# Publications — {対象者名}

検索実施日: 2026-04-29
検索式: {実際に使った検索式}
ヒット数: {N}件
選定: 主要論文{M}本

## 主要論文一覧

| # | 1st author | 雑誌 | 年 | タイトル | 被引用 | リンク |
|---|---|---|---|---|---|---|
| 1 | [Misawa M](https://pubmed.ncbi.nlm.nih.gov/29360439/) | *Gastroenterology* | 2018 | AI-Assisted Polyp Detection | 321 | [PMID](https://pubmed.ncbi.nlm.nih.gov/29360439/) |
| ... | ... | ... | ... | ... | ... | ... |

## MeSH 頻出語 Top 10

| 順位 | MeSH | 頻度 |
|---|---|---|
| 1 | Colonoscopy | 24 |
| ... | ... | ... |

## 主要共著者 Top 10

| 順位 | 共著者 | 共著回数 |
|---|---|---|
| ... | ... | ... |

## 主要投稿雑誌

| 雑誌 | 投稿数 |
|---|---|
| ... | ... |

## 研究テーマ要約

{3〜5文の要約}
```

---

## 3-H. 完了基準

Step 3 が完了した状態は以下のとおりです。

- [ ] 検索式（広め/標準/狭め）の3案を立案・count検証
- [ ] 確定した検索式で search 実行・全PMID取得
- [ ] 主要論文10〜20本を利用者と合意して選定
- [ ] MeSH頻出語・共著者・主要雑誌を集計
- [ ] 研究テーマ要約を生成・利用者と合意
- [ ] `outputs/publications.md` に保存
- [ ] persona.md の業績セクションへ反映（許可確認あり）
- [ ] PROGRESS.md の Step 3 行に追記済み

完了したら以下を発話:

```
### ゴール
Step 3 完了。{対象者名}の主要論文{M}本・MeSH頻出語・共著者を outputs/publications.md と persona.md に保存しました。

### 次のステップ（Step 4）
英文ライティングスタイル抽出に進みます。Step 3 で選定した主要論文のうち、FullText が取得できる論文（PMC OA または添付PDF）から英文の語彙・文構造・展開パターンを分析します。

「Step 4 に進みます」と打ってください。
```

---

## トラブルシューティング

| 症状 | 対処 |
|---|---|
| 同名著者が多くて絞れない | ORCID で絞る（`"0000-0002-XXXX-XXXX"[Author - ORCID]`）または所属を AND で追加 |
| 著者名の英表記揺れがある | `("Misawa M"[1au] OR "Misawa MA"[1au])` のように `[1au]` ベースで OR連結 |
| `[1au]` で結果が0件 | 代表論文 PMID を `fetch` で確認し、Author欄の正しい表記を採用 |
| First Author の論文が極端に少ない（学生・研修医など） | 利用者と相談のうえ `[au]` 検索（共著者含む）に切り替え。共著での書き分けは Step 4 の英文スタイル抽出時に注記する |
| 被引用数取得に時間がかかる | スコアリングは無くても可。利用者と合意のうえスキップ |
