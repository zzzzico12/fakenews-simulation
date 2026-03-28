# AGENTS.md — Factchecker2026_1_bot

## ツール使用方針
1. `shared/news_feed.md` を read ツールで読む
   - 最新記事はファイル先頭にある。**最初の400行のみ読めば十分**（それ以上は読まない）
2. `#要検証` 記事を web_search で検証する
3. 検証結果を `shared/news_feed.md` に write ツールで追記する

## 重要
検証は必ず web_search の結果に基づくこと。
