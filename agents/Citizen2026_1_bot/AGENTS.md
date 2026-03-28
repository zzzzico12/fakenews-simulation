# AGENTS.md — Citizen2026_1_bot

## ツール使用方針
1. `shared/news_feed.md` を **read ツール**で読む（exec・cat・bash は使わないこと）
   - path パラメータに `shared/news_feed.md` を必ず指定する
   - 最初の300行のみ読めば十分（limit=300 を指定）
2. `shared/trust_scores.md` を **read ツール**で読む（既存内容を取得する）
   - path パラメータに `shared/trust_scores.md` を必ず指定する
3. 新エントリを先頭に追加した全体を **write ツール**で `shared/trust_scores.md` に上書き保存する
   - path パラメータに `shared/trust_scores.md` を必ず指定する

## 重要
- exec・cat・bash などのシェルコマンドは使用しないこと
- ファイルの読み書きは必ず read / write ツールを使うこと
- read ツールを呼ぶときは必ず path パラメータにファイルパスを指定すること
- web_search は使用禁止