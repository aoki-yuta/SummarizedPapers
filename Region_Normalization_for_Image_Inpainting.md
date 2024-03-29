# Region Normalization for Image Inpainting

[論文ページ](https://arxiv.org/abs/1911.10375)  

一言でいうと：  
マスクを考慮したノーマライゼーション．一部が255とかの時，平均，分散がシフトしてしまうのを防ぐ．


## 著者
* Tao Yu
* Zongyu Guo
* Xin Jin
* Shilin Wu
* Zhibo Chen
* Weiping Li
* Zhizheng Zhang
* Sen Liu

## Abstract訳
特徴正規化（FN）は、ニューラルネットワークの学習を支援する重要な技術であり、一般に空間次元に渡って特徴を正規化する。これまでの画像修復手法の多くは、入力画像の破損領域が正規化に与える影響（例えば、平均や分散のシフトなど）を考慮せずに、ネットワークにFNを適用している。本研究では、全空間FNによる平均と分散のシフトが画像インペインティングネットワークの学習を制限することを示し、その制限を克服するために、領域正規化（RN）と名付けた空間領域単位の正規化を提案する。RNは入力マスクに従って空間画素を異なる領域に分割し、各領域の平均と分散を計算して正規化を行う。本論文では、2種類の領域正規化を開発し、画像処理ネットワークに適用した。(1) 基本RN(RN-B)は、平均と分散のずれを解決するために、元の塗りつぶしマスクに基づいて、塗りつぶし領域と非塗りつぶし領域の画素を別々に正規化する。 (2) 学習可能RN (RN-L) は、別々の正規化のために潜在的に塗りつぶし領域と非塗りつぶし領域を自動的に検出し、それらの融合性を高めるためにグローバルアフィン変換を実行する。RN-Bはネットワークの初期層に、RN-Lはネットワークの後半層にそれぞれ適用する。実験により，本手法は現在の最先端手法を量的にも質的にも凌駕することが示された．さらに、RNを他のインペインティングネットワークに一般化し、一貫した性能向上を達成する。

## 何のための研究？
* 入力画像の破損を考慮したノーマライズ
![](https://i.gyazo.com/d8c84f41ebf8eacc217e01cecfaf3208.png)

## 従来のアプローチとその問題点は？
### 従来のアプローチ
従来のノーマライゼーション手法は，欠損領域を考慮していない．  
提案しているのはインスタンスノルムの領域分割ver

![](https://i.gyazo.com/e887969d3cf173c1bf7f35e52c19b43c.jpg)

### 問題点
F2は欠損領域まで含めてノーマライズ，F3は別々にノーマライズ．（欠損領域は0）
F2もF3も，平均0分散1であるが，その実態は異なる．
(b)を見ると，F2は欠損領域のせいでノーマライズ時に全体が左にシフトしている．この問題点は，例えばReLUに通す時，ほとんどが0以下の非線形部分にいて，勾配消失を起こしやすい，．一方F3は全体はシフトしないため，そのような問題は起こりづらい．
![](https://i.gyazo.com/341c00badc80b6b08df4a75f2f11cda0.png)

## この論文ではどういうアプローチで問題を解決する？
RN-BとRN-L２つの手法が提案されている  
![](https://i.gyazo.com/ef71673498f052b974acb3dead6c1aa7.png)

### RN-B
あらかじめマスクを与えられ，空間方向で，欠損部と非欠損部それぞれで平均と分散を計算する．
（バッチ方向とチャンネル方向は計算しないことに注意）．ネットワークの序盤向き
### RN-L
マスクをネットワークで学習する．
普通の畳み込みを何層か行うと，欠損領域は周辺に滲んでじまう．そこで学習を用いて欠損領域(が影響を与える部分)を推論することで解決する．ネットワークの終盤向き
→p-convがあればマスクを得られるので不要？


## この論文の主定理，もしくは鍵となる式・図は？（最大2個まで）

### 1 問題点．統計量のシフト
問題点参照

![](https://i.gyazo.com/341c00badc80b6b08df4a75f2f11cda0.png)
### 2 具体的な計算式
K=2が一般的な欠損．K=1の時インスタンスノルム
仰々しいが，領域ごとに統計量を計算しているだけ

![](https://i.gyazo.com/5e66bb60f1feebf1e7eca6c582f64cb8.png)

## この論文で何が出来るようになった？and/or 何が良くなった？　
* 欠損領域を考慮して特徴量の正規化を行うため，勾配消失が起こりづらい


## 実験と評価方法

### ネットワーク
[EdgeConnect](https://arxiv.org/abs/1901.00212)のINをRNに置き換え.前半はRN-B，後半はRN-L  
![](https://i.gyazo.com/8380065d6c77d97b7334ecc797e39553.png)  
↓↓↓↓↓
![](https://i.gyazo.com/26978ce73829b46708e3d2b629f932dc.png)

### 問題
* Place2とCelebAのデータセット
* 中央1/4マスクと不規則マスク([p-conv](https://arxiv.org/abs/1804.07723))
不規則マスクは全部で12000個．
0-10, 10-20, ... 50-60％のそれぞれ2000個づつ



### 比較手法
* CA: Contextual Attention (Yu et al. 2018).
* PC: Partial Convolution (Liu et al. 2018).
* GC: Gated Convolution (Yu et al. 2019).
* EC: EdgeConnect (Nazeri et al. 2019).
* Baseline:ネットワークのRNの部分を全てINにしたもの

### 定量評価
PSNR, SSIM, L1  
![](https://i.gyazo.com/a9dd309f6bc9fa70e4f1fa577a718f28.png)

### 定性評価
主観評価のみ

![](https://i.gyazo.com/c4f0a4e97a64e2b00246718c0b9bddb7.png)

## この論文の問題点や課題は？
* バッチ方向には統計量をとっていないので，BNの拡張ではない．(INの拡張)

## 実装はある？
ある（Pytorch）

[Github](https://github.com/geekyutao/RN)
