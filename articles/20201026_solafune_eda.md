---
title: "[solafune] 衛星画像から空港利用者数予測のEDA" # 記事のタイトル
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
画像コンペということなので学習用/テスト用/スコア評価 画像データに重複がないか調べました。調査時にはこちらのツールを主に利用しました。
[Github - idealo/imagededup](https://github.com/idealo/imagededup)

また調査時に作成したGoogleColabratory、JupyterNotebookは次のURLにて公開しました。以降の説明はすっとばして同じことを試したいという方は次のURLを見たほうが早いです。

[GoogleColabratory - image_similarity.ipynb](https://colab.research.google.com/drive/1EJnYEzrhX4ctc1kWlXm_xWRz3FxlnzhJ?usp=sharing)

## 画像類似度比較をする前準備

主にCSVファイルの読み込みや画像ZIPファイルの解凍をしています。`drive_path`については各自solafuneからDLしたデータセットの格納先が異なるので適当なパスを設定してください、同様に画像ZIPファイルの解凍時のコマンドも。

```python
!pip install -q imagededup

from google.colab import drive
drive.mount('/content/drive/')

from pathlib import Path

# 各自solafuneからDLしたデータセットの格納先へのパスを設定してください
drive_path = Path("solafune_data_path")

import pandas as pd

cols = ['file_name', 'user_count']
train_labels = pd.read_csv(drive_path / "traindataset_anotated.csv", header=None)
test_labels = pd.read_csv(drive_path / "testdataset_anotated.csv", header=None)
# headerがないので適当なheaderをセット
train_labels.columns = cols
test_labels.columns = cols

# 画像群zipファイルの解凍
!cp 'solafune_data_path/trainimage.zip' ./
!cp 'solafune_data_path/testimage.zip' ./
!cp 'solafune_data_path/evaluatemodel.zip' ./
!unzip -q trainimage.zip
!unzip -q testimage.zip
!unzip -q evaluatemodel.zip
```

## imagededupを利用した画像類似度比較①
画像の類似度はPerceptual Hash(PHash)用いて計算しました。PHashを指定した理由は特にありません、気になる方はアルゴリズムを変えて試してみてください。

下記のコードを実行すると、比較元の画像ファイルと比較先の類似度が閾値以上である画像ファイル群を表示します。この結果を見ると`trainimage/testimage/evaluatemodel`それぞれのフォルダ毎に類似画像ファイルがあることがわかります。

```python
from imagededup.methods import PHash
phasher = PHash()

encodings = phasher.encode_images(image_dir='image')
duplicates = phasher.find_duplicates(encoding_map=encodings)

for k, v in duplicates.items():
    if v:
        print(k, v)
```

出力結果ですがだいぶ長くなってしまったので折り畳んであります、表示する際は気をつけてください。

:::details 上記コードの出力結果(クリックすると展開されます)

```
train_14.jpg ['train_11.jpg', 'train_13.jpg', 'train_15.jpg', 'train_10.jpg', 'train_12.jpg']
train_38.jpg ['train_36.jpg', 'train_40.jpg', 'train_37.jpg', 'train_39.jpg']
comp_50.jpg ['comp_53.jpg', 'comp_48.jpg', 'comp_49.jpg']
train_27.jpg ['train_26.jpg', 'train_25.jpg']
comp_53.jpg ['comp_50.jpg', 'comp_48.jpg', 'comp_49.jpg']
comp_48.jpg ['comp_49.jpg', 'comp_53.jpg', 'comp_50.jpg', 'comp_51.jpg', 'comp_52.jpg']
comp_57.jpg ['comp_55.jpg']
test_10.jpg ['test_9.jpg']
train_23.jpg ['train_24.jpg', 'train_20.jpg', 'train_21.jpg', 'train_22.jpg']
comp_37.jpg ['comp_38.jpg']
train_52.jpg ['train_53.jpg', 'train_51.jpg']
train_57.jpg ['train_55.jpg']
train_8.jpg ['train_7.jpg', 'train_9.jpg', 'train_5.jpg']
train_15.jpg ['train_10.jpg', 'train_12.jpg', 'train_14.jpg', 'train_13.jpg', 'train_11.jpg']
test_14.jpg ['test_17.jpg']
train_50.jpg ['train_47.jpg', 'train_48.jpg']
comp_8.jpg ['comp_9.jpg', 'comp_7.jpg']
comp_38.jpg ['comp_37.jpg']
comp_17.jpg ['comp_18.jpg']
train_2.jpg ['train_0.jpg']
comp_10.jpg ['comp_15.jpg', 'comp_12.jpg', 'comp_13.jpg', 'comp_14.jpg', 'comp_11.jpg']
train_40.jpg ['train_38.jpg']
comp_9.jpg ['comp_8.jpg', 'comp_7.jpg']
train_37.jpg ['train_38.jpg']
comp_55.jpg ['comp_57.jpg', 'comp_56.jpg']
train_21.jpg ['train_23.jpg', 'train_24.jpg', 'train_22.jpg', 'train_20.jpg']
comp_18.jpg ['comp_17.jpg']
test_4.jpg ['test_1.jpg', 'test_2.jpg']
train_44.jpg ['train_46.jpg']
train_25.jpg ['train_26.jpg', 'train_27.jpg']
train_19.jpg ['train_18.jpg', 'train_17.jpg', 'train_16.jpg']
train_24.jpg ['train_23.jpg', 'train_20.jpg', 'train_21.jpg', 'train_22.jpg']
comp_51.jpg ['comp_52.jpg', 'comp_48.jpg', 'comp_49.jpg']
test_7.jpg ['test_6.jpg', 'test_5.jpg']
train_5.jpg ['train_8.jpg', 'train_7.jpg']
comp_24.jpg ['comp_21.jpg', 'comp_26.jpg']
train_48.jpg ['train_50.jpg']
train_53.jpg ['train_52.jpg', 'train_51.jpg']
comp_4.jpg ['comp_6.jpg', 'comp_5.jpg']
comp_15.jpg ['comp_14.jpg', 'comp_10.jpg', 'comp_13.jpg', 'comp_11.jpg', 'comp_12.jpg']
train_39.jpg ['train_38.jpg', 'train_36.jpg']
train_46.jpg ['train_45.jpg', 'train_44.jpg']
comp_49.jpg ['comp_48.jpg', 'comp_50.jpg', 'comp_53.jpg', 'comp_52.jpg', 'comp_51.jpg']
train_13.jpg ['train_11.jpg', 'train_14.jpg', 'train_15.jpg', 'train_10.jpg', 'train_12.jpg']
comp_3.jpg ['comp_2.jpg', 'comp_1.jpg', 'comp_0.jpg']
test_2.jpg ['test_4.jpg']
train_26.jpg ['train_27.jpg', 'train_25.jpg']
comp_26.jpg ['comp_24.jpg']
train_47.jpg ['train_50.jpg']
train_10.jpg ['train_12.jpg', 'train_15.jpg', 'train_14.jpg', 'train_13.jpg', 'train_11.jpg']
comp_21.jpg ['comp_24.jpg', 'comp_25.jpg']
train_22.jpg ['train_23.jpg', 'train_21.jpg', 'train_24.jpg', 'train_20.jpg']
comp_2.jpg ['comp_3.jpg', 'comp_1.jpg', 'comp_0.jpg']
train_36.jpg ['train_38.jpg', 'train_39.jpg']
comp_6.jpg ['comp_4.jpg', 'comp_5.jpg']
train_18.jpg ['train_17.jpg', 'train_19.jpg', 'train_16.jpg']
test_5.jpg ['test_6.jpg', 'test_7.jpg']
comp_11.jpg ['comp_12.jpg', 'comp_15.jpg', 'comp_14.jpg', 'comp_13.jpg', 'comp_10.jpg']
train_11.jpg ['train_14.jpg', 'train_13.jpg', 'train_15.jpg', 'train_10.jpg', 'train_12.jpg']
train_4.jpg ['train_0.jpg']
train_7.jpg ['train_8.jpg', 'train_9.jpg', 'train_5.jpg']
comp_12.jpg ['comp_10.jpg', 'comp_11.jpg', 'comp_14.jpg', 'comp_15.jpg', 'comp_13.jpg']
train_20.jpg ['train_23.jpg', 'train_24.jpg', 'train_21.jpg', 'train_22.jpg']
comp_14.jpg ['comp_15.jpg', 'comp_13.jpg', 'comp_12.jpg', 'comp_10.jpg', 'comp_11.jpg']
comp_13.jpg ['comp_14.jpg', 'comp_10.jpg', 'comp_15.jpg', 'comp_11.jpg', 'comp_12.jpg']
train_17.jpg ['train_18.jpg', 'train_19.jpg', 'train_16.jpg']
train_55.jpg ['train_57.jpg']
comp_56.jpg ['comp_55.jpg']
test_6.jpg ['test_7.jpg', 'test_5.jpg']
comp_0.jpg ['comp_3.jpg', 'comp_2.jpg']
train_12.jpg ['train_10.jpg', 'train_15.jpg', 'train_14.jpg', 'train_11.jpg', 'train_13.jpg']
comp_25.jpg ['comp_21.jpg']
train_0.jpg ['train_2.jpg', 'train_4.jpg']
test_17.jpg ['test_14.jpg', 'test_15.jpg']
test_15.jpg ['test_17.jpg']
train_45.jpg ['train_46.jpg']
comp_5.jpg ['comp_4.jpg', 'comp_6.jpg']
comp_52.jpg ['comp_51.jpg', 'comp_49.jpg', 'comp_48.jpg']
test_1.jpg ['test_4.jpg']
comp_1.jpg ['comp_2.jpg', 'comp_3.jpg']
train_16.jpg ['train_19.jpg', 'train_18.jpg', 'train_17.jpg']
train_9.jpg ['train_8.jpg', 'train_7.jpg']
train_51.jpg ['train_52.jpg', 'train_53.jpg']
test_9.jpg ['test_10.jpg']
comp_7.jpg ['comp_8.jpg', 'comp_9.jpg']
```

:::

## imagededupを利用した画像類似度比較②

またラベル付きである`train/testimage`について、類似画像ファイル名とラベルを並べて表示してみました。

この結果からおそらくは類似画像ファイルは同じ空港を撮影した画像なのではないかと予測できます。

```python
for k, v in duplicates.items():
    if v:
        filenames = v
        filenames.append(k)
        if k.startswith('train'):
            print(train_labels[train_labels.file_name.isin(filenames)])
        elif k.startswith('test'):
            print(test_labels[test_labels.file_name.isin(filenames)])
        else:
            # comp
            pass
```

出力結果ですがこちらも同様にだいぶ長くなってしまったので折り畳んであります、表示する際は気をつけてください。

:::details 上記コードの出力結果(クリックすると展開されます)

```
   file_name  user_count
10  train_10.jpg      909512
11  train_11.jpg      900021
12  train_12.jpg      872088
13  train_13.jpg      787645
14  train_14.jpg      795973
15  train_15.jpg      743852
       file_name  user_count
36  train_36.jpg      884467
37  train_37.jpg      870028
38  train_38.jpg      830431
39  train_39.jpg      758354
40  train_40.jpg      808441
       file_name  user_count
25  train_25.jpg      152164
26  train_26.jpg      124207
27  train_27.jpg      124207
      file_name  user_count
9    test_9.jpg     2366077
10  test_10.jpg     2381354
       file_name  user_count
20  train_20.jpg     2541446
21  train_21.jpg     2466061
22  train_22.jpg     2411634
23  train_23.jpg     2068766
24  train_24.jpg     2117984
       file_name  user_count
51  train_51.jpg     2964163
52  train_52.jpg     2706397
53  train_53.jpg     2330076
       file_name  user_count
55  train_55.jpg     2826277
57  train_57.jpg     1717102
     file_name  user_count
5  train_5.jpg     1497688
7  train_7.jpg     1335911
8  train_8.jpg     1193869
9  train_9.jpg     1273146
       file_name  user_count
10  train_10.jpg      909512
11  train_11.jpg      900021
12  train_12.jpg      872088
13  train_13.jpg      787645
14  train_14.jpg      795973
15  train_15.jpg      743852
      file_name  user_count
14  test_14.jpg    11617797
17  test_17.jpg     8636108
       file_name  user_count
47  train_47.jpg     1242760
48  train_48.jpg     1095224
50  train_50.jpg      941877
     file_name  user_count
0  train_0.jpg     3137184
2  train_2.jpg     3107076
       file_name  user_count
38  train_38.jpg      830431
40  train_40.jpg      808441
       file_name  user_count
37  train_37.jpg      870028
38  train_38.jpg      830431
       file_name  user_count
20  train_20.jpg     2541446
21  train_21.jpg     2466061
22  train_22.jpg     2411634
23  train_23.jpg     2068766
24  train_24.jpg     2117984
    file_name  user_count
1  test_1.jpg     5596210
2  test_2.jpg     5539453
4  test_4.jpg     4461208
       file_name  user_count
44  train_44.jpg      847546
46  train_46.jpg      692259
       file_name  user_count
25  train_25.jpg      152164
26  train_26.jpg      124207
27  train_27.jpg      124207
       file_name  user_count
16  train_16.jpg    66823414
17  train_17.jpg    61934302
18  train_18.jpg    64211074
19  train_19.jpg    62598351
       file_name  user_count
20  train_20.jpg     2541446
21  train_21.jpg     2466061
22  train_22.jpg     2411634
23  train_23.jpg     2068766
24  train_24.jpg     2117984
    file_name  user_count
5  test_5.jpg    17656262
6  test_6.jpg    16537566
7  test_7.jpg    16748180
     file_name  user_count
5  train_5.jpg     1497688
7  train_7.jpg     1335911
8  train_8.jpg     1193869
       file_name  user_count
48  train_48.jpg     1095224
50  train_50.jpg      941877
       file_name  user_count
51  train_51.jpg     2964163
52  train_52.jpg     2706397
53  train_53.jpg     2330076
       file_name  user_count
36  train_36.jpg      884467
38  train_38.jpg      830431
39  train_39.jpg      758354
       file_name  user_count
44  train_44.jpg      847546
45  train_45.jpg      782770
46  train_46.jpg      692259
       file_name  user_count
10  train_10.jpg      909512
11  train_11.jpg      900021
12  train_12.jpg      872088
13  train_13.jpg      787645
14  train_14.jpg      795973
15  train_15.jpg      743852
    file_name  user_count
2  test_2.jpg     5539453
4  test_4.jpg     4461208
       file_name  user_count
25  train_25.jpg      152164
26  train_26.jpg      124207
27  train_27.jpg      124207
       file_name  user_count
47  train_47.jpg     1242760
50  train_50.jpg      941877
       file_name  user_count
10  train_10.jpg      909512
11  train_11.jpg      900021
12  train_12.jpg      872088
13  train_13.jpg      787645
14  train_14.jpg      795973
15  train_15.jpg      743852
       file_name  user_count
20  train_20.jpg     2541446
21  train_21.jpg     2466061
22  train_22.jpg     2411634
23  train_23.jpg     2068766
24  train_24.jpg     2117984
       file_name  user_count
36  train_36.jpg      884467
38  train_38.jpg      830431
39  train_39.jpg      758354
       file_name  user_count
16  train_16.jpg    66823414
17  train_17.jpg    61934302
18  train_18.jpg    64211074
19  train_19.jpg    62598351
    file_name  user_count
5  test_5.jpg    17656262
6  test_6.jpg    16537566
7  test_7.jpg    16748180
       file_name  user_count
10  train_10.jpg      909512
11  train_11.jpg      900021
12  train_12.jpg      872088
13  train_13.jpg      787645
14  train_14.jpg      795973
15  train_15.jpg      743852
     file_name  user_count
0  train_0.jpg     3137184
4  train_4.jpg     2816245
     file_name  user_count
5  train_5.jpg     1497688
7  train_7.jpg     1335911
8  train_8.jpg     1193869
9  train_9.jpg     1273146
       file_name  user_count
20  train_20.jpg     2541446
21  train_21.jpg     2466061
22  train_22.jpg     2411634
23  train_23.jpg     2068766
24  train_24.jpg     2117984
       file_name  user_count
16  train_16.jpg    66823414
17  train_17.jpg    61934302
18  train_18.jpg    64211074
19  train_19.jpg    62598351
       file_name  user_count
55  train_55.jpg     2826277
57  train_57.jpg     1717102
    file_name  user_count
5  test_5.jpg    17656262
6  test_6.jpg    16537566
7  test_7.jpg    16748180
       file_name  user_count
10  train_10.jpg      909512
11  train_11.jpg      900021
12  train_12.jpg      872088
13  train_13.jpg      787645
14  train_14.jpg      795973
15  train_15.jpg      743852
     file_name  user_count
0  train_0.jpg     3137184
2  train_2.jpg     3107076
4  train_4.jpg     2816245
      file_name  user_count
14  test_14.jpg    11617797
15  test_15.jpg    10994749
17  test_17.jpg     8636108
      file_name  user_count
15  test_15.jpg    10994749
17  test_17.jpg     8636108
       file_name  user_count
45  train_45.jpg      782770
46  train_46.jpg      692259
    file_name  user_count
1  test_1.jpg     5596210
4  test_4.jpg     4461208
       file_name  user_count
16  train_16.jpg    66823414
17  train_17.jpg    61934302
18  train_18.jpg    64211074
19  train_19.jpg    62598351
     file_name  user_count
7  train_7.jpg     1335911
8  train_8.jpg     1193869
9  train_9.jpg     1273146
       file_name  user_count
51  train_51.jpg     2964163
52  train_52.jpg     2706397
53  train_53.jpg     2330076
      file_name  user_count
9    test_9.jpg     2366077
10  test_10.jpg     2381354
```

:::

## imagededupを利用した画像類似度比較③

最後に実際の類似画像を表示してみます。結果を載せたかったのですが上記の結果同様、大量の画像ファイルを貼り付けることになるので断念しました。結果は[GoogleColabratory - image_similarity.ipynb](https://colab.research.google.com/drive/1EJnYEzrhX4ctc1kWlXm_xWRz3FxlnzhJ?usp=sharing)を参照してください。

出力結果とひとつ前の結果から類似画像は同じ空港を撮影したものであり、撮影時期や撮影した時間帯が異なる画像であることがわかります。

```python
from imagededup.utils import plot_duplicates

for k, v in duplicates.items():
    if v:
        plot_duplicates(image_dir='image', 
                    duplicate_map=duplicates, 
                    filename=k)
```


# まとめ
- `trainimage.zip` / `testimage.zip` / `evaluatemodel.zip`各フォルダ毎に類似画像が存在する
- それら類似画像は同じ空港を撮影したものであり、おそらく撮影時期や時間帯が異なると思われる

上記結果を受けて言えることとして
- 訓練データと検証データを分割する際、類似画像が含まれることを考慮して分割する必要がある

以上になります、最後まで読んでいただきありがとうございました。

# 参考
- [solafune - 衛星画像から空港利用者数を予測 ルール](https://solafune.com/#/competitions/ea90cba4-3e01-42df-9516-9ac0d7a44204)
- [Github - idealo/imagededup](https://github.com/idealo/imagededup)
