# Image Outpainting and Harmonization using Generative Adversarial Networks

[論文PDF](https://arxiv.org/abs/1912.10960)  

一言でいうと：  
inpaintingの逆のoutpaintingのタスクをContext EncoderベースのGANを用いて解決する．精度を上げるために，SinGANを用いてるのがおもしろい．

## 著者
- Basile Van Hoorick

## Abstract訳
画像の四辺の先に何があるのかを予測するという本質的に曖昧な課題はこれまでほとんど研究されてきませんでしたが、GANが合理的な外挿を生成するために強力な可能性を秘めていることを示します。1つ目のアプローチでは、一般的な描画アーキテクチャとパラダイムにインスパイアされたコンテキストエンコーダを使用し、2つ目のアプローチでは、単一画像生成モデルを使用した後処理ステップを追加しています。この方法では、ボケたディティールが元の画像のスタイルと統合され、結果の品質をさらに向上させ、任意の出力解像度をサポートすることができます。

## 何のための研究？
Outpainting．inpaintingは画像の内部の欠損領域の補間であったが，outpaintingは画像の外側の領域の予測と生成．
![idea](https://github.com/basilevh/image-outpainting/raw/master/idea.png)
## 従来のアプローチとその問題点は？
従来outpaintingについての研究は少ない．しかし，多くの部分でInpaintingと共通のモデルやパーツを利用できる．
### 従来手法
#### アプローチ１：ガイドを用いた幾何学的な補間
[CVPR2013][FrameBreak: Dramatic Image Extrapolation by Guided Shift-Maps](https://www.cv-foundation.org/openaccess/content_cvpr_2013/html/Zhang_FrameBreak_Dramatic_Image_2013_CVPR_paper.html)
対象となる画像と似ているガイド画像（より広範囲を写したもの）を使って，対象となる画像とガイド画像の類似点を探して幾何学的に補間していく方法．
![image](https://i.gyazo.com/b2c3ffeb20e0df1d4ccb934c77b3208f.png)
##### 問題点
* 都合の良いガイド画像を用意できないケースが多い

#### アプローチ２：GANを用いたOutpainting
[Painting Outside the Box: Image Outpainting with GANs](https://arxiv.org/abs/1808.08483)
Iizuka氏の[GLCIC](http://iizuka.cs.tsukuba.ac.jp/projects/completion/ja/)をベースにした水平方向のoutpaintingで，大域的整合性と局所的整合性を持った復元を行う．
![result](https://i.gyazo.com/058252f2d70842137a84e51aeb7f562d.png)

##### 問題点
*  水平方向のみの検証で，垂直方向への検証がされていない．
* ディスクリミネータに大域的整合性ネットワークと局所的整合性ネットワークを用いて学習すると，大域的整合性ネットワークのみで学習した時より，RSMEが上がる一方でアーチファクトを生成する．
![issue](https://i.gyazo.com/b119580ac42a2d46f85244472cd54e2e.png)

#### アプローチ３:encoder-decoderによるoutpainting
[CVPR2019][Wide-Context Semantic Image Extrapolation](http://openaccess.thecvf.com/content_CVPR_2019/html/Wang_Wide-Context_Semantic_Image_Extrapolation_CVPR_2019_paper.html)

![result](https://i.gyazo.com/7ede4336534733b05b84147ba7f668d8.png)

encoder-decoderモデルを用いて小さい画像から特徴量を抽出するネットワークと特徴量から大きい画像を復元するネットワークの二つを学習する．画像の一部のみから画像全体を復元することが可能

![framework](https://i.gyazo.com/f1e3b819f87dc4885f9e74a9baaa4e50.png)

##### 問題点
* 復元できる画像は学習したドメインに依存する．CelebAのように多くの同じシーンのデータセットから学習した際に同じドメインの画像復元はうまくいくが，Places2のような様々なシーンに対応できない．

## この論文ではどういうアプローチで問題を解決する？
ステップは大きく二つに分けられる．
1. ContextEncoderベースのOutpaintingステップ
2. SinGANを用いたHarmonizationステップ

### 1.Outpaintingステップ
* GANを用いてクロップされた画像から元の画像を復元するなネットワークを学習する．
* 基本的にはInpaindingで持ちいられる[Context Encoder](https://arxiv.org/abs/1604.07379)をベースにしている．
* 論文ではデータセットに数百万枚の様々なシーンを含む[Place365-standard](http://places2.csail.mit.edu/download.html)と，多くのイラストを含む[WikiArt](https://github.com/cs-chan/ArtGAN/tree/master/WikiArt%20Dataset)を用いて実験される．まずデータセットに含まれる画像を192×192にリサイズしたものをGTとする．それを上下左右均等にクロップして128×128にしたものを入力として用いる．

![Architecture](https://i.gyazo.com/27120f5b9e268a4c55eee169573962c5.png)

#### 損失関数
Reconstruction LossにL1を使うことでよりボケの少ない画像が生成できる（らしい）

![loss](https://i.gyazo.com/7ab52113ce1fa33566a10faff729572d.png)

ここで![lambda](https://i.gyazo.com/6336af4e098a1f3c9980c273d32f083c.png)である．

#### Generater
|                | **入力**                                            | **出力**       |
| -------------- | --------------------------------------------------- | -------------- |
| **解像度**     | 192×192                                             | 192×192        |
| **コンテンツ** | 真ん中に128×128にクロップされた画像（周りは０埋め） | 復元された画像 |

全部で１２層のネットワークで，最初の６層(encoder)でダウンサンプリングを行い，特徴量を抽出．後ろの６層（Decoder） でも復元する．Inpaintingの場合，　入力がは画像全体で，復元されるのは欠損領域のみで，入力と出力の解像度が異なるが，Outpaintingの場合は入力と出力の解像度が同じであるため，エンコーダとデコーダはほぼ鏡写しの形状になる．
#### Discriminater
InpaindingのContext Encoderは（欠損部分に似せて）生成された画像とその部分の現画像の一部のみで学習を行うが，この手法では周りとの整合性を確保するために復元された画像全体をDiscriminaterで見ている．

### 2.Harmonizationステップ
#### アプローチ
１.Outpaintingステップで生成された画像の外側の領域は曖昧であったり，ボケていたりする．これをシャープにするために，[SinGAN](https://webee.technion.ac.il/people/tomermic/SinGAN/SinGAN.htm)のHarmonizationを用いる．SinGANは単一画像からピラミッド状のGANで画像の特徴を学習する．SinGANのアーキテクチャは以下の通り
![SinGAN_Architecture](https://i.gyazo.com/4749b3a4ce48b06d26032d608ef9bb43.png)
SinGANの各層は粗い画像を細かい画像にエンコードする一種のSuper-resolusionと考えられる．ここで，予め元の画像でSinGANを学習しておき，1.outpaintingの結果画像をSinGANのピラミッドの途中（粗いスケール）に挿入することで，細かい部分に元の画像と同じスタイルを適用することができる．この操作は結果的にHarmonizationとSuper-resolusionを同時に行う事になる．その結果ボケていた部分の解像度が上がり，精度がよくなる．

#### 結果
1.OutpaintingステップでrecLossとadvLossを用いて学習したものよりも，1.Outpaintingステップで，advLossを用いず，recLossのみを用いて，SinGANによるHarmonizeを行った方がグリッチが存在せず，綺麗に見える．SinGANで出力の解像度を自由にできる．(実際には最大解像度はグラボの性能に依存する)
![result](https://i.gyazo.com/55710f1cad28bc74d0e34d49851088ff.png)
．→SinGANを使えばOutpainting時にGANを使わないで良い？

## この論文の主定理，もしくは鍵となる式・図は？（最大2個まで）

![Architecture](https://i.gyazo.com/27120f5b9e268a4c55eee169573962c5.png)

![result](https://i.gyazo.com/55710f1cad28bc74d0e34d49851088ff.png)


## この論文で何が出来るようになった？and/or 何が良くなった？　
* 全方向のドメイン依存の少ないOutpainting
* 繰り返しによる再起的なoutpainting
![](https://i.gyazo.com/eb72845e3381d9a7d4b853c8642f8784.png)


* SinGANのHarmonizationを用いたアイデアはinpaintingなどの他のタスクで活かせそう


## 実験と評価方法
他の手法との比較はない．以下の３つのパターンで平均のMSE（G(x),y）とD(G(x)) を比較している．
* Place365データセットの内20万枚で![1](https://i.gyazo.com/03bff81ac9a5044334d1dc5795b9e537.png)で学習．
* Place365データセットの内20万枚で![2](https://i.gyazo.com/65ef7df5acc1f1376d98a240285005d7.png)で学習．
* WikiArtデータセット内の5万枚で![3](https://i.gyazo.com/65ef7df5acc1f1376d98a240285005d7.png)で学習．

結果
![](https://i.gyazo.com/bd4f936ccafb1356e8a7844e2726d1f7.png)
![](https://i.gyazo.com/2edc3c08120bf10e1326c4a71ca9d661.png)
Place365よりWikiArtの方がMSEは低いものの，D(G(x))は高い．これはWikiArtの方がDiscriminaterで真偽を判断することが難しいことを示している．
MSEをみると，recのみの方が良いように見えるが，実際に生成された画像をみると，recのみの画像の補間箇所ははボケてしまっているように見える．一方でrec+advの補間箇所はシャープであるものの，グリッチが発生しているのがわかる．トレードオフの関係である．

* Harmonizationについての定量的な評価はなし．


## この論文の問題点や課題は？
* 画像によって復元した領域がボケたり暗くなったりする．
* SinGANによる調和は学習に１時間かかる．


## 実装はある？
[実装(GitHub)](https://github.com/basilevh/image-outpainting)  
ある．pytorchでの実装．SinGANによるHarmonizationのステップは（手動でやったらしいので）含まれていない．
