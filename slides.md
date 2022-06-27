---
# シンプルに始めるなら「default」も試してみてください。
theme: seriph
# ランダム画像：アンソニー氏によるUnsplashコレクションより
# 気に入りましたか？https://unsplash.com/collections/94734566/slidev をご覧ください。
background: https://source.unsplash.com/collection/94734566/1920x1080
# 現在のスライドに任意の windi css クラスを適用します。
class: 'text-center'
# https://sli.dev/custom/highlighters.html
highlighter: shiki
# コードブロックの行番号を表示
lineNumbers: false
# スライドに関するいくつかの情報、マークダウンを有効にする
info: |
  ## Slidev Starter Template
  Presentation slides for developers.

  Learn more at [Sli.dev](https://sli.dev)
# エクスポートとビルドで図面を保持する
drawings:
  persist: false
---

# Patrolling robot based on Bayesian learning for multiple intruders

公立はこだて未来大学 原田 裕斗

<div class="pt-12">
  <span @click="$slidev.nav.next" class="px-2 py-1 rounded cursor-pointer" hover="bg-white bg-opacity-10">
    Spaceキーを押して次へ <carbon:arrow-right class="inline"/>
  </span>
</div>

<div class="abs-br m-6 flex gap-2">
  <button @click="$slidev.nav.openInEditor()" title="Open in Editor" class="text-xl icon-btn opacity-50 !border-none !hover:text-white">
    <carbon:edit />
  </button>
  <a href="https://github.com/slidevjs/slidev" target="_blank" alt="GitHub"
    class="text-xl icon-btn opacity-50 !border-none !hover:text-white">
    <carbon-logo-github />
  </a>
</div>

<!--
各スライドの最後のコメントブロックは、スライドノートとして扱われます。発表者モードでは、スライドと一緒に表示され、編集可能です。 [詳しくはドキュメントをご覧ください](https://sli.dev/guide/syntax.html#notes)
-->

---

# TL;DR

長すぎるという方のための要約です. 飛ばします.

- 自律移動ロボットのパトロール問題について考える
  - 複数の部屋からなる屋内環境を効率的にパトロールするにはどうすればよいか
- ロボットは, 開始段階で, 侵入者が各部屋にいる確率(存在確率)を持っていない
- 試行を通じて存在確率を取得し, パトロールする部屋を決定する
  - 探索/活用のトレードオフとして多腕バンディット問題を導入
- 存在確率は存在確率分布の期待値として定義
  - 存在確率分布を算出するためにベイズ学習を導入
- シミュレーション実験の結果, ロボットは部屋の存在確率を正しく推定できた

<br>
<br>

[こちら](https://www.researchgate.net/publication/308814685_Patrolling_robot_based_on_Bayesian_learning_for_multiple_intruders)から論文を閲覧できます

<!--
マークダウンで `style` タグを使用すると、現在のページのスタイルを上書きすることができます。
詳細はこちら: https://sli.dev/guide/syntax#embedded-styles
-->

<style>
h1 {
  background-color: #2B90B6;
  background-image: linear-gradient(45deg, #4EC5D4 10%, #146b8c 20%);
  background-size: 100%;
  -webkit-background-clip: text;
  -moz-background-clip: text;
  -webkit-text-fill-color: transparent;
  -moz-text-fill-color: transparent;
}
</style>

---

# I. 概要 #1

自律移動ロボットにとってパトロールは重要なタスク

<div class="flex">

  - 通常警備を行う上で, 地図, 過去の犯罪データなどから犯罪発生率を予測
    - 危険な場所を中心にパトロール
  - ロボットが未知の侵入者をできるだけ多く検知するには...
    1. 観測点の発見
        - アートギャラリー問題, センサープランニング問題 [1][2]
        - Trevaiらの反応拡散方程式に基づくパトロールアルゴリズム[3]
    2. 観測点間をどのように移動するか
        - Srivastavaらのマルコフ連鎖に基づくグラフ[5]
        - Srivastavaらの観測点(ノード)に現れるかもしれない, ランダムな異常に対しての永続的な監視戦略[6]
        - Agharkarらのマルコフ連鎖を用いたロボット監視のための効率的なルーティングアルゴリズム[7]

</div>


---

# I. 概要 #2

先行研究をもとに本論文で行うこと

- 複数の部屋からなる屋内環境を1台の警備ロボットがパトロール
- 複数の侵入者が同時に部屋に入ってくる可能性あり
- ロボットは初期状態として侵入者の情報(侵入者が現れる部屋)を持たない

<br>

#### パトロール戦略 <span class="text-xs m-0 text-gray-400">後で詳しく取り上げます</span>


  - ベイジアンを用いて, 各部屋に対しての存在確率に対する確率分布を定義
    - Portugalらのベイズ学習アプローチ [8][9]
  - 存在確率をもとに警備先の部屋を決定し, 警備先で得られた巡回結果を基に, 次の警備先を決定
    - 多腕バンディット問題 - Tompson Samplingアルゴリズム[12][13][14]

<br>
<div class="text-center">

  #### 先行研究との違いは, 侵入者に応じた環境パトロールを行うという点

</div>

---

# II. 確率的多腕バンディット問題
多腕バンディット問題ってなに？

### 問題提起
<br>

- スロットマシーンが$N$台あり, それぞれアームが1つ付いている
- アームを引き, 勝てば報酬がもらえる, 負ければ報酬は支払われない
- 初期状態では, どのスロットマシーンが勝率が高いかわからない
- 十分に試行しないとどのアームが勝率が高いかわからないが, 勝率の低いアームを引き続けると, より多くの報酬を見送ることになる

<div class="flex flex-col items-center">
  <img src="/multiarmedbandit.jpg" class="w-60">
</div>

---

# II. 確率的多腕バンディット問題
この問題にどう扱っていくか

### アプローチ方法
<br>

- この問題は第二次世界大戦中に生み出されたもの
- あまりに難解で今でも多くの人がこの問題に取り組んでいる
  - 機械学習 - 強化学習(: 試行錯誤を通じて最適な選択を見出していく)
- 試行錯誤にどれだけのリソースを費やすべきか, 利益を最大化するか
  - **探索と活用のトレードオフの問題**

<div class="flex flex-col items-center">
  <img src="/multiarmedbandit.jpg" class="w-60">
</div>

---

# II. 確率的多腕バンディット問題
この問題を数学的なモデルで表すとどうなる？

### 数学的定式化 #1
<br>

- 各スロットマシーンについて, 下の図のような確率分布が定義できる
  - 下の図では, 左に行くほど勝率が高い ←確率の確率
  - 確率分布は$N$台のスロットマシーンで独立
  - 独立同分布

<br>

<div class="flex flex-col items-center">
  <img src="/unknown-parameters-to-players.png" class="h-48">
</div>

---

# II. 確率的多腕バンディット問題
この問題を数学的なモデルで表すとどうなる？

### 数学的定式化 #2
<br>

- ここでは, 勝率は確率分布の期待値で定義される
- プレイヤーは開始時, この確率分布はわからない
  - ゲームをプレイし, 情報を蓄積(＝探索)する必要がある

<br>
<div class="flex flex-col items-center mt-8">
  <img src="/unknown-parameters-to-players.png" class="h-48">
</div>

---

# II. 確率的多腕バンディット問題
この問題を数学的なモデルで表すとどうなる？

### 数学的定式化 #3
<br>

- 試行回数 $T$回のゲームにおける報酬最大化を行う目的関数:
$$
\max_iE[\sum_{t=1}^T\mu_{i(t)}]
$$
- スロットマシーン $i$ 
- 勝率(期待報酬) $\mu_i$ ←確率分布の期待値
- $i(t)$ : $t$ 番目のゲームをプレイしたマシーン$i$
- 期待値$\mu_{i(t)}$ が最も高いスロットマシーン$i$でプレイすると報酬(勝率)が最大になる
- 目標: 報酬の総和ができるだけ大きくなるようにする

---

# II. 確率的多腕バンディット問題
この問題を数学的なモデルで表すとどうなる？

### 数学的定式化 #4
<br>

- 個々のスロットマシーンは独立であるから:
$$
\sum_{i=1}^N\mu_{i(t)} \neq 1.0
$$
- スロットマシーン $i$ 
- 勝率(期待報酬) $\mu_i$ ←確率分布の期待値
- $i(t)$ : $t$ 番目のゲームをプレイしたマシーン$i$

---

# III. 複数の侵入者におけるパトロール問題
確率的多腕バンディット問題を応用すると...

### 問題提起 #1
<br>

- 部屋が$N$個ある
- ロボットは巡回結果(侵入者を検出, 非検出のどちらか)を観測する
- 検出, 非検出に応じて, それぞれ報酬が与えられる
- 複数の侵入者が同時に入ってくる可能性あり<span class="text-gray-400">(再活)</span>
  - 1回のパトロールで複数の部屋を訪問しなければならない

<!-- note:  
存在確率が最も高い最適な部屋だけを探して訪問するのでは、パトロール問題としては不十分

パトロール試行ごとに、ロボットは$N$個の部屋のうちいくつかを訪問する行動をとる
-->

<div class="flex flex-col items-center mt-8">
  <img src="/patrolling-problem-for-multiple-intruders.png" alt="複数の侵入者におけるパトロール問題" class="w-125">
</div>



---

# III. 複数の侵入者におけるパトロール問題
確率的多腕バンディット問題を応用すると...

<div class="flex flex-row">
  <div>

  ### 問題提起 #2
  <br>

  - 図3のように侵入者は何らかの侵入目標を持っているはず
  - ロボットは, 侵入者の思考を直接うかがい知ることはできない
    - パトロールで得られた情報を基に, 侵入目標に関する確率分布を定義
    - 確率分布の期待値 = 存在確率<span class="text-gray-400"> (勝率)</span>
  - 多腕バンディット問題と同じ条件
    - 確率分布は$N$部屋間で独立
    - ある部屋を繰り返し訪問することによって得られる報酬は独立同分布

  </div>

  <div class="flex flex-col items-center">
    <img src="/schematic-view-of-intelligent-patrolling.png" alt="複数の侵入者におけるパトロール問題" class="w-80 ml-10">
  </div>

</div>

---

# III. 複数の侵入者におけるパトロール問題
ベイズの定理を使って確率分布をつくる

### ベイズの定理を使った存在確率分布の算出 #1
<br>

#### 事後確率 $P_i(\theta_i\mid X_i)$

<br>

- 部屋$i$ の存在確率の確率密度関数$P_i(\theta_i\mid X_i)$は, ベイズの定理を使って…

$$ 
P_i(\theta_i \mid X_i)=\frac{P_i(X_i \mid \theta_i) P_i(\theta_i)}{P_i(X_i)} \propto P_i(X_i \mid \theta_i) P_i(\theta_i) \tag{1}
$$

- $\theta_i$ :  $i$番目の部屋への存在確率
- $X_i$ : 報酬パラメータ
- $N$個の部屋への存在確率は、$\sum_{i}\theta_i\neq1.0$ とする

---

# III. 複数の侵入者におけるパトロール問題
ベイズの定理を使って確率分布をつくる

### ベイズの定理を使った存在確率分布の算出 #2
<br>

#### 事前確率 $P_i(\theta_i)$
<br>

- 巡回結果は侵入者を検知するかしないかのどちらか
  - 2値しか結果を持たない = ベルヌーイ試行
- 式(1)の事前確率密度関数$P_i(\theta_i)$ はベータ分布より…

$$ 
P_i(\theta_i)=\frac{\theta_i^{\alpha_i-1}(1-\theta_i)^{\beta_i-1}}{Be(\alpha_i, \beta_i)} \propto \theta_i^{\alpha_i-1}(1-\theta_i)^{\beta_i-1} \tag{2}
$$

- $\alpha_i$ , $\beta_i$ : 巡回試行を通じてわかった $i$番目の部屋の侵入者の検出数と非検出数の合計
- パトロールの初期状態: $\alpha_i=1$、$\beta_i=1$ (式(2)は一様分布を示す )
- $Be(\cdot, \cdot)$ : ベータ関数

---

# III. 複数の侵入者におけるパトロール問題
ベイズの定理を使って確率分布をつくる

### ベイズの定理を使った存在確率分布の算出 #3
<br>

#### 尤度関数 $P_i(X_i \mid \theta_i)$
<br>

- 巡回行動はベルヌーイ試行であるから、尤度関数$P_i(X_i\mid\theta_i)$は二項分布に基づいて…

$$ 
P_i(X_i \mid \theta_i)=\left(\begin{array}{c}
A_i+B_i \\
A_i
\end{array}\right) \theta_i^{A_i}(1-\theta_i)^{B_i} \propto \theta_i^{A_i}(1-\theta_i)^{B_i} \tag{3}
$$

- $A_i$ , $B_i$ : 報酬 $X_i$に含まれる報酬パラメータ
  - 侵入者を検出した場合 : $A_i=1$および$B_i=0$
  - 侵入者を検出しなかった場合 : $A_i=0$および$B_i=1$

---

# III. 複数の侵入者におけるパトロール問題
ベイズの定理を使って確率分布をつくる

### ベイズの定理を使った存在確率分布の算出 #4
<br>

#### 事後確率 $P_i(\theta_i\mid X_i)$の変形 ##1
<br>

- 式（2）と式（3）で定式化された事前確率密度関数$P_i(\theta_i)$と尤度関数$P_i(X_i \mid \theta_i)$から、式（1）は以下のように再定式化される

$$ 
P_i(\theta_i \mid X_i) \propto \theta_i^{\alpha_i+A_i-1}(1-\theta_i)^{\beta_i+B_i-1} \tag{4}
$$

- 事前確率 $P_i(\theta_i)$をベータ分布, 尤度関数を二項分布として事後確率 $P_i(\theta_i\mid X_i)$を再度, 定式化
- ベータ分布である事前確率 $P_i(\theta_i)$は, 二項分布である尤度関数$P_i(X_i \mid \theta_i)$の共役事前分布なので, 事後確率$P_i(\theta_i \mid X_i)$もベータ分布
  - $P_i(\theta_i \mid X_i)$は、式（4）において$P_i(\theta_i)$として繰り返し使用可能

---

# III. 複数の侵入者におけるパトロール問題
ベイズの定理を使って確率分布をつくる

### ベイズの定理を使った存在確率分布の算出 #5
<br>

#### 事後確率 $P_i(\theta_i\mid X_i)$の変形 ##2
<br>

- 式(4)を一般化すると式(5)に..

$$ 
P_i(\theta_i \mid X_i) \propto \theta_i^{\alpha_i+A_i-1}(1-\theta_i)^{\beta_i+B_i-1} \tag{4}
$$

$$ 
P_i(\theta_i \mid X_i)=f_i(\theta_i ; \alpha_i+A_i, \beta_i+B_i) \tag{5}
$$

- 本論文では, 巡回結果に応じて、$i$番目の部屋の事後確率密度関数を式(5) から算出

---

# IV. ベイズ学習に基づく確率的パトロール戦略
実際にどのようなアルゴリズムパトロールを行うのか

<div class="flex flex-row justify-between">

  <div>

  ### 侵入者が1人の場合のパトロール戦略 #1
  <br>

  - この場合, 多腕バンディット問題とパトロール問題は等価
  - パトロール問題には, Thompson Sampling アルゴリズム [12] を適用
    - 期待存在確率が最も高い最適な部屋を見つけることができる[13][14]
  
  </div>

  <div class="flex flex-col items-center">
    <img src="/tompson-sampling-for-a-single-intruder.png" alt="アルゴリズム1: 単一の侵入者に対するトンプソンサンプリング" class="w-80 ml-5">
    <img src="/pdf-and-cdf-of-beta-distribution.png" alt="アルゴリズム1: 単一の侵入者に対するトンプソンサンプリング" class="w-80 mt-2 ml-5">  
  </div>

</div>

---

# IV. ベイズ学習に基づく確率的パトロール戦略
実際にどのようなアルゴリズムパトロールを行うのか

<div class="flex flex-row justify-between">

  <div>

  ### 侵入者が1人の場合のパトロール戦略 #2
  <br>

  #### Thompson Sampling アルゴリズム
  <br>

  - `(1), (2)`について
    - 図4(a) $\alpha=10$, $\beta=10$ に基づく$P(\theta)$ の確率密度関数(PDF)
    - 図4(b) ベータ分布に基づくPDFから導かれる累積確率分布(CDF)
    - CDFに対して、ロボットは、$[0, 1]$ の一様乱数値を発生させる
    - その乱数値に基づいてロボットが確率$\theta$をサンプリング
    - サンプリング処理は各部屋で実行
    - 最後に、$\theta_{i(t)}$の値が最大となる部屋$i$のみが巡回試行のターゲット

  </div>

  <div class="flex flex-col items-center">
    <img src="/tompson-sampling-for-a-single-intruder.png" alt="アルゴリズム1: 単一の侵入者に対するトンプソンサンプリング" class="w-80 ml-5">
    <img src="/pdf-and-cdf-of-beta-distribution.png" alt="アルゴリズム1: 単一の侵入者に対するトンプソンサンプリング" class="w-80 mt-2 ml-5">  
  </div>

</div>

---

# IV. ベイズ学習に基づく確率的パトロール戦略
実際にどのようなアルゴリズムパトロールを行うのか

<div class="flex flex-row justify-between">

  <div>

  ### 侵入者が1人の場合のパトロール戦略 #3
  <br>

  <p class="text-gray-400">アルゴリズムの解説のづづき</p>

  - ロボットは訪問した部屋の報酬を観測
    - 報酬$r_{i(t)}=1$は巡回結果に応じて$A_{i(t)}$ or $B_{i(t)}$
    - もう一方は$0$
  - 式(5)を用いて部屋の事後確率$P_{i(t)}(\theta_{i(t)} \mid X_{i(t)})$を算出
  - この場合, 試行中の侵入者検出数を最大にすることができる

  </div>

  <div class="flex flex-col items-center">
    <img src="/tompson-sampling-for-a-single-intruder.png" alt="アルゴリズム1: 単一の侵入者に対するトンプソンサンプリング" class="w-80 ml-5">
    <img src="/pdf-and-cdf-of-beta-distribution.png" alt="アルゴリズム1: 単一の侵入者に対するトンプソンサンプリング" class="w-80 mt-2 ml-5">  
  </div>

</div>

---

# IV. ベイズ学習に基づく確率的パトロール戦略
実際にどのようなアルゴリズムパトロールを行うのか

<div class="flex flex-row justify-between">

  <div>

  ### 侵入者が複数の場合のパトロール戦略 #1
  <br>

  - ロボットが最適な部屋を訪問しても、他の部屋に侵入が入ってくる可能性がある
    - 1回の試行で複数の部屋を巡回する必要がある
  - 目標: 最小限の訪問数 & 侵入者検知の最大化
  - 2つの巡回戦略を提示する 
    - <span class="text-gray-400">次のスライドから→</span>

  </div>

</div>

---

# IV. ベイズ学習に基づく確率的パトロール戦略
実際にどのようなアルゴリズムパトロールを行うのか

<div class="flex flex-row justify-between">

  <div>

  ### 侵入者が複数の場合のパトロール戦略 #2
  <br>

  #### 予想される存在確率に着目したアルゴリズム

  - ロボットは各部屋にいる侵入者のPDFをもつ
  - 期待存在確率$E$は、PDFより
  $$
  E = \frac{\alpha}{\alpha+\beta}
  $$
  - $\alpha$, $\beta$は侵入者の検出と非検出の合計数
  - $t$回目の巡回試行, 各部屋$i$に対し, 期待存在確率$E_{i(t)}$ に基づいて判定
    - ロボットは複数の部屋の巡回が可能
  - 存在確率が小さい場合（例えば、$\sum_iE_i << 1.0$)
    - ロボットはある巡回試行でどの部屋にも行かない

  </div>

  <div class="flex flex-col items-center">
    <img src="/decision-making-based-only-on-expected-intrusion-probabilities.png" class="w-90 ml-5">
  </div>

</div>

---

# IV. ベイズ学習に基づく確率的パトロール戦略
実際にどのようなアルゴリズムパトロールを行うのか

<div class="flex flex-row justify-between">

  <div>

  ### 侵入者が複数の場合のパトロール戦略 #3
  <br>

  #### アルゴリズム1とアルゴリズム2の融合

  - Thompson Samplingアルゴリズムで部屋(ターゲット)を決定
  - ターゲットを巡回後, 予想存在確率を基に巡回する部屋を再度決定
    - 部屋$i$を訪問後、部屋$i$を除く$N-1$個の部屋$j$に対し, 
    - 期待存在確率$E_{j(t)}$に基づいて追加的な判断をする
  - 期待存在確率が小さくても, 少なくとも1つの部屋を訪問

  </div>

  <div class="flex flex-col items-center">
    <img src="/tompson-sampling-and-decision-making.png" class="w-90 ml-5">
  </div>

</div>

---

# V. シミュレーション実験
提案した3つのアルゴリズムについて有効性を検証する

<div class="flex flex-row justify-between">

  <div>

  ### シミュレーション実験の条件
  <br>

  - 1台のロボットを用いて屋内を巡回
  - 図5のように, 6つの部屋、すなわち$N=6$で構成
  - 存在確率は2種類(High, Low)あり
    - 各部屋$i$で定義される真確率$TP_i$
  - 部屋2が最も存在確率が高いが, 6つの部屋間の差は小さい
    - 探索 / 活用のトレードオフ
    - もちろんロボットはこれらの真確率を知らない
  - 真の確率 $TP_i$に応じて侵入者が入ってくる
  - ロボットは10万人の侵入者を検知するまで試行を繰り返す
  - アルゴリズム1, 2, 3と環境の組み合わせに対して，5回実験

  </div>

  <div class="flex flex-col items-center">
    <img src="/patrolling-environment.png" class="w-90 ml-5">
  </div>

</div>

---

# V. シミュレーション実験
提案した3つのアルゴリズムについて有効性を検証する
### 存在確率の学習結果の考察 #1
- 図6は、巡回試行 $t\in T$における各部屋 $i$の存在確率の期待値の推移である$E_{i(t)}$を表す
- 横軸が試行回数で, ( a )から( c )は試行回数が3万回以上, ( d )から( f )は18万回以上
- 縦軸が確率を表し, ( a )から( c ), ( d )から( f )でそれぞれ範囲が異なる
- 巡回試行回数の増加に伴い, 各確率は収束していく

<div class="flex flex-col items-center">
  <img src="/patrolling-actions-of-robot-to-each-room.png" class="w-125">
</div>

---

# V. シミュレーション実験
提案した3つのアルゴリズムについて有効性を検証する
### 存在確率の学習結果の考察 #2
- 真の確率$TP_2$(=部屋2の真の存在確率)、0.6、0.1に収束
  - ロボットが最も危険な部屋の特定に成功した
- アルゴリズム2, 3の場合, ( b ), ( c ), ( e ), ( f )に示すように部屋2以外の値も真の確率に収束
- アルゴリズム1の場合, ( a ), ( d )は, 他のアルゴリズムに比べばらつきが多い

<div class="flex flex-col items-center">
  <img src="/patrolling-actions-of-robot-to-each-room.png" class="w-125">
</div>

---

# V. シミュレーション実験
提案した3つのアルゴリズムについて有効性を検証する

<div class="flex flex-row justify-between">

  <div>

  ### 存在確率の学習結果の考察 #3
  <br>

  - テーブルの各パトロール戦略に対して
    - 上段: 平均化された期待存在確率
    - 下段: エラーレート$e_i$［％］
      - $e_i=100 \times |TP_i - \bar{E_i}|/TP_i$
    - $\bar{e}$: エラー率の平均
      - $\bar{e}=1/N\sum_ie_i$

  </div>

  <div class="flex flex-col items-center">
    <img src="/table1.png" class="w-90 ml-5 mb-5">
    <img src="/table2.png" class="w-90 ml-5">
  </div>

</div>

---

# V. シミュレーション実験
提案した3つのアルゴリズムについて有効性を検証する

<div class="flex flex-row justify-between">

  <div>

  ### 存在確率の学習結果の考察 #4
  <br>
  
  - 部屋2の誤差率$e_2$は, どの戦略, 環境であれ, 存在確率の推定に成功
  - 戦略1について, 各部屋の誤差率$e_i$より
    - 表Iの部屋1, 5, 6, 表IIの部屋4, 5, 6の推定に失敗
    - Thompson Samplingアルゴリズムは, 1つの部屋しか訪問しないため
  - 戦略2, 3について, 各部屋の誤差率$e_i$より
    - 最も存在確率の高い部屋2だけでなく, 他の部屋も正しく推定できた

  </div>

  <div class="flex flex-col items-center">
    <img src="/table1.png" class="w-90 ml-5 mb-5">
    <img src="/table2.png" class="w-90 ml-5">
  </div>

</div>

---

# V. シミュレーション実験
提案した3つのアルゴリズムについて有効性を検証する

### パトロール試行結果の考察 #1
<br>

<div class="flex flex-row justify-between">

  <div>
  
  - 図7は, 各部屋に訪れた回数の合計を示す
  - 棒グラフは部屋への侵入者の検出数と非検出数
  - 各グラフのタイトルに侵入者検知数の合計を記載
  - 戦略1(Thompson Sampling)と戦略2(予想存在確率)
    - 図7(a), (d)より, 戦略1は主に部屋2を主に巡回
    - 図7(b), (e)より, 戦略2はすべての部屋を一律に巡回
      - 各部屋の存在確率が似ているため. 図5参照
    - 存在確率が高い場合は戦略1, 低い場合は戦略2が有効
  
  <p class="text-gray-300">性能限界??...</p>

  </div>

  <div class="flex flex-col items-center">
    <img src="/fig7.png" class="h-53 ml-3">
  </div>

</div>

---

# V. シミュレーション実験
提案した3つのアルゴリズムについて有効性を検証する

### パトロール試行結果の考察 #2
<br>

<div class="flex flex-row justify-between">

  <div>
  
  - 戦略3(Thompson Samplingと予想存在確率)
    - 図7(c), (f)より, 主に部屋2を巡回,
    - 他の部屋は予想存在確率に応じて一様に巡回
    - **戦略3が最も多く侵入者検知を行えた**

  </div>

  <div class="flex flex-col items-center">
    <img src="/fig7.png" class="h-53 ml-3">
  </div>

</div>

---

# V. 結論
おつかれさまでした. まとめです
<br>

- 屋内環境における一台の巡回ロボット
- 目標: 室内を巡回, できるだけ多くの侵入者を検出
  - 多腕バンディット問題
- ベイズ学習による巡回戦略を提示
- Thompson Samplingと予想存在確率の統合した戦略の有効性

---

# V. 結論
おつかれさまでした. まとめです
<br>

#### 今後の展望
- 存在確率は時間とともに変化
  - Srivastavaらの効率的なアルゴリズム[17]
- 侵入者の各部屋への滞在時間を考慮
  - スケジューリング問題
