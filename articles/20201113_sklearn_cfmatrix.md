---
title: "sklearn.metrics.ConfusionMatrixDisplayを使った混合行列の可視化" # 記事のタイトル
emoji: "😸" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["Python", "scikit-learn"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

# はじめに
scikit-learnを利用すると簡単に混合行列の可視化は可能だが、[sklearn.metrics.plot_confusion_matrix](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.plot_confusion_matrix.html) は**estimatorが引数に必要になる**。

可視化するだけなのでestimatorが不要な方法はないかと調べていたら [sklearn.metrics.ConfusionMatrixDisplay](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.ConfusionMatrixDisplay.html) が見つかったので簡単にコードを書いてみた。

```python

import matplotlib.pyplot as plt
from sklearn.datasets import load_breast_cancer
from sklearn.metrics import confusion_matrix, ConfusionMatrixDisplay
from sklearn.model_selection import train_test_split
from sklearn.svm import SVC

data = load_breast_cancer()
X, y = data.data, data.target
X_train, X_test, y_train, y_test = train_test_split(X, y, random_state=0)
clf = SVC(random_state=0)
clf.fit(X_train, y_train)

y_pred = clf.predict(X_test)
cm = confusion_matrix(y_pred=y_pred, y_true=y_test)
cmp = ConfusionMatrixDisplay(cm, display_labels=data.target_names)

cmp.plot(cmap=plt.cm.Blues)

```

結果はこんな感じ。

![ダウンロード.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/40310/d565406a-b5f0-c0d0-db0a-a842be60abd3.png)



# 参考

- [sklearn.metrics.plot_confusion_matrix](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.plot_confusion_matrix.html)
- [sklearn.metrics.ConfusionMatrixDisplay](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.ConfusionMatrixDisplay.html)
- [plot_confusion_matrix sample_code](https://scikit-learn.org/stable/auto_examples/model_selection/plot_confusion_matrix.html)
