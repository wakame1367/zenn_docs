---
title: "EfficientNetを最速で試す方法" # 記事のタイトル
emoji: "😸" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["Python", "tensorflow"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---


# TL;DR
- Tensorflow Hubにて[imagenetによる事前学習済み重み]((https://tfhub.dev/google/collections/efficientnet/1))が公開されている
 - EfficientNetB0 - B7まですべて公開
 - 転移学習用のfeature vectorsが公開されているのでそれを使う
- [公式のTensorFlow Hubを使った転移学習のチュートリアル](https://www.tensorflow.org/tutorials/images/transfer_learning_with_hub)と共に、EfficientNetを最速で試す方法の紹介
 - GoogleColabratory環境で[画像分類のデモ]((https://colab.research.google.com/drive/18IVEXsLwHnuXq1GU18-6LvsuuDCjjYaD))を動かす
 - 環境さえ用意できればデモと同じコードで動作確認できるはず 
 - やることは[tensorflow-hub](https://pypi.org/project/tensorflow-hub/)ライブラリでDLするモデルを表すURLを変えるだけ

# はじめに

「EfficientNet」については素晴らしい記事がありますのでこちらをご覧ください。
[Qiita - 2019年最強の画像認識モデルEfficientNet解説](https://qiita.com/omiita/items/83643f78baabfa210ab1)

今回の記事はEfficientNetを動かす方法の紹介に重きをおきます。

## 早速デモを動かしてみる

動かすデモのGoogleColabratoryは用意しておきましたのでこちらをお使いください。

[GoogleColabratory - transfer_learning_EfficientNet](https://colab.research.google.com/drive/18IVEXsLwHnuXq1GU18-6LvsuuDCjjYaD)

上記GoogleColabratoryより肝になるコード、モデルの定義部分だけ抽出してみました。これだけでEfficientNetを利用できるようになります。
あとの学習部分をどうするか気になる方は上のデモを動かしてみてください。

```python
import tensorflow as tf
import tensorflow_hub as hub
from tensorflow.keras import layers

# クラス数はデータに合わせて適当に変えてください
num_classes = 10

# URLはこちらのページ https://tfhub.dev/google/collections/efficientnet/1
# 末尾に記載されているURLから使いたいモデル(B0-B7)までのどれかを選んでください
# 今回はB0を使います
feature_extractor_url = "https://tfhub.dev/google/efficientnet/b0/feature-vector/1"

# width/heightについてはB0は(224, 224)が推奨とされているのでそうしています
# 推奨のwidth/heightについてはこちらのページをご覧ください https://tfhub.dev/google/collections/efficientnet/1
feature_extractor_layer = hub.KerasLayer(feature_extractor_url,
                                         input_shape=(224,224,3))
# 学習済み重みは固定
feature_extractor_layer.trainable = False

# Keras functional APIで動くかなと試したのですがうまく動かず
# 公式Tutorialに倣って以下の通りにしています
model = tf.keras.Sequential([
  feature_extractor_layer,
  layers.Dense(num_classes, activation='softmax')
])
```

# 参考
- [Qiita - 2019年最強の画像認識モデルEfficientNet解説](https://qiita.com/omiita/items/83643f78baabfa210ab1)
- [tfhub.dev - efficientnet](https://tfhub.dev/google/collections/efficientnet/1)
- [公式のTensorFlow Hubを使った転移学習のチュートリアル](https://www.tensorflow.org/tutorials/images/transfer_learning_with_hub)
