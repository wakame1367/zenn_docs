---
title: "sklearn.metrics.classification_reportの出力結果をCSVファイルで出力する" # 記事のタイトル
emoji: "😸" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["Python", "scikit-learn"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

# なんで書いたのか

お仕事の都合で[sklearn.metrics.classification_report](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.classification_report.html)の出力結果をレポートに張り付けたかったのでコードを書きました。

# サンプルコード

[sklearn.metrics.classification_report](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.classification_report.html)はデフォルトで文字列を出力するのみになっておりますが、引数の`output_dict=True`に設定することで辞書化した文字列を返すようになります。辞書は構造化されているので`pandas.DataFrame`に取り込んでCSVファイルとして出力すればやりたかったことはできました。

```python
import numpy as np
import pandas as pd
from sklearn.metrics import classification_report

# https://scikitlearn.org/stable/modules/generated/sklearn.metrics.classification_report.html
y_true = np.array([0, 1, 2, 2, 2])
y_pred = np.array([0, 0, 2, 2, 1])
target_names = ['class 0', 'class 1', 'class 2']

report = classification_report(y_pred=y_pred, y_true=y_true,
                               target_names=target_names, output_dict=True)

# 転置しているのはmetrics名をheaderにするため
report_df = pd.DataFrame(report).T
report_df
"""
	precision	recall	f1-score	support
class 0	0.5	1.000000	0.666667	1.0
class 1	0.0	0.000000	0.000000	1.0
class 2	1.0	0.666667	0.800000	3.0
accuracy	0.6	0.600000	0.600000	0.6
macro avg	0.5	0.555556	0.488889	5.0
weighted avg	0.7	0.600000	0.613333	5.0
"""

# index=Falseにするとラベル名が消えてしまうので注意
report_df.to_csv("report.csv")

pd.read_csv("report.csv", index_col=0)
"""
	precision	recall	f1-score	support
class 0	0.5	1.000000	0.666667	1.0
class 1	0.0	0.000000	0.000000	1.0
class 2	1.0	0.666667	0.800000	3.0
accuracy	0.6	0.600000	0.600000	0.6
macro avg	0.5	0.555556	0.488889	5.0
weighted avg	0.7	0.600000	0.613333	5.0
"""
```

# 参考
- [scikit-learn - sklearn.metrics.classification_report](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.classification_report.html)
