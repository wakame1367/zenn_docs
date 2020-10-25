---
title: "[solafune] 衛星画像から空港利用者数予測ベースモデルのEDA" # 記事のタイトル
emoji: "😸" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["Python", "solafune"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

# はじめに
[solafune - 衛星画像から空港利用者数を予測](https://solafune.com/#/competitions/ea90cba4-3e01-42df-9516-9ac0d7a44204)で簡単にEDAした結果を以下にまとめます。

また、[solafune - 衛星画像から空港利用者数を予測 ルール(情報の取り扱い)](https://solafune.com/#/competitions/ea90cba4-3e01-42df-9516-9ac0d7a44204)より

> ただし、公開した際には、全ての参加者が閲覧できるような状態にしてください。

ということなのでzennにて全ユーザーが閲覧できる状態で公開致します。

cf. @solafune (https://solafune.com)

# 概要
画像コンペということなので学習用/テスト用/スコア評価 画像データに重複がないか調べました。

<script src="https://gist.github.com/wakamezake/0edeac59fc337c242f2a08fe6a945da8.js"></script>



# 参考
- [solafune - 衛星画像から空港利用者数を予測 ルール](https://solafune.com/#/competitions/ea90cba4-3e01-42df-9516-9ac0d7a44204)
