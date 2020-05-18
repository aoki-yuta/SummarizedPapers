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
3. 新しいL2ノルムとL0ノルムを合わせた新しい目的関数を色と勾配のために提案する．これはテクスチャのシャープネスを損ねずに二つの画像間で段階的な変化を与える    

３つの一般化によってパッチベースの解決法によって一貫性のない画像を含む様々な画像間で起こるImageMeldingの課題の解決ができるようになった（課題：オブジェクトのクローン，複雑なパノラマの作成，多様な写真の穴埋め（インペインティング），画像調和）．それぞれのケースで我々の統合された手法は過去の（特にこれらのアプリケーションに向けて作られた）SOTAを上回った．


## 何のための研究？
* 【main】二つの画像間の自然な遷移（Image Melding）
* 【sub】欠損領域の補間（Image Impainting)
* 【sub】画像の調和（Image Harmonaize）

## 従来のアプローチとその問題点は？

### 従来のアプローチ
Single-Source synthesize(単一画像からのインペインディング)では
Serch（探索）ステップ→Vote（投票）ステップを繰り返す
シンプルなパッチマッチを用いて探索し，勾配を用いないで，画像のRGB情報のみから推測する．
目的関数は
<!-- latex:E(T,S) = \sum _{Q\subset T}\min_{P\subset S}\left\{ D(Q,P) \right \} -->
![image](https://latex.codecogs.com/gif.latex?E%28T%2CS%29%20%3D%20%5Csum%20_%7BQ%5Csubset%20T%7D%5Cmin_%7BP%5Csubset%20S%7D%5Cleft%5C%7B%20D%28Q%2CP%29%20%5Cright%20%5C%7D)
ここでTは穴，Sは穴以外の領域，QとPは各利用域内のパッチ．
Dはパッチ間の距離を測る関数（SSE）

---

### アルゴリズム
#### Search（探索）ステップ
 [Patch Match](https://gfx.cs.princeton.edu/pubs/Barnes_2009_PAR/)や[Generalized PatchMatch](https://gfx.cs.princeton.edu/pubs/Barnes_2010_TGP/)を用いて領域T内の全てのピクセルに対し，各ピクセルを含むパッチQ（⊂T）（w×w）に最も似ているパッチP（⊂S）(w×w)を（w×w個）探す．ここでのPatch Matchによる探索は反射やアスペクト比を考慮しないパッチの探索である．
#### Vote（投票）ステップ
T(ターゲット領域）内の各ピクセルに対し，Serchステップで得た全ての類似パッチのRGBそれぞれの単純な平均をとり，そのピクセルの値とする

---

### 問題点
* パッチの探索がシンプルなため複雑な曲面や対象の部分（口など）に対応できない．
* パッチ間の色同士のユークリッド距離しか見ていないため，埋めようとする場所とソースが違う色(照明のあたり具合など)を持っていると不自然な画像になる．
![image](https://user-images.githubusercontent.com/60776249/82153090-59fa8100-98a0-11ea-8565-6d53356fb064.png)



## この論文ではどういうアプローチで問題を解決する？
### この論文のアプローチ
Single-Source synthesize(単一画像からのインペインディング)では従来のSerch（探索）ステップ→Vote（投票）ステップに加え，Screend Poison reconstructionステップを行う．
復元は低解像度から各スケールで何度かこの３つを繰り返しながら徐々にスケールアップして高解像度にする．

目的関数は従来の目的関数に加えて赤い部分が追加された．
<!-- latex:E(T,S) = \sum _{Q\subset T}\min_{P\subset S}\{ D(Q,{\color{Red} f}(P))+{\color{Red} \lambda D(\bigtriangledown Q,\bigtriangledown f(P))}\} -->
![image](https://latex.codecogs.com/gif.latex?E%28T%2CS%29%20%3D%20%5Csum%20_%7BQ%5Csubset%20T%7D%5Cmin_%7BP%5Csubset%20S%7D%5C%7B%20D%28Q%2C%7B%5Ccolor%7BRed%7D%20f%7D%28P%29%29&plus;%7B%5Ccolor%7BRed%7D%20%5Clambda%20D%28%5Cbigtriangledown%20Q%2C%5Cbigtriangledown%20f%28P%29%29%7D%5C%7D)

加えられたものとその効果を示す
#### 【関数f】
パッチP（⊂S）に適用
幾何学的な不整合への対処と色の不整合に対処を同時に行う
##### 幾何学的な不整合に対処
パッチの探索をGeneraized Patch Matchに反射とnon uniform scaleを加えてよりリッチにすることで，幾何学的な不自然さを減らした．（ex:反射によって片方の目の一部が欠損していても，もう片方の目を反転して持ってこれる）

  |                                    | Patch Match | Generaized PatchMatch | 提案手法 |
  | ---------------------------------- | ----------- | --------------------- | -------- |
  | 等倍                               | ○           | ○                     | ○        |
  | 拡大・回転                         | ×           | ○                     | ○        |
  | 反射・non uniform scale（アスペクト変化） | ×           | ×                     | ○        |

##### 色の不整合に対処
似ているパッチを探す際に単純な色同士の距離を測っているため，日の当たり具合による領域ごとの色の違い（単一画像補間）やホワイトバランスの違い（複数画像での補間）などによって，ターゲットパッチの真の値とソースパッチの値が違う色あいを持っている場合，似ているパッチの探索がうまくできない．そこでHaCohenらによる[Generaized PatchMatchの拡張](https://www.cse.huji.ac.il/~yoavhacohen/nrdc/)では
- color gain
- color bias

の二つを用いてパッチ探索を行うことで，より正確に似ているパッチを探している．

#### 【勾配項】
画像の勾配（∇Q,∇ f(P)）の距離を追加．
画像の勾配を考慮することで，照明や色の変化に対して，より頑健になる

---
### アルゴリズム
#### 1.Serch（探索）ステップ
ターゲット領域内の各ピクセルを含む全てのパッチに対し，それぞれに似ているパッチを前述の拡張されたPatch Matchを用いて探す．結果として得られるパッチは（ターゲットのピクセルを左上に含むパッチからターゲットのパッチを右下に含むパッチまでの）（w×w）個になる．（wはパッチの一辺）

#### 2.Vote（投票）ステップ　
ターゲット領域T内のあるピクセルについて，
1. Serchで求めた（w×w）個のそれぞれのサンプル内で，求めたいピクセルに対応するピクセルの勾配(∇X，∇Y）を求める．
2. 各ピクセルに対し，サンプルされたw×w個のパッチの画素を平均する（この論文ではLab色空間を使っている）
3. 各ピクセルに対し，サンプルされたw×w個のパッチの勾配を平均する．

この時,領域T内のあるピクセル(i,j)について，投票後の画素値![Tbar](https://latex.codecogs.com/gif.latex?%5Coverline%20%7B%5Cnabla%20T%7D%28i%2Cj%29),及び勾配![nTbar](https://latex.codecogs.com/gif.latex?%5Coverline%20%7BT%7D%28i%2Cj%29)はサンプルしたパッチから平均を求めたもので，以下の式で求まる．  

![image](https://latex.codecogs.com/gif.latex?%5Coverline%20%7B%20T%7D%28i%2Cj%29%3D%5Csum%20_%7B%5Csubstack%7Bk%3D0...w-1%20%5C%5C%20l%3D0...w-1%7D%7D%5Cfrac%7BNN%28Q_%7Bi-k%2Cj-l%7D%29%28k%2Cl%29%7D%7Bw%5E2%7D)

![image](https://latex.codecogs.com/gif.latex?%5Coverline%20%7B%5Cnabla%20T%7D%28i%2Cj%29%3D%5Csum%20_%7B%5Csubstack%7Bk%3D0...w-1%20%5C%5C%20l%3D0...w-1%7D%7D%5Cfrac%7BNN%28Q_%7Bi-k%2Cj-l%7D%29%28k%2Cl%29%7D%7Bw%5E2%7D)  

ここで![Q](https://latex.codecogs.com/gif.latex?Q_%7Bi%2Cj%7D)（左上をi,jとする）は注目パッチで，
![NN](https://latex.codecogs.com/gif.latex?NN%28Q_%7Bi-k%2Cj-l%7D%29%28k%2Cl%29%7D%7B)は注目パッチに類似するパッチである．（k,l）は類似パッチ内の求めたいピクセルの位置になる．ピクセルに対して全てのパッチの対応する箇所をw<sup>2</sup>で割ることで平均している
<!--
\overline {\nabla T}(i,j)=\sum _{\substack{k=0...w-1 \\ l=0...w-1}}\frac{NN(Q_{i-k,j-l})(k,l)}{w^2}
-->

#### 3.Screend Poison reconstructionステップ
画像の画素値と勾配の両方を考慮したポアソン方程式を解くことで，画像Tを得る.画像Tは以下の式で表される．

![T](https://latex.codecogs.com/gif.latex?T%20%3D%20%5Carg%20%5Cmin_I%20%5C%7BD%28I%2C%5Coverline%20T%29&plus;%5Clambda%20D%28%5Cnabla%20I%2C%20%5Coverline%7B%5Cnabla%20T%7D%29%5C%7D)

ここでλは画素値と勾配のどちらを重視するかのパラメータである．
ポアソン方程式は[Gradient Shop](https://dl.acm.org/doi/10.1145/1731047.1731048)を用いて解かれる（らしい）





いくつかの手法のいいとこ取りをして組み合わせる．
1. パッチの探索範囲の拡張【f】Done
2. 画像の標準化を行う【f】Done
3. パッチベース の手法に勾配を加える．【∇】Done
4. 目的関数にL2とL0ノルムを使う【アニメ】



### 3.目的関数にL2とL0ノルムを使う
### 4.画像の標準化を行う



## この論文で何が出来るようになった？and/or 何が良くなった？　
パッチマッチにリフレクションやnon uniform scaleを加えることで

## 実験と評価方法


## この論文の問題点や課題は？


## 実装はある？
