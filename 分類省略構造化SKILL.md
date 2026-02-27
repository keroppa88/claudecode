---
name: stock-market-article-analyzer
description: 米国株・日本株の市況記事を要約・構造化。記事（「米国株、」「東証大引け」で始まる）を分類・分解・省略し、指数の動き（NYダウ/日経平均）・理由・市場センチメントを階層的に抽出。要約結果は米国株市況.txt・日本株市況.txtに追記。記事要約、市況記事要約、米国株市況、日本株市況の分析に使用。
keywords: 記事, 要約, 記事要約, 市況記事, 米国株市況, 日本株市況, 東証大引け, NYダウ, 日経平均, 株式市場, 市況分析
---

# 市況記事の分類・分解・省略・構造化

## Article Classification

**IMPORTANT:** Only activate this skill for articles matching these patterns:

- **Japanese market:** Article title starts with `東証大引け`
  
  - Example: "東証大引け　日経平均は反発..."
  - Output file: `C:\Users\kaika\Desktop\ClaudeCode\日本株市況.txt`

- **US market:** Article title starts with `米国株、`
  
  - Example: "米国株、ダウ反発し319ドル高..."
  - Output file: `C:\Users\kaika\Desktop\ClaudeCode\米国株市況.txt`

- **Other articles:** Do NOT activate this skill

## Output Requirements

- **Determine market type** based on article title classification above
- **Append** to file end (2-3 blank lines spacing)
- Display result in markdown code block

## Output Format

**Japanese market:**

```markdown
### [Article Title] MM/DD（曜日）

* 日経平均株価＝[value]円、[+/-change]円（[+/-percent]％）
* 東証プライム値上がり率＝[percent]％

--------------------------

* [Market movement summary]
  * [Supporting factor]
    * [Detail]
  * 好感された材料/嫌気された材料
    * [Factor]
```

**US market:**

```markdown
### [Article Title] MM/DD（曜日）

* NYダウ＝[value]ドル、[+/-change]ドル（[+/-percent]％）

--------------------------

* [Market movement summary]
  * [Supporting factor]
```

**Note:** US articles use **trading date**, not publication date.

## Formatting Rules

**Title:**

- `### [Article Title] MM/DD（曜日）`
- **Date extraction:**
  - **US market:** Extract trading date from article text starting with "●日の米株式市場で" (e.g., "10日の米株式市場で" → 01/10)
  - **Japanese market:** Extract date from article
- Convert to MM/DD format
- Add Japanese day of week (月火水木金土日)

- **曜日の決定ルール（最重要）:**
  - **曜日は必ず日付から数学的に計算すること**
  - **絶対禁止：「週の最初の取引日だから月曜日」「前日が月曜日だから火曜日」などの推論で曜日を決めてはならない**
  - 祝日・休場日が挟まる場合、取引日の連続性と曜日は一致しない（例：月曜休場なら火曜が週の最初の取引日だが、曜日は「火」）
  - 計算方法：`currentDate`のコンテキストの今日の日付を基準に、対象日付の曜日を正確に算出する
  - 算出後、記事本文に曜日の手がかり（「週明け」「週初め」など）があれば整合性を確認し矛盾があれば記事の記述を優先する

**Index data:**

- Japan: `* 日経平均株価＝XX円、+/-XX円（+/-X.XX％）`
- Japan: Calculate 東証プライム値上がり率: 値上がり ÷ (値上がり + 値下がり + 横ばい) × 100, round to 1 decimal
- US: `* NYダウ＝XX.XXドル、+/-XX.XXドル（+/-X.XX％）`

**Hierarchy:**

- Level 1: `* ` (0 spaces)
- Level 2: `  * ` (2 spaces)
- Level 3: `    * ` (4 spaces)
- Level 4: `      * ` (6 spaces)

**Content:**

- Start with market movement (反発/続落/値を戻した/失速した)
- Add reasons in order of importance
- Use 好感された材料/嫌気された材料 for factors
- Keep sentences short, omit unnecessary particles
- Only use information from article

## Content Rules

**CRITICAL: 仕様に基づくことが最大の要求です。出力前に必ず除外ルールをチェックすること。**

- **Include ONLY:**
  
  - Market-wide factors (景気見通し、金利動向、為替など)
  - Individual stocks **ONLY IF** they had quantifiable market-wide impact (e.g., "ファストリが日経平均を500円押し上げ")
  - Notable divergence (日経平均 vs TOPIX)
  - Trading volume if highlighted (e.g., "2カ月ぶり高水準")

- **MUST EXCLUDE (最重要):**
  
  - **個別銘柄の羅列は基本的に不要** - 記事末尾の「○○や○○が上げた」「××、△△が下げた」などの銘柄リスト
  - 個別銘柄名の詳細列挙（例：「マイクロン・テクノロジー、AMD上昇」→ 除外）
  - 市況全体に影響を与えていない個別銘柄の動き（例：「テスラが下落」→ 市況への影響が明記されていなければ除外）
  - 指数の詳細データの羅列（例：「SOXは4日ぶりに反発し4.0%高」→ 不要）
  - External information not in article
  - Speculation beyond article content

