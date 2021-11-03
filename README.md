# SIGNATE　医学論文の自動仕分けチャレンジ
このレポジトリは、SIGNATE上で開催された[医学論文の自動仕分けチャレンジ](https://signate.jp/competitions/471)で使用したipynbファイルを保管するものです。<br>

## コンペティションの概要
上述のコンペティションのタスクは、論文のタイトルおよび抄録のテキストデータを用いて、システマティックレビューの対象となる文献か否か（2値）を判定するものとなっています。


具体的には、与えれたcsvファイルの"title","abstract"列を入力として、"judgement"列が正例か負例かを予測するものです。<br>
評価スコアは、FBetaScore(beta=7)により算出されます。

## ipynbの概要
1. [medical_eda.ipynb](medical_eda.ipynb) `train.csv`及び`test.csv`を分析し文字化けや欠損値の処理を行います。
2. [medical_bert_tf.ipynb](medical_bert_tf.ipynb) 1.で得られたデータを元に、公開されている数種のBERTを訓練します。
3. [medical_ensemble.ipynb](medical_ensemble.ipynb) 2.で得られた予測値を元に、アンサンブル手法を試行し提出ファイルを作成します。
4. [medical_summary.ipynb](medical_summary.ipynb) 入力文の要約を生成します。長文を圧縮する為に使用しましたが、効果は上がりませんでした。今後データ拡張法としての使用を検証してみたいと考えています。
## 実行環境
Google colabratoryでの実行を想定しています。

## try & error
### custom loss function
testデータの正例負例の個数をtrainデータから類推し、偽陰性(FN)と偽陽性(FP)の数を変数とするFBetaScore(beta=7)の関数を仮定しました。
例えば、正例98個、負例2個だとすると、`TP = 98 - FN`, `TF = 2 - FP`　より
`recall = (98 - FN) / (98 - 2FN)`<br>
`precision = (98 - FN) / (98 - FN + FP)`<br>
`F7Score = 1 / {(0.98 / recall) + (0.02 / precision)}`<br>
となります。<br>
F7ScoreをFNとFPでそれぞれ偏微分し、その比を計算したところFNによる偏導関数の方が大体50倍大きな値となりました。<br>
実際に数値を入力した実験とも合致したので、スコアに対する影響度がFNの方が50倍大きいと仮定し、<br>
FNに対する損失がFPに対する損失より50倍大きくなるようなbinary crossentropy関数を自作しました。
### model
BERTの出力以降の構造を数種類試行しました。<br>
 - 最終４層分[cls]トークンを連結し線形変換するモデル
 - 最終層の後に１次元CNNを追加するモデル
 - 最終層の後に双方向LSTMを追加するモデル


## 交差検証スコア

## 最終リーダーボードスコア
