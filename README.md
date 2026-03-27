# 🗞️ Fakenews Simulation

**6体のAIエージェントが自律的にフェイクニュースを生成・拡散・検証・観察するマルチエージェントシミュレーション**

[openclaw](https://github.com/openclaw/openclaw) 上で動作し、毎サイクル自動でニュースが流れ、Telegramにレポートが届く。

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
│   ├── Journalist2026_1_bot/SOUL.md   # 記者の人格定義
│   ├── Spreader2026_1_bot/SOUL.md     # タブロイド記者の人格定義
│   ├── Citizen2026_1_bot/SOUL.md      # 市民1の人格定義
│   ├── Citizen2026_2_bot/SOUL.md      # 市民2の人格定義
│   ├── Factchecker2026_1_bot/SOUL.md  # ファクトチェッカーの人格定義
│   └── Observer2026_1_bot/SOUL.md     # 観察者の人格定義
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

## SOUL.md とは

各エージェントの「人格」を定義するMarkdownファイル。openclaw はこれをシステムプロンプトとして使用する。

```markdown
# タブロイド記者エージェント「Spreader2026_1_bot」

あなたは「週刊〇〇砲」のような煽情的なタブロイド紙で働くライターです。
地味なニュースを派手な見出しに「味付け」して読者の関心を引くのが得意で...
```

エージェントの性格・行動指針・出力フォーマットをすべてMarkdownで記述できる。

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
