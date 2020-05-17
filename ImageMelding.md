# Image Melding: Combining Inconsistent Images using Patch-based Synthesis

[論文PDF](https://www.ece.ucsb.edu/~psen/melding)

## 著者
- Soheil Darabi
- Eli Shechtman
- Connelly Barnes
- Dan B Goldman
- Pradeep Sen

## Abstract訳
近年の手法では，二つの異なる画像を合成する際，その二つの画像が（全く）異なるテクスチャや構造を持っている時，アーティファクトが発生する．私たちは２つの画像間の領域の遷移？（a transition region：整合性のない色、テクスチャ、構造的の全ての特性が徐々に一つの画像からもう一つの画像に先するような操作）を合成のための新しい手法を提案する.私たちはこれを"Image Melding"と呼ぶ．私たちの手法は３つの重要な一般化を含むパッチベースの最適化の基礎の上に成り立つ。
1. （追加の幾何、輝度を考慮した変形を行うことで）パッチサーチの探索空間をよりリッチにした
2. 画像勾配をパッチ表現に拡張し、一般的な画像平均を選ばれたポアソン分布の解で置き換える？
3. 新しいL2ノルムとL0ノルムを合わせた新しい損失関数を色と勾配のために提案する．これはテクスチャのシャープネスを損ねずに二つの画像間で段階的な変化を与える    

３つの一般化によってパッチベースの解決法によって一貫性のない画像を含む様々な画像間で起こるImageMeldingの課題の解決ができるようになった（課題：オブジェクトのクローン，複雑なパノラマの作成，多様な写真の穴埋め（インペインティング），画像調和）．それぞれのケースで我々の統合された手法は過去の（特にこれらのアプリケーションに向けて作られた）SOTAを上回った．


## 何のための研究？
【main】二つの画像間の自然な遷移（Image Melding）
【sub】欠損領域の補間（Image Impainting)
【sub】画像の調和（Image Harmonaize）

## 従来のアプローチとその問題点は？

### 従来のアプローチ
Single-Source synthesize(単一画像からのインペインディング)
Serch（探索）ステップとVote（投票）ステップを繰り返す
勾配を用いないで，画像のRGB情報のみから推測する．
シンプルなパッチマッチ
損失関数は
<!-- latex:E(T,S) = \sum _{Q\subset T}\min_{P\subset S}\left\{ D(Q,P) \right \} -->
![image](https://latex.codecogs.com/gif.latex?E%28T%2CS%29%20%3D%20%5Csum%20_%7BQ%5Csubset%20T%7D%5Cmin_%7BP%5Csubset%20S%7D%5Cleft%5C%7B%20D%28Q%2CP%29%20%5Cright%20%5C%7D)
ここでTは穴，Sは穴以外の領域，QとPは各利用域内のパッチ．
Dはパッチ間の距離を測る関数（SSE）

#### Search（探索）ステップ
 [Patch Match](https://gfx.cs.princeton.edu/pubs/Barnes_2009_PAR/)や[Generalized PatchMatch](https://gfx.cs.princeton.edu/pubs/Barnes_2010_TGP/)を用いて領域T内の各ピクセルを含むパッチQ（⊂T）に最も似ているパッチP（⊂S）を探す．ここでのPatch Matchによる探索は反射やアスペクト比を考慮しないパッチの探索である．
#### Vote（投票）ステップ
T(ターゲット領域）内の各ピクセルに対し，Serchステップで得た全ての類似パッチのRGBそれぞれの平均をとり，そのピクセルの値とする
### 問題点
* パッチ間の色同士のユークリッド距離しか見ていないため，埋めようとする場所とソースが違う色(照明のあたり具合など)を持っていると不自然な画像になる．
* パッチの探索がシンプルなため複雑な曲面や対象の部分（口など）に対応できない．
![image](https://user-images.githubusercontent.com/60776249/82153090-59fa8100-98a0-11ea-8565-6d53356fb064.png)



## この論文ではどういうアプローチで問題を解決する？
### この論文のアプローチ
・領域S（ソース）のパッチに関数を食わせてからやる
関数：
回転
拡大
アスペクト比
反射
↑幾何学的な違和感
colorgain
color bias
↑色的な違和感
・勾配を加える
↑ライティングや色の変化に剛健になる

損失関数
<!-- latex:E(T,S) = \sum _{Q\subset T}\min_{P\subset S}\{ D(Q,{\color{Red} f}(P))+{\color{Red} \lambda D(\bigtriangledown Q,\bigtriangledown f(P))}\} -->
![image](https://latex.codecogs.com/gif.latex?E%28T%2CS%29%20%3D%20%5Csum%20_%7BQ%5Csubset%20T%7D%5Cmin_%7BP%5Csubset%20S%7D%5C%7B%20D%28Q%2C%7B%5Ccolor%7BRed%7D%20f%7D%28P%29%29&plus;%7B%5Ccolor%7BRed%7D%20%5Clambda%20D%28%5Cbigtriangledown%20Q%2C%5Cbigtriangledown%20f%28P%29%29%7D%5C%7D)


いくつかの手法のいいとこ取りをして組み合わせる．
１．パッチの探索範囲の拡張
２．パッチベース の手法に勾配を加える．
３．損失関数にL2とL0ノルムを使う
４．画像の標準化を行う

### 1.パッチの探索範囲の拡張
Generaized Patch Matchに反射とnon uniform scaleを加えた

|                                    | Patch Match | Generaized PatchMatch | 提案手法 |
| ---------------------------------- | ----------- | --------------------- | -------- |
| 等倍                               | ○           | ○                     | ○        |
| 拡大・回転                         | ×           | ○                     | ○        |
| 反射・非均一拡大（アスペクト変化） | ×           | ×                     | ○        |

### 2.パッチベースの手法に勾配を加えた
パッチベースの手法に勾配を加えてうまいこと混ぜることでなんんかいい感じ

### 3.損失関数にL2とL0ノルムを使う
### 4.画像の標準化を行う



## この論文で何が出来るようになった？and/or 何が良くなった？　
パッチマッチにリフレクションやnon uniform scaleを加えることで

## 実験と評価方法


## この論文の問題点や課題は？


## 実装はある？
