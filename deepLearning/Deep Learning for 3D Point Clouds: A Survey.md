## [Deep Learning for 3D Point Clouds: A Survey](https://github.com/QingyongHu/SoTA-Point-Cloud)

## Abstract

Deep learningを用いた, 3D shape classification, 3D object detection and tracking, 3D point segmentationに焦点を当てたサーベイ。


## INTRO

[^1], [^2]と比較して、Deeplearningを用いた手法に着目している。

## データセット

おすすめのデータセット

+ [ModelNet](http://modelnet.cs.princeton.edu/)
+ [ShapeNet](https://www.shapenet.org/)
+ [ScanNet](http://www.scan-net.org/)
+ [Semantic3D](http://semantic3d.net/)
+ [KITTI](http://www.cvlibs.net/datasets/kitti/)

## 3D shape classification

1. 手法の一般的な流れ  
   各点の埋め込みを学習し、集計法を用いて点群全体からグローバルな形状上の埋め込みを抽出する。(埋め込み?? Embedding)。その後、FC層で分類結果を得る。
2. 主に、Projection-based methodsとPoint-based methods
   + 投影ベース - 構造化されていない点群を中間正規表現に投影し、その後、2D or 3D畳み込みを活用して分類
   + 点ベース - ボクセル化や投影を行わず、直接ポイントクラウドに対して分類する手法 (情報損失が少ない)。
  
### 投影ベースの手法
  点群を様々な表現モダリティに変換。

1. マルチビュー表現  
   3D物体を複数のビューに投影し、対応するビューごとの特徴を抽出し、抽出した特徴を合算して正確なオブジェクトの認識を達成する。
   #### 課題
   ビュー単位の特徴を識別可能なグローバル表現に集約すること => 最大プーリング(MVCNN)、ビューごとの関係性など内界の関係を説明するためにRelation Networkを構築([^5])など
2. Volumetric (ボクセル?) 表現
   初期は、3D点群のボクセル表現に基づいた3DCNNを適用をして達成していた。その後、体積専有(0〜1表現)ネットワークであるVoxNetが導入された。=> 解像度の増加により必要なメモリ増加し、スケールできない。
   そのため、階層的でコンパクトなグラフ構造(Octreeなど)を導入して、計算コストとメモリコストを削減。 => OctNet, PointGrid

### 点ベース

各点の機能学習に使用されるネットワークアーキテクチャが複数ある。

1. Pointwise MLP(多層パーセプトロン) Network
対称関数を用いてグローバルな特徴を集約することで、点群の順列不変性を実現。しかし、点群間の幾何学的関係を完全には考慮しない。

PointNet - 複数のMLP層でPointwiseな特徴を学習し、最大プーリング層でグローバル特徴を抽出する。

  + Qiらが、順列不変性を達成するためには、全ての表現を集合し、非線形変換を適用することが重要だと実証。
  + 各点が独立して学習されるため、点群間のローカル構造情報は取得されない。　==> PointNet++ が提案される。

PointNet++ - 抽象化のレイヤーを層構造とすること(sampling, grouping, PointNet layer)で、ローカルの幾何学的構造から特徴を学習し、ローカル特徴をレイヤーごとに抽象化できるようになる。
[多くの派生ネットワークがある](https://qiita.com/arutema47/items/cda262c61baa953a97e9)

2. Convolutionベースのネットワーク
   畳み込み時のカーネルを点群用に工夫し、連続畳み込みネットワークと離散畳込みネットワークという分野に分かれる。

3. Graph-based ネットワーク
   各点をグラフの頂点とみなし各点の隣接点に基づいてグラフの有向エッジを生成。[特徴の学習を空間またはスペクトル領域で実行する。](https://www.arxiv-vanity.com/papers/1704.02901/)

4. Data Indexing - based ネットワーク
   異なるデータ構造(例えば octreeとkd-treeなど)に基づいて構築される。

### その他のネットワーク
  + [Y. Zhao, T. Birdal, H. Deng, and F. Tombari, “3D point capsule networks,” in CVPR, 2019.](https://github.com/yongheng1991/3D-point-capsule-networks)
    3D点群の表現学習のために教師なし自動エンコーダを提案。

### 考察

+ Pointwise networkは、他のタイプのネットワークの基礎（土台）となっている。
+ 畳み込みベースのネットワークは、不規則な点群に対して優れたパフォーマンスを実現可能
+ 基本的には、ほとんどのネットワークでは、点群を固定された小さいサイズにダウンサンプリングする必要があり、形状の詳細は破棄している。（研究は、初期段階)

## 3D Point Cloud Segmentation

セグメンテーションのレベルとして、semantic segmentation(シーンレベル), instance segmentation(オブジェクトレベル), part segmentation(パーツレベル)の3つのカテゴリに分類できる。

### 投影ベース

マルチビュー表現(RGBが必要), 球面表現(中間表現の離散化エラーやオクルージョンなどの問題がある), 体積表現など
RangeNet++として、リアルタイムなLiDAR点群のセマンティックセグメンテーションの手法が提案されている。この中間表現は必然的に、離散化エラーやオクルージョンなどのいくつかの問題をもたらす可能性がある。

### ポイントベース
  Pointwise MLP, Conv, RNNなど、、、

## Instance Segmentation !!!

semantic segmentationと比較し、ポイントのより正確で詳細な推論を必要とするため、困難。特に、異なるクラスタのポイントを区別するだけでなく、同じクラスタを持つポイントの区別も必要。Proposal-based methodとproposal-free methodsの２つの研究分野がある。

### Proposal-based methods

セグメンテーションの問題を、3Dオブジェクト検出とインスタンスマスク予測の２つのサブタスクに変換。(3D-BoNet:室内適用, [Instance segmentation of lidar point clouds](https://github.com/PRBonn/lidar-bonnetal): 疎な点群データであるLiderによる点群に適用)

### Proposal-free methods

提案ベースでは、多段階トレーニングが必要であり、計算コストも高いので、オブジェクト検出を省き、直接インスタンスセグメンテーションを行う。
殆どの手法では、同じインスタンスに属する点は類似した特徴を持つべきであるという仮定に基づいている。
先駆的な論文としてい、[Similarity Group Proposal Network](https://github.com/laughtervv/SGPN) (SGPN)を導入し、最初に各点の特徴とセマンティックマップを学習し、その後、特徴間の類似性を表す類似性まトリックすを導入する。
計算コストは低いが、明示的にオブジェクト境界を明示的に検出しないため、グループ化された状態のインスタンスセグメントの個別性(オブジェクト性)が低くなる。



## 参照 - Survey

[^1]: A. Ioannidou, E. Chatzilari, S. Nikolopoulos, and I. Kompatsiaris, “Deep learning advances in computer vision with 3D data: A survey,” ACM Computing Surveys, vol. 50, no. 2, p. 20, 2017.   
[^2]: E. Ahmed, A. Saint, A. E. R. Shabayek, K. Cherenkova, R. Das, G.Gusev,D.Aouada,andB.Ottersten,“Deeplearningadvances on different 3D data representations: A survey,” arXiv preprint arXiv:1808.01462, 2018.   


[^3]: Y. Xie, J. Tian, and X. X. Zhu, “A review of point cloud semantic 4,7
segmentation,” arXiv preprint arXiv:1908.08854, 2019.  
[^4]: M. M. Rahman, Y. Tan, J. Xue, and K. Lu, “Recent advances in 3D object detection in the era of deep neural networks: A survey,”
IEEE TIP, 2019.  

[^5]: Z. Yang and L. Wang, “Learning relationships for multi-view 3D object recognition,” in ICCV, 2019, pp. 7505–7514.