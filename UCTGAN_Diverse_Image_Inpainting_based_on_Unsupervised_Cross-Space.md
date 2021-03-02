# UCTGAN: Diverse Image Inpainting based on Unsupervised Cross-Space

[論文ページ](https://openaccess.thecvf.com/content_CVPR_2020/html/Zhao_UCTGAN_Diverse_Image_Inpainting_Based_on_Unsupervised_Cross-Space_Translation_CVPR_2020_paper.html)  


一言でいうと：  


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


## 何のための研究？
与えられた欠損画像に対して多種多様なインペインティングを行うタスク．【diverse image Inpainting】の提案．
今までは一つの欠損画像に対して一つの復元結果であったが，一つの欠損画像に対して複数の，整合性のある画像を生成できる．
![idea](https://i.gyazo.com/71f36b76a3605c359b05df53ab970ddf.png)

## 従来のアプローチとその問題点は？
### 従来手法
#### アプローチ１：パッチベース手法

##### 問題点
* 基本はコピペなので新たなオブジェクトの生成ができない．

#### アプローチ２：学習ベースのアプローチ

##### 問題点
*  

#### アプローチ３:ああああ

##### 問題点
* 復

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
