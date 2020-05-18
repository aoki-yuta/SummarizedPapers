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
Single-Source synthesize(単一画像からのインペインディング)では
Serch（探索）ステップ→Vote（投票）ステップを繰り返す
シンプルなパッチマッチを用いて探索し，勾配を用いないで，画像のRGB情報のみから推測する．
損失関数は
<!-- latex:E(T,S) = \sum _{Q\subset T}\min_{P\subset S}\left\{ D(Q,P) \right \} -->
![image](https://latex.codecogs.com/gif.latex?E%28T%2CS%29%20%3D%20%5Csum%20_%7BQ%5Csubset%20T%7D%5Cmin_%7BP%5Csubset%20S%7D%5Cleft%5C%7B%20D%28Q%2CP%29%20%5Cright%20%5C%7D)
ここでTは穴，Sは穴以外の領域，QとPは各利用域内のパッチ．
Dはパッチ間の距離を測る関数（SSE）

#### Search（探索）ステップ
 [Patch Match](https://gfx.cs.princeton.edu/pubs/Barnes_2009_PAR/)や[Generalized PatchMatch](https://gfx.cs.princeton.edu/pubs/Barnes_2010_TGP/)を用いて領域T内の全てのピクセルに対し，各ピクセルを含むパッチQ（⊂T）（w×w）に最も似ているパッチP（⊂S）(w×w)を（w×w個）探す．ここでのPatch Matchによる探索は反射やアスペクト比を考慮しないパッチの探索である．
#### Vote（投票）ステップ
T(ターゲット領域）内の各ピクセルに対し，Serchステップで得た全ての類似パッチのRGBそれぞれの平均をとり，そのピクセルの値とする
### 問題点
* パッチの探索がシンプルなため複雑な曲面や対象の部分（口など）に対応できない．
* パッチ間の色同士のユークリッド距離しか見ていないため，埋めようとする場所とソースが違う色(照明のあたり具合など)を持っていると不自然な画像になる．
![image](https://user-images.githubusercontent.com/60776249/82153090-59fa8100-98a0-11ea-8565-6d53356fb064.png)



## この論文ではどういうアプローチで問題を解決する？
### この論文のアプローチ
Single-Source synthesize(単一画像からのインペインディング)では従来のSerch（探索）ステップ→Vote（投票）ステップに加え，Screend Poison reconstructionステップを行う．
復元は低解像度から各スケールで何度かこの３つを繰り返しながら徐々にスケールアップして高解像度にする．

損失関数は従来の損失関数に加えて赤い部分が追加された．
<!-- latex:E(T,S) = \sum _{Q\subset T}\min_{P\subset S}\{ D(Q,{\color{Red} f}(P))+{\color{Red} \lambda D(\bigtriangledown Q,\bigtriangledown f(P))}\} -->
![image](https://latex.codecogs.com/gif.latex?E%28T%2CS%29%20%3D%20%5Csum%20_%7BQ%5Csubset%20T%7D%5Cmin_%7BP%5Csubset%20S%7D%5C%7B%20D%28Q%2C%7B%5Ccolor%7BRed%7D%20f%7D%28P%29%29&plus;%7B%5Ccolor%7BRed%7D%20%5Clambda%20D%28%5Cbigtriangledown%20Q%2C%5Cbigtriangledown%20f%28P%29%29%7D%5C%7D)

加えられたものとその効果を示す
#### 1. 関数f
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

#### 2.勾配項
- 【勾配項】勾配（∇Q,∇ f(P)）の距離を追加
  - 照明や色の変化に対して，より頑健になる


いくつかの手法のいいとこ取りをして組み合わせる．
1. パッチの探索範囲の拡張【f】
2. 画像の標準化を行う【f】
3. パッチベース の手法に勾配を加える．【∇】
4. 損失関数にL2とL0ノルムを使う【アニメ】

##### 1.パッチの探索範囲の拡張


### 2.パッチベースの手法に勾配を加えた
パッチベースの手法に勾配を加えてうまいこと混ぜることでなんんかいい感じ
色の変化や

### 3.損失関数にL2とL0ノルムを使う
### 4.画像の標準化を行う



## この論文で何が出来るようになった？and/or 何が良くなった？　
パッチマッチにリフレクションやnon uniform scaleを加えることで

## 実験と評価方法


## この論文の問題点や課題は？


## 実装はある？
