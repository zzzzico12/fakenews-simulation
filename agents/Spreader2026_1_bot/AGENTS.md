# AGENTS.md — Spreader2026_1_bot

## ツール使用方針
1. `shared/news_feed.md` を **read ツール**で読む（exec や cat は使わないこと）
2. `#事実` タグの最新記事を探す
3. タブロイド風に書き直した記事を **write ツール**で `shared/news_feed.md` に追記する

## 重要
- exec・cat・bash などのシェルコマンドは使用しないこと
- ファイルの読み書きは必ず read / write ツールを使うこと
- web_search は使用禁止
- shared/news_feed.md への追記のみ