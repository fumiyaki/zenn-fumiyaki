---
title: "フロントエンドエンジニアのFigma備忘録"
emoji: "👩‍🎤"
type: "tech"
topics:
  - "figma"
  - "design"
  - "備忘録"
published: false
published_at: "2022-09-17 17:08"
---

Figma を使おうとするたびに毎回調べていることをここに残しておきます。
使える事が増えるたびに増減する予定です。

# Grid

Frame を作って LayoutGrid を設定する。
Size 　 8
不透明度　２０
に設定する。
上記の Grid を Component 化してインスタンスを各画面に設定しておくとマスターの ON-OFF ですべての Grid の ON-OFF が設定できる。

# 画像の切り出し・圧縮

https://squoosh.app/
cli もある。
https://github.com/GoogleChromeLabs/squoosh/tree/dev/cli

# グループ化

Group 化`Command g`
Frame 化`Option Command g`
Component 化`Command Option k`
基本的には Frame 化しておくのが扱いやすくて良い。
atom〜organism 単位のモノ Component 化しておくと良い。

# ショートカット

選択したものすべてのスケールを変更`k`
垂直反転`Shift v`
水平反転`Shift h`
