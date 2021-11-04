# SIGNATE　<br>医学論文の自動仕分けチャレンジ
このレポジトリは、SIGNATE上で開催された[医学論文の自動仕分けチャレンジ](https://signate.jp/competitions/471)で使用したipynbファイルを保管するものです。<br>

## コンペティションの概要
上述のコンペティションのタスクは、論文のタイトルおよび抄録のテキストデータを用いて、システマティックレビューの対象となる文献か否か（2値）を判定するものとなっています。


具体的には、与えれたcsvファイルの"title","abstract"列を入力として、"judgement"列が正例か負例かを予測するものです。<br>
評価スコアは、FBetaScore(beta=7)により算出されます。

## ipynbの概要
1. [medical_eda.ipynb](medical_eda.ipynb) `train.csv`及び`test.csv`を分析し文字化けや欠損値の処理を行います。
2. [medical_bert_tf.ipynb](medical_bert_tf.ipynb) 1.で得られたデータを元に、公開されている数種のBERTを訓練します。
3. [medical_ensemble.ipynb](medical_ensemble.ipynb) 2.で得られた予測値を元に、アンサンブル手法を試行し提出ファイルを作成します。
4. [medical_summary.ipynb](medical_summary.ipynb) 入力文の要約を生成します。長文を圧縮する為に使用しましたが、効果は上がりませんでした。今後データ拡張法としての使用を検討しています。
## 実行環境
Google colabratoryでの実行を想定しています。

## 試行錯誤
### custom loss function
testデータの正例負例の個数をtrainデータから類推し、偽陰性(FN)と偽陽性(FP)の数を変数とするFBetaScore(beta=7)の関数を仮定しました。
例えば、正例98個、負例2個だとすると、`TP = 98 - FN`, `TF = 2 - FP`　より<br>
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


## スコア
|項目|説明|
|----|----|
|model|使用したBERTモデル|
|model structure|BERTの出力以降の構造 cf.)medical_bert_tf.ipynb|
|best monitor|採用モデルの選定指標|
|loss weight|損失関数に乗算する値 cf.)medical_bert_tf.ipynb|
|loss fn|使用した損失関数|
|temp threshold|訓練中使用する仮の閾値|
|cv score opt|oof全体で最適化した閾値で評価した交差検証スコア|
|cv score mean|各foldで最適化した閾値の平均値で評価した交差検証スコア|
|public score opt|oof全体で最適化した閾値で評価したリーダーボードのpublic score|
|public score mean|各foldで最適化した閾値の平均値で評価したリーダーボードのpublic score|
|private score opt|oof全体で最適化した閾値で評価したリーダーボードのprivate score|
|private score mean|各foldで最適化した閾値の平均値で評価したリーダーボードのprivate score|
|opt threshold|oof全体で最適化した閾値|
|mean threshold|各foldで最適化した閾値の平均値|


|model|model structure|best monitor|loss weight|loss fn|temp threshold|cv score<br>opt|cv score<br>mean|public score<br>opt|public score<br>mean|private score<br>opt|private score<br>mean|opt threshold|mean threshold|note|
| ----|----|----|----|----|----|----|----|----|----|----|----|----|----|----|
|microsoft/BiomedNLP-PubMedBERT-base-uncased-abstract-fulltext|logit|val_loss|1:50|weigted bce|0.0233|0.910|0.907|0.899|0.914|0.927|0.923|0.4537|0.3314||
|microsoft/BiomedNLP-PubMedBERT-base-uncased-abstract-fulltext|last hidden state - cnn|val_loss|1:50|weighted bce|0.1|0.905|0.904|0.906|0.903|0.931|0.923|0.428|0.262||
|microsoft/BiomedNLP-PubMedBERT-base-uncased-abstract-fulltext|4 layers [cls] concat|val_fbeta|class_weight="balanced"|bce|0.1|0.920|0.914|0.911|-|0.936|-|0.12|0.247|use summary insted of text|
|cambridgeltl/SapBERT-from-PubMedBERT-fulltext|single [cls]|val_fbeta|class_weight="balanced"|bce|0.0233|0.9182|0.9105|0.9024|0.9009|0.9185|0.9201|0.016|0.026||
|cambridgeltl/SapBERT-from-PubMedBERT-fulltext|single [cls]|val_loss|1:50|weigted bce|0.0233|0.9142|0.9151|0.8998|0.9052|0.9309|0.9279|0.471|0.3933||
|cambridgeltl/SapBERT-from-PubMedBERT-fulltext|logit|val_loss|1:50|weigted bce|0.0233|0.9155|0.9152|0.9045|0.8835|0.9336|0.9217|0.4335|0.6317||
|cambridgeltl/SapBERT-from-PubMedBERT-fulltext|single [cls]|val_auc|class_weight="balamced"|bce|0.0233|0.876|0.873|0.884|0.887|0.895|0.927|0.031|0.109||
|cambridgeltl/SapBERT-from-PubMedBERT-fulltext|single [cls]|val_loss|class_weight="balamced"|bce(label smooth=0.1)|0.015|0.904|0.9015|0.898|-|0.9127|-|0.079|0.105||
|cambridgeltl/SapBERT-from-PubMedBERT-fulltext|last hidden state - bilstm|val_loss|1:50|weighted bce|0.1|0.9108|0.9095|0.898|0.9013|0.9338|0.9338|0.424|0.407||
|cambridgeltl/SapBERT-from-PubMedBERT-fulltext|4 layers [cls] concat|val_loss|1:50|weighted bce|0.1|0.9184|0.9157|0.888|0.9016|0.9224|0.930|0.539|0.4315||
|cambridgeltl/SapBERT-from-PubMedBERT-fulltext|4 layers [cls] concat|val_fbeta|class_weight="balamced"|bce|0.1|0.9116|0.9106|0.910|0.906|0.9124|0.921|0.1012|0.1446|use summary insted of text|
|kamalkraj/bioelectra-base-discriminator-pubmed|logit|val_loss|1:50|weighted bce|0.0233|0.904|0.9005|0.9019|0.9028|0.9203|0.9209|0.539|0.5472||
|kamalkraj/bioelectra-base-discriminator-pubmed-pmc|4 layers [cls] concat|val_fbeta|class_weight="balamced"|bce|0.1|0.917|0.9104|0.898|-|0.9064|-|0.135|0.366|use summary insted of text|
|dmis-lab/biobert-base-cased-v1.2|4 layers [cls] concat|val_fbeta|class_weight="balamced"|bce|0.1|0.909|0.908|0.8989|-|0.9103|-|-|-|use summary insted of text|
|ensemble randomforest||val_fbeta||||0.9257||0.9186||0.9324|||||
|ensemble svm||val_fbeta||||0.9311||0.9099||0.9363|||||
|ensemble decision tree||val_fbeta||||0.9033||0.8994||0.9276|||||
## 最終リーダーボード順位
