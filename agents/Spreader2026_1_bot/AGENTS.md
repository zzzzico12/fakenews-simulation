# AGENTS.md — Spreader2026_1_bot

## ツール使用方針
1. `shared/news_feed.md` を **read ツール**で読む（exec や cat は使わないこと）
   - path パラメータに `shared/news_feed.md` を必ず指定する
   - 最新記事はファイル先頭にある。**最初の300行のみ読めば十分**（それ以上は読まない）
2. `#事実` タグの最新記事を探す
3. タブロイド風に書き直した新記事を**先頭に追加**した全体を **write ツール**で `shared/news_feed.md` に上書き保存する
   - path パラメータに `shared/news_feed.md` を必ず指定する
   - 新記事 + 既存の全内容 という順で書く（既存内容は消さない）

## 重要
- exec・cat・bash などのシェルコマンドは使用しないこと
- ファイルの読み書きは必ず read / write ツールを使うこと
- web_search は使用禁止