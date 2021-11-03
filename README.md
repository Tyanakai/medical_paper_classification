# SIGNATE　医学論文の自動仕分けチャレンジ
このレポジトリは、SIGNATE上で開催された[医学論文の自動仕分けチャレンジ](https://signate.jp/competitions/471)で使用したipynbファイルを保管するものです。<br>

## コンペティションの概要
上述のコンペティションのタスクは、論文のタイトルおよび抄録のテキストデータを用いて、システマティックレビューの対象となる文献か否か（2値）を判定するものとなっています。


具体的には、与えれたcsvファイルの"title","abstract"列を入力として、"judgement"列が正例か負例かを予測するものです。<br>
評価スコアは、FBetaScore(beta=7)により算出されます。

## ipynbの概要
1. [medical_eda.ipynb](medical_eda.ipynb) : 訓練データtrain.csv、及び検証データtest.csvを分析し文字化けや欠損値の処理を行います。
2. [medical_bert_tf.ipynb](medical_bert_tf.ipynb) : 1.で得られたデータを元に、公開されている数種のBERTを訓練します。
3. [medical_ensemble.ipynb](medical_ensemble.ipynb) : 2.で得られた予測値を元に、アンサンブル手法を試行し、提出ファイルを作成します。
4. [medical_summary.ipynb](medical_summary.ipynb) : 入力文の要約文を生成します。Data augmentの為に作成しましたが、今回は使用しませんでした。

## 実行環境
Google colabratoryでの実行を想定しています。

## try & error

## 交差検証スコア

## 最終リーダーボードスコア
