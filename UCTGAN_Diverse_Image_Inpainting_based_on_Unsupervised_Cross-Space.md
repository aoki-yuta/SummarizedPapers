# UCTGAN: Diverse Image Inpainting based on Unsupervised Cross-Space

[論文ページ](https://openaccess.thecvf.com/content_CVPR_2020/html/Zhao_UCTGAN_Diverse_Image_Inpainting_Based_on_Unsupervised_Cross-Space_Translation_CVPR_2020_paper.html)  


一言でいうと：

画像復元にバリエーションを持たせる．データセットの一枚から復元画像へと変換するネットワークを教師なしで学習？


## 著者
- Lei Zhao
- Qihang Mo
- Sihuan Lin
- Zhizhong Wang
- Zhiwen Zuo
- Haibo Chen
- Wei Xing
- Dongming Lu

## Abstract訳

既存の画像塗り潰し手法は、視覚的にリアルで意味的に正しい結果を得ることができたが、マスクされた入力に対して1つの結果しか得られなかった。そこで、本研究では、条件付きエンコーダモジュール、マニホールド投影モジュール、生成モジュールの3つのネットワークモジュールを中心に構成された、教師なし異空間変換生成逆襲ネットワーク（Unsupervised Cross-space Translation Generative Adversarial Network: UCTGAN）を提案している。マニホールド投影モジュールと生成モジュールを組み合わせることで、インスタンス画像空間と条件補完画像空間を共通の低次元マニホールド空間に投影することで、教師なしで2つの空間間の1対1の画像マッピングを学習することができ、再サンプルの多様性を大幅に向上させることができます。また、大域的な情報を理解するために、既知部分と完成部分との間の長距離依存性を利用した新しいクロスセマンティック注意層を導入し、修復されたサンプルのリアリズムと外観の一貫性を向上させることができる。CelebA-HQ, Places2, Paris Street View, ImageNetなどの様々なデータセットを用いた広範な実験により、我々の手法が、修復される同一画像から多様なインペイントソリューションを生成するだけでなく、高品質な画像を実現することが明らかになった。

## 何のための研究？
* 一般に画像インペインティングは不良設定問題．であれば復元画像を複数提案できるほうがよい．

* 与えられた欠損画像に対して多種多様なインペインティングを行うタスク．【diverse image Inpainting】の提案．

* 今までは一つの欠損画像に対して一つの復元結果であったが，一つの欠損画像に対して複数の，整合性のある画像を生成できる．
![idea](https://i.gyazo.com/71f36b76a3605c359b05df53ab970ddf.png)

## 従来のアプローチとその問題点は？

### アプローチ１：背景から持ってくる（パッチベース手法）
[1](https://gfx.cs.princeton.edu/pubs/Barnes_2009_PAR/)
[21](http://www.olivier-augereau.com/docs/2004JGraphToolsTelea.pdf)
[2](https://www.math.ucla.edu/~lvese/PAPERS/01217265.pdf)


#### 問題点
* 基本はコピペなので新たなオブジェクトの生成ができない．
* 意味のある（大域的整合性のある）画像の生成ができない

### アプローチ２：学習ベースのアプローチ(non-gan)

#### 問題点
* 最適な解１つしか出力できない（多様性がない）

### アプローチ３:GANベースのアプローチ

[6](https://papers.nips.cc/paper/2014/hash/5ca3e9b122f61f8f06494c97b1afccf3-Abstract.html)
[19](https://papers.nips.cc/paper/2015/hash/8d55a249e6baa5c06772297520da2051-Abstract.html)
[9](https://arxiv.org/abs/1710.10196)
[3](https://arxiv.org/abs/1809.11096)

GANベースのアプローチでは以下の二つの理由からdiverse image Inpaintingには直接用いれない．

#### 問題点
1. 訓練時マスク画像に対して正解画像が一つである．（バリエーションの条件をつけられない）
条件付き確率分布を明示的に表現するデータセットがない．（？）
2. diverse image Inpaintingには強い制約（周辺との整合性）があるため，モード崩壊（同じ画像を出してしまう）の影響を受けやすい

## この論文ではどういうアプローチで問題を解決する？
ステップは大きく二つに分けられる．
1. ContextEncoderベースのOutpaintingステップ
2. SinGANを用いたHarmonizationステップ

## この論文の主定理，もしくは鍵となる式・図は？（最大2個まで）


## この論文で何が出来るようになった？and/or 何が良くなった？　
*
* SinGANのHarmonizationを用いたアイデアはinpaintingなどの他のタスクで活かせそう


## 実験と評価方法



## この論文の問題点や課題は？



## 実装はある？
