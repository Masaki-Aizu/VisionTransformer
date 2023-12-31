# VisionTransformer
## ポイント
- 初期Transformerに**少しだけ改良**を加え、画像分類に適用できるようにした
- 画像を複数のパッチに分割し、パッチを単語のように学習させる（パッチの数、サイズは固定）
- 分割時、画像位置を学習させるために、**2DのPositionalEmmbeding**を行う
## 画像をTransformerの入力へ変換する
1. (H, W, C)の画像を2Dパッチに分割する
- $Image: x ∈ R^{H×W×C}$ → $2D patches: x_p ∈ R^{N×(P^2·C)}$
- $(H, W) $は学習データの解像度, $C$ は画像のチャンネル数, **(P, P)** はパッチの解像度, $N = HW/P^2$はパッチ数
- この時パッチがTransformerの入力となる
2. 画像パッチをD次元ベクトルに変換する
- D個のニューロンをもつMLPから、パッチの埋め込み表現を得る
3. PositinalEmbeddingを加算する
- 2, 3は下記の等式で表現できる
- $z0 = [x_{class}; x^1_pE; x^2_pE; · · · ; x^N_p E] + E_{pos},    E ∈ R^{(P2·C)×D}, E_{pos} ∈ R^{(N+1)×D}$
- ここで$x_{class}$は、BERTの[class] トークンと同様にクラス分類のための特徴量トークン。学習可能なパラメータで、パッチのシーケンスの先頭に追加する
4. LayerNormalization後、Multi-Head=Attention層へ（この時スキップ結合を適用）
5. LayerNormalization後、D次元ベクトルを出力するMLP層へ（この時スキップ結合を適用）
6. この操作を繰り返し、エンコーダー層最終出力ベクトル:$Z^0_L$をMLPHeadへ投入し、クラス分類
-　※2をCNNモデルを使用し、分散表現を得てもよい。これを論文では**HybridArchitecture**を呼んでいる
## アーキテクチャの利点
1. Vision Transformer には、CNN よりも画像固有の誘導バイアスがはるかに少ない
- CNN では、画像の局所性(2次元近傍構造)および並進等分散性がモデル全体の各レイヤーに焼きつけらけてしまう
- ViT では、セルフアテンション層により、局所的な特徴ではなく、グローバルな特徴量を得ることができる
##　モデル
### アーキテクチャ
<img alt="ViT" src=./image/vit.png></img>
### モデルコンフィグ
<img alt="ViT config" src=./image/vit_config.png></img>
## ファインチューニング
- 通常、大規模なデータセットで ViT を事前トレーニングし、解きたいタスクに合わせてファインチューニングする
- このために、事前トレーニングされた予測ヘッドを削除し、ゼロ初期化された $D × K$ フィードフォワード層を接続する
- ここで、$K はダウンストリーム クラスの数
- 多くの場合、事前トレーニングよりも高い解像度で微調整する方が有益(Touvron et al., 2019; Kolesnikov et al., 2020)
- だが、事前にトレーニングされた位置の埋め込みは意味を持たなくなる可能性がある
-  したがって、元の画像内の位置に応じて、事前にトレーニングされた位置埋め込みの 2D 補間が必要
## 学習方法
### 事前学習
- Adam: β1 = 0.9, β2 = 0.999
- batch size: 4096
- high weight decay of 0.1(転移学習で有効なパラメータらしい)
### ファインチューニング
- SGD with momentum
- batch size: 512
- 事前学習時より、画像解像度を大きくする
- ViTのMLPヘッドを取り替える。この時重みを0で初期化
## 参考
1. https://qiita.com/omiita/items/0049ade809c4817670d7
2. https://openreview.net/pdf?id=YicbFdNTTy
