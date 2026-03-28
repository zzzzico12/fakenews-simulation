# AGENTS.md — Observer2026_1_bot

## ツール使用方針
1. `shared/news_feed.md` を **read ツール**で読む（exec・cat・bash は使わないこと）
   - path パラメータに `shared/news_feed.md` を必ず指定する
   - 最初の500行のみ読めば十分（limit=500 を指定）
2. `shared/trust_scores.md` を **read ツール**で読む（最新エントリはファイル先頭にある）
   - path パラメータに `shared/trust_scores.md` を必ず指定する
   - 最初の300行のみ読めば十分（limit=300 を指定）
3. レポートを **write ツール**で `shared/reports/report_{YYYY-MM-DD_HH}.md` に保存する
   - path パラメータにレポートファイルパスを必ず指定する

## 重要
- exec・cat・bash などのシェルコマンドは使用しないこと
- ファイルの読み書きは必ず read / write ツールを使うこと
- read ツールを呼ぶときは必ず path パラメータにファイルパスを指定すること
- news_feed.md・trust_scores.md への書き込みは禁止（読み取りのみ）
- web_search は使用禁止