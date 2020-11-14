---
title: "KaggleへのサブミットをGithub Actionsを使って自動化する" # 記事のタイトル
emoji: "🤔" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["Python", "Kaggle"] # タグ。["markdown", "rust", "aws"]のように指定する
published: false # 公開設定（falseにすると下書き）
---

# はじめに
この記事は [Kaggle Advent Calendar 2020のアドベントカレンダー](https://qiita.com/advent-calendar/2020/kaggle)6日目の記事です。

去年の[Kaggle-part2 Advent Calendar 2019のアドベントカレンダー](https://qiita.com/advent-calendar/2019/kaggle-part2)に書いた[記事](https://zenn.dev/wakame/articles/20201114_kaggle_code_management)の続きの内容になります。続きの記事ですがなるべくこの記事で完結できるような内容になるよう書きました。

また、こちら[Kaggleの学習から投稿までをAWS, GitHub Actionsを使って自動化する](https://tepppei.hatenablog.com/entry/2020/07/19/120625)本記事のリスペクト元になります。この記事を書くにあたり大きく影響を受けております。

# この記事を読むとできるようになること

- `git push`して`merge`するとGithub Actionsが走り、Kaggle Notebookが生成・動作するという流れの自動化。

# この記事で説明すること
- 上記のKaggleへのサブミット自動化までの方法を主に記事で取り扱います。
- 逆に説明しないこととして、設定するパラメータの仕様等は参照URLを紹介するレベルに留めます。

# なぜやるのか

- 「Gitでコード管理しているのに、KaggleのNotebook提出用に別のコードを用意するのが面倒だ」、Kaggleの`Code Competition`というルールの場合に発生する悩みだと思います。
  - `Code Competition`について簡単に説明すると、CSVなどのファイルを直接アップロードするのではなく、Notebookを利用してそこにファイル提出するまでのソースコードを書きそれを提出するという形式のコンペのことです。
  - `Code Competition`について詳しく知りたい方は以前の記事に説明を書きましたのでそちらを読んでもらえると嬉しいです。([記事](https://zenn.dev/wakame/articles/20201114_kaggle_code_management))
- これらの悩みを解消したのが前回の記事でしたが、今回はさらにサブミットも自動化すれば楽になるなということで試してみました。

# 自動化までに必要なこと
以降は具体的にどうやって解決するのか、方法の部分の説明になります。

作業の流れはこのような形になっております。

- 提出するコードを用意する
- Github SecretsにAPI Tokenを設定
- Github TemplateをCloneする
- Github Templateのデフォルト設定を修正
- Github Actionsのデフォルト設定を修正

各項目について説明します。

## 提出するコードを用意する

参考として[Kaggle - petfinder-adoption-prediction](https://www.kaggle.com/c/petfinder-adoption-prediction)への提出自動化したリポジトリを作成しておきました。[Github - wakamezake/kaggle_petfinder](https://github.com/wakamezake/kaggle_petfinder)参考に見てもらえればと思います。

Github Actions動作確認のため、はじめに提出するコードは簡単なアルゴリズムにしておくとすぐ結果が確認できます。

## Github TemplateをCloneする
Github TemplateのURLは[Github - wakamezake/kaggle_submission_pipeline_template](https://github.com/wakamezake/kaggle_submission_pipeline_template)になります。こちらをCloneしてください。
以降Cloneしたリポジトリの中身を修正してゆきます。

## Github SecretsにAPI Tokenを設定

CloneしたリポジトリのGithub SecretsにKaggleAPIを利用するためのAPI Tokenを設定します。

Github Actionsで参照しているGithub Secretsをそれぞれ、KAGGLE_USERNAME、KAGGLE_KEYとしているのでそれに合うようにユーザ名とKeyを設定してください。具体的な設定の方法は次の[URL](https://docs.github.com/en/free-pro-team@latest/actions/reference/encrypted-secrets#creating-encrypted-secrets-for-a-repository)を参照してください。

## Github Templateの設定を修正

修正部分を下記にまとめましたのでそれに従って設定を修正します。

- 修正点
  - easy_goldというフォルダ名を適当な名前に変更してください。またこのフォルダに提出用のコードを置いてください。
  - script_template.py内のeasy_goldの部分を変更した名前に修正してください。`run('python easy_gold/main.py')`となっていますが実行して欲しいpythonファイルを設定してください。
  - setup.py内のeasy_goldの部分を変更した名前に修正してください。

具体的な修正点は[Github - wakamezake/kaggle_submission_pipeline_template#correctional-point](https://github.com/wakamezake/kaggle_submission_pipeline_template#correctional-point)にも記載しているので参考にしてください。

## Github Actionsのデフォルト設定を修正

修正部分を下記にまとめましたのでそれに従って設定を修正します。

- 修正点
  - [upload.yml](https://github.com/wakamezake/kaggle_submission_pipeline_template/blob/master/.github/workflows/upload.yml)内の`with`以下の値に、提出に使うNotebook名等を設定して下さい。
  - 各パラメータの仕様については[KaggleAPI - Kernel-Metadata](https://github.com/Kaggle/kaggle-api/wiki/Kernel-Metadata)を参照して下さい。
  - 例として[Kaggle - petfinder-adoption-prediction](https://www.kaggle.com/c/petfinder-adoption-prediction)に提出する場合の[upload.yml](https://github.com/wakamezake/kaggle_petfinder/blob/master/.github/workflows/upload.yml)を紹介します。

具体的な修正点は[Github - wakamezake/kaggle_submission_pipeline_template#github-actionuploadyml](https://github.com/wakamezake/kaggle_submission_pipeline_template#github-actionuploadyml)にも記載しているので参考にしてください。

## 作業用リポジトリへの`push`
CloneしてきたリポジトリのGithub Actionsには「`pull_request`がされたらKaggle Notebookを生成する」というワークフローを設定してあります、適当なbranchを切って`push`、`merge`してみてください。Github Actionsが正しく動作していればKaggle Notebookが生成されているはずです。


説明は以上になります。

# おわりに

今回はKaggleへのサブミットをGithub Actionsを使って自動化してみました。前回の記事に書いた課題を解決できたのでとても満足しています。
今回の記事を実践してみてサブミットが自動化されたKaggle Lifeを是非楽しんでください！

最後に、今回作ったKaggleへのサブミット自動化を実際に適用した例を紹介します。検証に使ったコンペは[Kaggle - petfinder-adoption-prediction](https://www.kaggle.com/c/petfinder-adoption-prediction)で、以下それぞれGithubのリポジトリ、生成されたKaggle Notebookになります。

- [Github - wakamezake/kaggle_petfinder](https://github.com/wakamezake/kaggle_petfinder)
- [Kaggle - wakamezake/petfinder-sample-pipeline](https://www.kaggle.com/wakamezake/petfinder-sample-pipeline)

以上です、最後まで読んで頂きありがとうございました。

# 参考
- [Github - harupy/push-kaggle-kernel](https://github.com/harupy/push-kaggle-kernel)
- [Github - lopuhin/kaggle-script-template](https://github.com/lopuhin/kaggle-script-template)
- [Kaggle CodeCompetitions](https://www.kaggle.com/docs/competitions)
