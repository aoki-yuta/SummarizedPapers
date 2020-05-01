# Globally and Locally Consistent Image Completion
http://iizuka.cs.tsukuba.ac.jp/projects/completion/data/completion_sig2017.pdf

## 著者
- SATOSHI IIZUKA
- EDGAR SIMO-SERRA
- HIROSHI ISHIKAWA

## 何のための研究？
欠損領域の補間（インペインディング)


## 従来のアプローチとその問題点は？
### 1.パッチベースのアプローチ
#### アプローチ
元の画像からサンプルしたパッチ（局所領域）を（加工して）ターゲット画像にペーストする。サンプルには[PatchMatch](https://gfx.cs.princeton.edu/gfx/pubs/Barnes_2009_PAR/patchmatch.pdf)のようなアルゴリズムを用いることで高速に行うことが可能
#### 問題点
* 複雑な構造を復元できない
* 画像にない新しいオブジェクトを生成できない

### 2.学習ベースのアプローチ
#### アプローチ
CNNを用いた画像処理
#### 問題点
* 小さな画像でしかできない
* 処理に時間がかかる


## この論文ではどういうアプローチで問題を解決する？
画像の大域的な整合性と画像の局所的な整合性を得る。

３つのネットワークで構成される
1. 復元ネットワーク（ジェネレータ）
2. 大域的整合性ネットワーク（ディスクリミネータ）
3. 局所的整合性ネットワーク（ディスクリミネータ）

![画像](https://user-images.githubusercontent.com/60776249/80776952-a48cb580-8b9e-11ea-82d2-effd89588d8f.png)

## この論文の主定理，もしくは鍵となる式・図は？（最大2個まで）


## この論文で何が出来るようになった？and/or 何が良くなった？　

## 実験と評価方法


## この論文の問題点や課題は？
