---
title: "Kaggleでのソースコード管理の手助け" # 記事のタイトル
emoji: "😸" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["Python", "Kaggle"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

# はじめに
**「Gitでコード管理しているのに、KaggleのNotebook提出用に別のコードを用意するのが面倒だ」**。この不満、心当たりがある方が多いのではないでしょうか。

今回書く記事はその不満を解消する方法を発見しましたので共有し、その方法について解説するというのが趣旨になります。

また記事内容ですが主にこちらの[Kaggle Kernel - lopuhin/imet-2019-submission](https://www.kaggle.com/lopuhin/imet-2019-submission)の内容を参考に書いております。

前置きのために記事では`Code Competition`とはから説明していますが本題から読みたい方は[#Git等で管理しているファイル構造をそのまま使う](#Git等で管理しているファイル構造をそのまま使う)から読んで下さい。

# Code Competitionとは

冒頭で触れた`Code Competition`について説明します。

[Kaggleのドキュメント](https://www.kaggle.com/docs/competitions#kernels-only-competitions)で説明されている内容を引用しますとこのようになっています。

> Some competitions are code competitions. In these competitions all submissions are made from inside of a Kaggle Notebook, and it is not possible to upload submissions to the Competition directly.

下記は意訳です。

> CSVなどのファイルを直接アップロードするのではなく、Notebookを利用してそこにファイル提出するまでのソースコードを書きそれを提出するという形式のコンペを指します。

開催中(2019/12/25現在)のコンペ8種の内(Featured/Researchのみカウント)6種類が`Code Competition`であるのでおそらく参加されている方も多いと思います。

## [参考] Code Competition 例

赤枠で囲った`Code Competition`というタグが付けられたコンペが`Code Competition`になります。

![code-competitionn.PNG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/40310/7bd3d9c5-10ec-ef38-3dcb-f94bc11c5945.png)

## Code Competitionでの提出方法

Kaggle内でコードを書こうとすると`Scripts`と`Notebooks`という2種類の記述する手段があり、`Code Competition`では2つの形式のどちらかに提出ファイルを生成するまでのコードを書きそれを提出する形になります。

## Code Competitionの不満点

Kaggleに提出するソースコードはデバッグやテストのしやすさを考えると、こんな感じで処理毎にモジュール(ファイル)で分けたいという思いがあると思います。([タイタニックコンペ](https://www.kaggle.com/c/titanic)に取り組んだ際の一例です)

```:タイタニックでのファイル構造例
titanic_sample
  │  dataset.py
  │  model.text
  │  predict.py
  │  train.py
  │  utils.py
  │  __init__.py
  │
  ├─resources
     gender_submission.csv
     test.csv
     train.csv

```

上記のファイル構造で提出できればよいのですが、ScriptsかもしくはNotebooksにほぼ提出専用のコードを書くのが`Code Competiton`のルール上通る道であり現状の`Code Competiton`の不満点の一つです。

# Git等で管理しているファイル構造をそのまま使う！
## パッケージとしてインストールしてしまおう

遅くなりましたが、本題に入ります。

Gitで管理しているコード郡を`Notebook`環境下でそのまま使えれば、KaggleのNotebook提出用に別のコードを用意する必要なんてないわけです。というわけで`Notebook`環境下にそれらのコードを用意してしまうというのが今回説明する解決方法になります。

## 概要

KaggleのNotebookにGitで管理しているコードをアップロードして、それらをパッケージとしてインストールしSubmitまでの書かれたコードをコマンドライン上から実行するというのが以降で説明する手順になります。

## Submitするまでの手順説明

説明をわかりやすくするために[タイタニックのデータ](https://www.kaggle.com/c/titanic/data)を使って説明します。
また、前処理、モデルの学習等のコードはこちらを参考にさせていただきました。
[Kaggle Kernel - sishihara/upura-kaggle-tutorial-04-lightgbm](https://www.kaggle.com/sishihara/upura-kaggle-tutorial-04-lightgbm)

### 手順①「GithubTemplateをForkする」
[Github - lopuhin/kaggle-script-template](https://github.com/lopuhin/kaggle-script-template)をForkします。

### 手順②「GithubTemplateをTemplateリポジトリに設定する」
ForkしたリポジトリをTemplateリポジトリにする。この作業は色々なコンペでこのTemplateリポジトリを使い回すためにやっています。
参考に私が作成したTemplateリポジトリのURLを貼っておきます。
[Github - wakamezake/kaggle-script-template](https://github.com/wakamezake/kaggle-script-template)

### 手順③ 「作業用リポジトリの作成」
手順②で作成したTemplateリポジトリを使って、コンペ用のリポジトリを作成する。
[Github - wakamezake/Titanic_submit_script](https://github.com/wakamezake/Titanic_submit_script)

### 手順④ 「自環境用に定数や変数名の修正」
easy_goldディレクトリ([Github - wakamezake/Titanic_submit_script](https://github.com/wakamezake/Titanic_submit_script)だとtitanic_sampleにrenameしてます)にいつもどおりコードを書く。注意点として以下のように最終的に`submission.csv`が出力されるようにコードを書いてください。

```python
def run(command):
    os.system('export PYTHONPATH=${PYTHONPATH}:/kaggle/working && ' + command)

run('python setup.py develop --install-dir /kaggle/working')
run('python titanic_sample/train.py model.text --test_size 0.3')
run('python titanic_sample/predict.py model.text submission.csv')
```

最終的には以下のようなフォルダ構成になると思います。

```:タイタニックでのフォルダ構成例
.
│  .gitignore
│  build.py
│  README.rst
│  script_template.py
│  setup.py
│
├─build
│   .keep
│   script.py
│
├─titanic_sample
  │  dataset.py
  │  model.text
  │  predict.py
  │  train.py
  │  utils.py
  │  __init__.py
  │
  ├─resources
       gender_submission.csv
       test.csv
       train.csv
```

### 手順⑤「Kaggleへサブミット！」
`build.py`を実行します。実行後`build/script.py`が生成されているのでそれを`Notebooks`に貼り付けます。その後はCommitボタンを押してsubmitするといういつもの流れです。
貼り付けた例としてこちらのカーネルを用意しました。[Kaggle Kernel - wakamezake/titanic-submission-sample](https://www.kaggle.com/wakamezake/titanic-submission-sample)

Submitするまでの手順説明は以上になります。

# 各手順の解説

手順の中で実行した`build.py`について。

まずすべてのコードをbase64でエンコードして文字列に変換します、その際エンコードしたコードのファイルパスをkeyにエンコードして文字列をvalueにした辞書を作ります。そして`script_template.py`という`script.py`を作るためのテンプレートの`file_data`部分に辞書の内容を置換します。

`script.py`を`Notebooks`に貼り付けたあとエンコードされたコードをデコードして元のファイル構造を保った状態に戻します。そして`python setup.py develop --install-dir /kaggle/working`を実行して`Notebooks`の環境にデコードしたコードをパッケージとしてインストールします。

解説は以上です。

# おわりに

今回はKaggleの開催形式である`Code Competition`の問題点とそれを解消する方法を紹介しました。`Code Competition`の問題点の解決方法と言いつつ、Titanicコンペという`Code Competition`ではない例を用いて検証してしまいました。

次回、機会がありましたもう少しアップデートした記事をお届けしたいと考えております。

以上です、最後まで読んで頂きありがとうございます。

# 参考
- [Kaggle Kernel - lopuhin/imet-2019-submission](https://www.kaggle.com/lopuhin/imet-2019-submission)
- [Github - lopuhin/kaggle-script-template](https://github.com/lopuhin/kaggle-script-template)
- [Kaggle Kernel - sishihara/upura-kaggle-tutorial-04-lightgbm](https://www.kaggle.com/sishihara/upura-kaggle-tutorial-04-lightgbm)
- [Kaggle Docs](https://www.kaggle.com/docs/competitions#kernels-only-competitions)
- [Kaggle competitions - titanic](https://www.kaggle.com/c/titanic)