- **個別銘柄記載の判断基準:**
  
  1. 記事に「○○が日経平均を××円押し上げた/押し下げた」と明記されているか？ → YES なら記載
  2. 記事に「○○の決算が市場全体に影響を与えた」と明記されているか？ → YES なら記載
  3. 上記以外の個別銘柄は全て除外 → 「半導体関連株が買われた」「輸出関連株が上昇」など**セクター単位で簡潔に**記載

- **セクター言及のルール:**
  
  - 「半導体関連株が買われた」「輸出関連株が上昇」→ OK（セクター全体の動き）
  - 「半導体関連株が買われた。エヌビディア、AMDが上昇」→ NG（個別銘柄の羅列）
  - 「半導体関連株が買われた。TSMCの決算が好感された」→ OK（具体的な理由がある場合）

## 出力前チェックリスト

**必ず以下を確認してから出力すること:**

1. [ ] 個別銘柄の羅列を全て削除したか？
2. [ ] 記載した個別銘柄は市況全体への影響が明記されているか？
3. [ ] 「○○、△△が上昇」のような銘柄リストを除外したか？
4. [ ] セクター言及に個別銘柄名を含めていないか？
5. [ ] 記事末尾の銘柄リストを機械的にコピーしていないか？
6. [ ] 曜日は日付から数学的に計算したか？（取引日の順序から推論していないか？）

## Common Patterns

- "米国株の上昇が波及" - nest reasons if provided
- "円安ドル高で輸出関連を支えた" - nest details
- "半導体関連に買い" - nest specific news (e.g., TSMC決算)

## Special Cases

- **Multiple indices:** List all, note different directions
- **Mixed signals:** Present all, note which dominated if stated
- **No clear reasons:** Note "Reasons not specified"

## Processing Guidelines（処理方法の重要な注意事項）

**CRITICAL: このセクションは処理効率とコスト削減のために最重要です。**

### 効率的な処理方法

**DO NOT use Task agents for this skill:**
- この構造化タスクは明確で定型的なため、Taskエージェントを使用する必要はありません
- エージェント使用は過剰なトークン消費（150円相当）と処理時間（10分以上）を引き起こします

**正しいアプローチ：**

1. **日本株市況を先に処理する場合：**
   - 東証大引け.txtから記事を読み取る
   - 上記の構造化ルールに従って直接 日本株市況.txt を作成

2. **米国株市況を処理する場合：**
   - **MUST READ 日本株市況.txt FIRST** - 既存の構造を確認する
   - 同じ階層構造とフォーマットを使用して 米国株市況.txt を作成
   - 日本株市況.txtの構造を完全に踏襲すること

3. **複数記事の一括処理：**
   - 記事を1つずつ順番に処理
   - 各記事を構造化してから次の記事へ
   - 全記事を1つのファイル書き込みで完了

### コスト削減のための具体的手順

```
Step 1: 元ファイル（東証大引け.txt または 米国株.txt）を読み取る
Step 2: 日本株市況なら直接処理、米国株市況なら日本株市況.txtを参照
Step 3: 各記事を上記フォーマットに従って構造化
Step 4: 全記事を1回のWriteツールで出力ファイルに書き込む
```

**避けるべき失敗例：**
- ❌ Taskエージェントに丸投げ → 過剰なトークン消費
- ❌ 米国株市況を日本株市況の構造を見ずに作成 → フォーマット不一致
- ❌ 記事ごとに複数回のファイル書き込み → 非効率

### 処理時の確認事項

**米国株市況作成時は特に注意：**
1. 日本株市況.txtを必ず先に読み取って構造を確認したか？
2. 同じ階層構造（第1レベル→第2レベル→第3レベル）を使用しているか？
3. 「好感された材料」「嫌気された材料」のセクション分けを実装したか？
4. 個別銘柄の羅列を全て除外したか？

### 実装例の比較

**❌ 間違った例（構造化不足）：**
```markdown
* 市場の動きのサマリー
  * 1月のCPIが市場予想を下回り、FRBの追加利下げ観測が高まった
  * 半導体製造装置のAMATの業績見通しが好感された
```

**✅ 正しい例（階層的な構造化）：**
```markdown
* 1月の消費者物価指数（CPI）が市場予想を下回り、FRBの追加利下げ観測が高まった
  * 1月のCPIが前年同月比2.4％上昇と市場予想（2.5％上昇）を下回る
    * 6月か7月の利下げ観測を背景に米長期金利が低下
  * 好感された材料
    * 半導体製造装置のアプライドマテリアルズ（AMAT）の業績見通しが好感され、AI市場の拡大期待が高まった
```
