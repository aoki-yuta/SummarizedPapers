# Pluralistic_Image_Completion

[論文ページ](https://arxiv.org/abs/1903.04227)  


一言でいうと：

画像復元にバリエーションを持たせる手法．GANのジェネレータにCVAEを用いる．


## 著者

* Chuanxia Zheng
*  Tat-Jen Cham
* Jianfei Cai

## Abstract訳

多くの画像補完手法は、多くの合理的な可能性があるにもかかわらず、マスクされた入力に対して1つの結果しか得られない。この論文では、複数の画像補完のためのアプローチ、すなわち、画像補完のための複数かつ多様なもっともらしい解を生成する課題を提示する。学習ベースのアプローチが直面している大きな課題は、通常、ラベルごとに1つの基底真理学習インスタンスしかないことである。そのため、条件付きＶＡＥからのサンプリングでは、依然として最小限の多様性しか得られない。これを克服するために、我々は2つの並列パスを持つ、確率的に原理的に新しいフレームワークを提案する。一つは、与えられた唯一の基底真理を利用して欠落部分の事前分布を取得し、この分布から元の画像を再構築する再構築的なパスです。もう一つは、条件付き事前分布が再構成パスで得られた分布に結合された生成パスです。どちらもGANでサポートされています。また、デコーダとエンコーダの特徴の間の距離関係を利用して、外観の一貫性を向上させる新しい短期+長期注目層を導入する。建物(Paris)、顔(CelebA-HQ)、自然画像(ImageNet)のデータセットでテストしたところ、我々の手法はより高品質な補完結果を生成するだけでなく、複数の多様なもっともらしい出力を生成することができた。

## 何のための研究？
* 多様性を持った出力が可能な画像インペインティング→image completion
![](https://i.gyazo.com/b3d910dd12be4d820a0ebdb473755256.png)
## 従来のアプローチとその問題点は？

画像インペインティングは不良設定問題．本来自然に見える復元画像は複数あるが，従来の画像インペインティング（パッチベースの手法や機械学習ベースの手法）は一つの入力に対して一つの出力しか得ることができていなかった．

## この論文ではどういうアプローチで問題を解決する？
CVAE（Conditional Variational Auto Encoder）をGANのジェネレータとして組み込むことで多様性のあるインペインティングを実現している．
### 　ネットワーク
![network](https://i.gyazo.com/6c4d2e12e9ccfafb5b843da34385ed47.png)
* 二つのパスを持ったネットワークアーキテクチャ．
訓練のみに用いる非マスク→マスクのパス（reconstructiveパス）と訓練時，推論時両方用いるマスク→非マスクのパス(generativeパス)を持つ．エンコーダデコーダは重みを共有していて，違いは中間層inf1,inf2のresidual Blockである．
* 基本的にはVAEをベースにしている．エンコーダーによって潜在空間の平均と分散を求める．その分布に従うサンプルからデコーダで処理するが，Reparametrization Trickによってバックプロパゲーションを可能にしている．

### Short+Long Term Attention
デコーダーのself attentionに用いたAttention Mapをエンコーダーに対しても用いる？
ネットワークが状況に応じてエンコーダーのより細かい特徴に注目するか，デコーダーの特徴に注目するか決めている？
![](https://i.gyazo.com/ece82af8319a5bfd60c1c8d624c100a4.png)

## この論文の主定理，もしくは鍵となる式・図は？（最大2個まで）
![network](https://i.gyazo.com/6c4d2e12e9ccfafb5b843da34385ed47.png)
![](https://i.gyazo.com/58acf8b2c8c7bf894ef8256e6306a0f1.png)

## この論文で何が出来るようになった？and/or 何が良くなった？　
* 多様性を持った画像の生成が可能となった．

## 実験と評価方法

### 定量評価
この論文の立場では原画像は数ある正解画像の一つにすぎないため，定量評価は困難？
ディスクリミネータによる上位10サンプルのうち定性的なものも評価に入れて一つ選んだ．
既存のSOTA手法と比較．L1loss，PSNR，TVloss，[IS](https://papers.nips.cc/paper/2016/hash/8a3363abe792db2d8761d6403605aeb7-Abstract.html)
![](https://i.gyazo.com/efbe68644cf7faa5abcbb83dc56627d8.png)


### 定性評価
複数の画像で多様性のある画像生成が可能
![](https://i.gyazo.com/7a0cb94e0c6d8948f9af171d015442ad.png)

### 多様性の評価
[Learned Perceptual Image Patch Similarity(LPIPS)](https://arxiv.org/abs/1605.03557)を用いている．
* 生成画像間の平均特徴距離を計測
* 値が大きいほど多様性がある
![](https://i.gyazo.com/b5dff2ba51a02044938e5f6ddd55d582.png)


### Short+Long Term Attentionの評価
比較対象として[contextual attention(CA)](https://arxiv.org/abs/1801.07892)

Short+Long Term Attentionは上手く画面外から特徴を得ている．→エンコーダとデコーダーの両方の特徴量を適切に利用できている？
![](https://i.gyazo.com/bed78c96aa302aeac7d5311b4b51aa65.png)

## この論文の問題点や課題は？
* 別の人物に感じられるほどの変化はない（→[UCTGAN](https://openaccess.thecvf.com/content_CVPR_2020/html/Zhao_UCTGAN_Diverse_Image_Inpainting_Based_on_Unsupervised_Cross-Space_Translation_CVPR_2020_paper.html)）
* 人物や動物では整合性が低い？破綻していることがある
## 実装はある？
ある(PyTorch)

[Github](https://github.com/lyndonzheng/Pluralistic-Inpainting)