# coherent semantic attention for image inpainting


[論文ページ](https://arxiv.org/abs/1905.12384)  


一言でいうと：
パッチ同士の相関を考慮したCoherent Semantic Attention layer(CSA)を提案．CSAの性能向上のためにconsistency lossやfeature patch discriminatorを導入．

## 著者

* Hongyu Liu 
* Bin Jiang 
* Yi Xiao 
* Chao Yang

## Abstract訳
最新のディープラーニングベースのアプローチは、画像の欠落領域を塗り潰すという困難な課題に対して有望な結果を示している。しかし，既存の手法では，局所的な画素の不連続性のために，ぼやけたテクスチャや歪んだ構造を持つコンテンツが生成されることが多い．意味レベルの観点から見ると、これらの手法は主に穴領域の意味的関連性や特徴の連続性を無視しているため、局所画素の不連続性が原因である。この問題に対処するために、我々は写真の修復における人間の行動を調査し、新しいコヒーレントセマンティックアテンション(CSA)層を用いた有限ディープ生成モデルベースのアプローチを提案している。タスクをラフに分割し、2つのステップとして絞り込みを行い、各ステップをU-Netアーキテクチャのニューラルネットワークでモデル化し、絞り込みステップのエンコーダにCSA層を埋め込む。ネットワークの学習プロセスを安定化させ、CSA層のより効果的なパラメータ学習を促進するために、デコーダ内のCSA層と対応するCSA層が同時に基底真実画像のVGG特徴層に近づくように強制するための一貫性損失を提案している。CelebA, Places2, Paris StreetViewデータセットを用いた実験により、提案手法の画像インペイントタスクにおける有効性が検証され、既存の最先端のアプローチと比較して、より高品質な画像を得ることができる。

## 何のための研究？
* ピクセル同士の相関関係を考慮した画像インペインティング

## 従来のアプローチとその問題点は？
### パッチベース
周辺から似ているパッチをコピペ
#### 問題点
大きな欠損があるとき大域的整合性（意味）の反映が難しい．
### 学習ベース(GAN)
#### 1.[context-encoder](https://arxiv.org/abs/1604.07379)
##### 問題点
テクスチャ生成に不向き
#### 2.[GLCIC](http://iizuka.cs.tsukuba.ac.jp/projects/completion/ja/)
##### 問題点
穴付近の整合性のために後処理が必要
#### 3.[High-Resolution Image Inpainting using Multi-Scale Neural Patch Synthesis](https://arxiv.org/abs/1611.09969)
#### 4.[partialconv](https://arxiv.org/abs/1804.07723)，[Gated Conv](https://arxiv.org/abs/1806.03589)
##### 問題点
特徴量の相関関係を明示的に考慮していないため，生成画像に色の不一致がおこる→（提案手法）

## この論文ではどういうアプローチで問題を解決する？
最初にラフに補完するネットワークで補完して，パッチ間の類似度を用いてマスク部分を徐々に綺麗にしていく．
### Coherent Semantic Attention
注目したパッチとマスク外の最も類似度の高いパッチとの類似度()と注目したパッチとマスク領域内の一つ前のパッチの類似度（）の二つを用い，類似度の比率でパッチを合成することで新たなパッチを生成する．

![](https://i.gyazo.com/3fbd515d274ac6ba811e4f8146e7669f.png)
![](https://i.gyazo.com/49f32f3f6e834a8f4fe81a51f6b4be0e.png)
### Consistency loss
ImageNetでトレーニングされたVGG-16のactivation map(画像のどの部分に注目しているか)と，CSA layer後のactivation mapのl2ノルムと，VGG-16のactivation mapと，CSAと対応する layerのl2ノルムの合計．

![](https://i.gyazo.com/1ff5137e19ac8bbbf0ad1fc67e2c8cb0.png)

### Future Patch Discriminator
VGG16といくつかの畳み込み層をつなげたディスクリミネータ．
従来の[feature discriminator](https://openaccess.thecvf.com/content_ECCV_2018/html/Seong-Jin_Park_SRFeat_Single_Image_ECCV_2018_paper.html)と[patch discriminator](https://arxiv.org/abs/1611.07004)のいいとこどり．

## この論文の主定理，もしくは鍵となる式・図は？（最大2個まで）
![](https://i.gyazo.com/e9564208d60730c4000a65b7df15b638.png)

![](https://i.gyazo.com/3fbd515d274ac6ba811e4f8146e7669f.png)

## この論文で何が出来るようになった？and/or 何が良くなった？　
* 欠損領域内パッチ間の相関を考慮したインペインティングが可能になった．



## 実験と評価方法
以下の既存の手法に比べて定量的にも定性的にも良い結果がでた．
- [CA: Contextual Attention, proposed by Yu et al.](https://arxiv.org/abs/1801.07892)
- [SH: Shift-net, proposed by Yan et al.](https://arxiv.org/abs/1801.09392)
- [PC: Partial Conv, proposed by Liu et al.](https://arxiv.org/abs/1804.07723)
- [GC: Gated Conv, proposed by Yu et al.](https://arxiv.org/abs/1806.03589)

### 定性評価
![](https://i.gyazo.com/18f4e376fbbd7cdafaa0590a66ac244d.png)

### 定量評価
Celebaデータセットから500枚ランダムに選んで，センターのマスクとランダムマスクとを生成，

#### センターのマスク
![](https://i.gyazo.com/cf7e7fd88c7f43b5e346bd417fe3ac59.png)

#### ランダムマスク
![](https://i.gyazo.com/ae3364e0ca74a4cdb395bafbe2c7fed5.png)

## この論文の問題点や課題は？
* 今後スタイル変換や超解像にも使える？

## 実装はある？
ある(PyTorch)

https://github.com/KumapowerLIU/CSA-inpainting
