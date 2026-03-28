# 🗞️ Fakenews Simulation

**6体のAIエージェントが自律的にフェイクニュースを生成・拡散・検証・観察するマルチエージェントシミュレーション**

[openclaw](https://github.com/openclaw/openclaw) 上で動作し、毎サイクル自動でニュースが流れ、Telegramにレポートが届く。

---

## このシミュレーションで何を確認するか

フェイクニュースの拡散・信頼判断・ファクトチェックという一連の流れを、AIエージェントで自動化することで、以下を定量的に観察する。

**情報の劣化プロセス**
- 事実のニュースがタブロイド化される過程で、どの要素が誇張・省略されるか
- 誇張度（元記事との乖離）はトピックによって変わるか（政治・経済・技術など）

**受け手の信頼判断の傾向**
- 感情的な見出しと中立的な見出しで、市民の信頼度スコアはどう変わるか
- 市民の属性（会社員 vs 高齢者）によって、引っかかりやすいニュースのカテゴリは異なるか

**ファクトチェックの有効性**
- 検証が入ると市民の信頼スコアはどう変化するか
- 検証が追いつかない（遅延する）場合、誤情報はどの程度広まった状態になるか

**AIエージェント自律運用の検証**
- LLMが「役割」を与えられたときに、どこまで一貫して振る舞えるか
- エージェント間でファイル共有のみを通信手段とした場合、情報の受け渡しは破綻しないか

---

## デモ

```
[記者] SoftBank、OpenAIに400億ドル投資へ──AIインフラ覇権争いが激化  #事実 ✅
    ↓
[タブロイド] 【激震】孫正義、6兆円爆投資!!AIの神様を買い占める気か?!  #要検証 ⚠️
    ↓
[市民1] 「信じた（信頼度：高）」孫さんの投資姿勢から本物と判断
[市民2] 「信じた（信頼度：中）」でも東京ドーム換算は盛りすぎでは…
    ↓
[ファクトチェッカー] 概ね正確。「6兆円」は誇張。元記事は「最大400億ドル」  ✅
    ↓
[Observer → Telegram] 📊 4時間レポート: 事実2件 / 要検証2件 / デマ拡散率 50%
```

---

## アーキテクチャ

```
┌─────────────────────────────────────────────────────┐
│                   shared/news_feed.md               │
│                                                     │
│  Journalist ──→ Spreader ──→ Factchecker            │
│  （事実投稿）   （タブロイド化）  （検証）             │
│                                                     │
│            Citizen1 / Citizen2                      │
│            （信頼判断 → trust_scores.md）            │
│                                                     │
│  Observer（全体観察 → reports/ + Telegram通知）      │
└─────────────────────────────────────────────────────┘
```

| エージェント | 役割 | 実行間隔 |
|------------|------|---------|
| 📰 Journalist | 実ニュースをWeb検索して投稿 | 4時間毎 :00 |
| 📢 Spreader | ニュースをタブロイド風に誇張 | 4時間毎 :10 |
| 👤 Citizen1 | 会社員。感情的見出しに流されやすい | 4時間毎 :20 |
| 👴 Citizen2 | 高齢者。政府・健康・お金に反応しやすい | 4時間毎 :25 |
| 🔍 Factchecker | 要検証記事をWeb検索で検証 | 4時間毎 :40 |
| 📊 Observer | 全体集計・レポート生成・Telegram通知 | 4時間毎 :50 |

---

## 特徴

- **完全自律**: セットアップ後はcronで自動実行、人間の操作不要
- **実ニュース連動**: Tavilyで実際のニュースを取得してシミュレーション
- **定量的観察**: 信頼スコア・デマ拡散率をObserverが自動集計
- **Telegram連携**: Observerのレポートが定期的にスマホに届く
- **サンドボックス分離**: 各エージェントはDockerサンドボックスで隔離実行

---

## 必要なもの

- Docker / Docker Compose
- [openclaw](https://github.com/openclaw/openclaw)
- Anthropic API キー
- Tavily API キー（Web検索用、無料枠あり）
- Telegram アカウント + BotFather で6ボット作成

---

## セットアップ

詳細は [SETUP.md](SETUP.md) を参照。概要:

```bash
# 1. openclaw をビルド
git clone https://github.com/openclaw/openclaw.git
cd openclaw
docker build -t openclaw:local .
docker build -f Dockerfile.sandbox -t openclaw-sandbox:bookworm-slim .

# 2. このリポジトリをワークスペースとして配置
git clone https://github.com/yourname/fakenews-simulation.git
~/.openclaw/workspace/fakenews

# 3. 設定ファイルを用意
cp .env.example .env               # APIキーを記入
cp openclaw.json.example ~/.openclaw/openclaw.json  # botトークンを記入
cp cron_jobs.example.json ~/.openclaw/cron/jobs.json

# 4. 起動
docker compose up -d

# 5. Telegram ペアリング（各ボットに /start → コードを承認）
openclaw pairing approve telegram {CODE}
```

---

## ファイル構成

```
fakenews/
├── agents/
│   ├── Journalist2026_1_bot/
│   │   └── SOUL.md                    # 記者の人格定義・行動指針
│   ├── Spreader2026_1_bot/
│   │   ├── SOUL.md                    # タブロイド記者の人格定義・行動指針
│   │   └── AGENTS.md                  # ツール使用方針（read/write限定）
│   ├── Citizen2026_1_bot/
│   │   └── SOUL.md                    # 市民1の人格定義（会社員）
│   ├── Citizen2026_2_bot/
│   │   └── SOUL.md                    # 市民2の人格定義（高齢者）
│   ├── Factchecker2026_1_bot/
│   │   ├── SOUL.md                    # ファクトチェッカーの人格定義・検証フォーマット
│   │   └── AGENTS.md                  # ツール使用方針（read/write/web_search）
│   └── Observer2026_1_bot/
│       └── SOUL.md                    # 観察者の人格定義・レポートフォーマット
├── shared/                            # エージェント間の共有データ（gitignore）
│   ├── news_feed.md                   # 記事が蓄積されるメインファイル
│   ├── trust_scores.md                # 市民の信頼判断ログ
│   └── reports/                       # Observerの自動レポート
├── openclaw.json.example              # 設定テンプレート（botトークンなし）
├── cron_jobs.example.json             # cronテンプレート（チャットIDなし）
├── .env.example                       # 環境変数テンプレート
└── SETUP.md                           # 詳細構築手順・トラブルシューティング
```

---

## 他に試せるパターン

SOUL.md とcron設定を変えるだけで、様々なシナリオに応用できる。

### エージェント構成のバリエーション

| パターン | 変更点 | 確認できること |
|---------|--------|-------------|
| **市民を増やす** | Citizen3〜5を追加（10代・医療従事者・陰謀論者など） | 属性によって騙されるニュースのジャンルが違うか |
| **Spreaderを複数に** | Spreader2（SNS拡散型）を追加 | タブロイドとSNSで誇張スタイルが異なるか |
| **Factcheckerなし** | factchecker_runを無効化 | 検証者がいない場合の誤情報到達率の変化 |
| **Factcheckerの遅延** | cronを `:50`（Observerの直前）に変更 | 検証が遅れると信頼スコアに影響が出るか |

### ニュースカテゴリの絞り込み

Journalistの`SOUL.md`でトピックを限定することで、特定分野に特化した観察ができる。

```markdown
# 例: 健康・医療ニュース専門に絞る
今日のニュースは「健康・医療」カテゴリのみ検索すること。
ワクチン、サプリ、新薬、がん治療などのトピックを優先する。
```

健康・医療は誇張されやすく、かつ高齢者市民が特に反応するため、拡散率が高くなる傾向を観察できる。

### 言語・地域の変更

```markdown
# 例: 英語ニュース + 国際市民エージェント
- Journalist: English news only (US/EU politics, tech)
- Citizen_US: American, easily swayed by political news
- Citizen_EU: European, skeptical of tech monopolies
```

### 検証手法の比較

Factcheckerを2体にして、異なる検証スタイルを持たせる実験。

```markdown
# Factchecker_Strict: ソースを3件以上引用しないと確定しない
# Factchecker_Fast: スピード重視、1件のソースで即判定
```

どちらのスタイルが「正確さ」と「速度」のトレードオフで優れているかを観察できる。

### Observerのレポート拡張

Observerの`SOUL.md`に分析項目を追加することで、レポートをリッチにできる。

```markdown
## レポートに含める分析
- 最も誇張された記事TOP3（元記事との差分スコア付き）
- 市民別の正答率（信じた記事のうち #事実 だった割合）
- ファクトチェックが間に合わなかった記事一覧
```

---

## SOUL.md とは

各エージェントの「人格」を定義するMarkdownファイル。openclaw はこれをシステムプロンプトとして使用する。

```markdown
# タブロイド記者エージェント「Spreader2026_1_bot」

あなたは「週刊〇〇砲」のような煽情的なタブロイド紙で働くライターです。
地味なニュースを派手な見出しに「味付け」して読者の関心を引くのが得意で...
```

エージェントの性格・行動指針・出力フォーマットをすべてMarkdownで記述できる。

## AGENTS.md とは

ツール使用方針を補足するファイル。SOUL.md はシステムプロンプトとして読み込まれるが、AGENTS.md はエージェントが参照する追加指示ファイルとして機能する。

openclawの isolated cron 実行では、エージェントは Docker サンドボックス内で起動し、`shared/` ディレクトリへのアクセスに gateway の **read/write ツール**を使う必要がある（sandbox 内の exec/cat は機能しない）。AGENTS.md でこの使い分けを明示することで、エージェントが正しいツールを選択できるよう補助している。

```markdown
## ツール使用方針
1. `shared/news_feed.md` を **read ツール**で読む（exec や cat は使わないこと）
2. `#事実` タグの最新記事を探す
3. タブロイド風に書き直した記事を **write ツール**で `shared/news_feed.md` に追記する
```

---

## 実装上の工夫・ハマりどころ

このシミュレーションを構築・運用する中で判明した重要な知見をまとめる。

### 1. `tools.profile: "coding"` が必須

openclaw.json の `tools.profile` を `"messaging"` にすると、isolated cron 実行時に gateway の read/write ツールが正しく注入されない。全エージェントが exec/cat を試みてファイルアクセスに失敗する。

```json
"tools": {
  "profile": "coding"   ← これが必須。"messaging" にすると isolated cron で read/write が使えない
}
```

### 2. AGENTS.md でツール使用を明示する

isolated cron 実行ではエージェントが Docker サンドボックス内で起動し、`exec` や `cat` でのファイル操作が機能しない。SOUL.md だけでなく AGENTS.md にも「read/write ツールを使うこと」「exec は使わない」と明示することで、モデルが正しいツールを選択する。

### 3. タグはエージェント間で厳密に統一する

Spreaderが出力するタグ（`タグ: #要検証`）と Factchecker が探すタグ（`#要検証 タグがついた記事`）は完全一致していないとスルーされる。片方だけ変えると連鎖が途切れる。

### 4. 「タブロイド記者」として定義してフィルター回避

SOUL.md で「フェイクニュース生成」「デマ拡散」という表現を使うと Claude の安全フィルターが反応してコンテンツ生成を拒否する。「週刊誌のタブロイド記者」「センセーショナルな書き方が得意なライター」として定義することで回避できる。

また、拒否履歴が積まれたセッションは以降も拒否し続けるため、isolated cron（毎回新規セッション）で実行することが重要。

### 5. news_feed.md の読み込み行数を制限する

`news_feed.md` が肥大化するとモデルがページネーションを試みて `Offset N is beyond end of file` エラーが発生する。最新記事はファイル先頭に配置されているため、SOUL.md / AGENTS.md に「最初のN行のみ読む」と明示することでエラーを防げる（アーカイブ機構は不要）。

---

## 追加の変更・知見（2026-03-29）

### 6. AGENTS.md に `path` パラメータを明示しないと read ループが発生する

openclaw の `patchToolSchemaForClaudeCompatibility` 関数は、`path` パラメータに `file_path` / `filePath` / `file` のエイリアスを追加する際に、`path` を `required` から削除する。しかしエイリアスは `required` に追加されないため、**モデルからはすべてのパラメータが任意に見える**。

その結果、モデルが `read({})` のようにパスなしでツールを呼び出し、エラー `"Missing parameter: path alias. Supply correct parameters before retrying."` を受け取ってもループし続ける問題が発生する。

**対策**：AGENTS.md に以下を明示する。

```markdown
- `shared/news_feed.md` を **read ツール**で読む
  - path パラメータに `shared/news_feed.md` を必ず指定する
  - exec・cat・bash などのシェルコマンドは使用しないこと
```

SOUL.md だけでは不十分で、AGENTS.md に具体的なファイルパスとパラメータ名を記載することが重要。

### 7. trust_scores.md は新エントリを先頭に追加する（prepend 形式）

市民エージェントが trust_scores.md に追記する際、末尾追加（append）にすると Observer やデバッグ時に最新エントリを読むために全行を読む必要が生じる。

**新エントリを先頭に追加（prepend）** することで、Observer が `limit=300` で先頭を読むだけで最新の反応ログを確認できる。

SOUL.md の手順を以下のように変更する：
1. `shared/trust_scores.md` を read で読む（既存内容取得）
2. 新エントリを先頭に追加した全体を write で上書き保存

---

## セキュリティ

- 全エージェントをDockerサンドボックスで隔離（`sandbox.mode: "all"`）
- `exec` ツールは全エージェントに与えない（最小権限）
- APIキー・botトークンは `.env` 管理、リポジトリに含まない
- Gateway は loopback のみにバインド（外部アクセス不可）
- `openclaw security audit` で定期的に監査（現状: 0 critical）

---

## License

MIT
