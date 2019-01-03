## 4. 通信データ・クラスタリング
　クラスタリング（Clustering）とは**教師なし学習**の1種で、ラベル付けされていないデータ集合に対して、近い属性を持つデータをグループ化（クラスタリング）する手法です。クラスタ解析、クラスタ分析とも呼ばれます。  
　クラスタリングは大きく下記の2種類に分類されます。  

 * 階層的クラスタリング  
 クラスタリングの結果を木構造で出力。縦方向の深さは類似度を示し、深いほど類似度が低く、浅いほど類似度が高いことを示す。代表的な手法としてWard法がある。

 * 非階層的クラスタリング  
 予め決められたクラスタ数に従って、近い属性のデータをグループ化する。代表的な手法にK平均法（K-Means）がある。

　クラスタリングは様々なタスクに応用できますが、本ブログではホストへのアクセスデータ（通信データ）を**K平均法**でクラスタリングし、**通信の種類を推定**する例を挙げます。ここで通信の種類とは、正常通信は勿論のこと、ホストへの攻撃を示唆する通信も含まれます。  

| 教師なし学習（Unsupervised Learning）|
|:--------------------------|
| データ集合は存在するものの、各データに答え（ラベル）が付与されていない学習データに対し、何らかの構造や法則を見つけるための手法。|

　以下、K平均法の簡単な解説を行った後、Pythonによる実装コードの例示と実行結果および考察を示していきます。  

### 4.1. K平均法
　K平均法は、クラスタの平均を使用し、予め与えられたクラスタ数（K）のクラスタを作成することで、データを分類するアルゴリズムです。  
　少し分かり易くするために、図を用いてクラスタリングの手順を説明します。

 1. 分析対象のデータをプロット  
  ![データのプロット](.\img\clustering1.png)  

 2. K個の点の初期値をランダムな値（m[0]～m[2]）に決定  
  ![初期化](.\img\clustering2.png)  

 3. 各データの所属を最も距離の近いm[k]（重心点）に決定  
  ![重心点の決定](.\img\clustering3.png)  
  各重心点を中心に仮のクラスタを作る。  

 4. m[k]を各クラスタに所属するデータの**平均値**に更新  
  ![重心点の更新](.\img\clustering4.png)  
  各重心点が移動する。  

 5. 新しいm[k]を中心にクラスタの再作成  
  ![収束テスト](.\img\clustering5.png)  
  収束条件を満たしたらクラスタリング終了（6）。  
  収束しない場合は、m[k]の更新とクラスタ作成を繰り返す（3～5）。  

 6. クラスタリング完了  
  ![収束完了](.\img\clustering6.png)  

　上記のように、重心点の更新とクラスタ作成を繰り返すことで、データを複数のクラスタに分割することができます。K平均法では、予め与えるKの数に応じてクラスタ数が決まるため、Kを適切に設定することが重要となります。また、クラスタリングではデータを複数のクラスタに分割するのみであり、**各クラスタが何を意味しているのか**までは示すことはできません。よって、各クラスタの意味は人間等が分析して判断する必要があります。  
　※本例では、データの次元は2次元ですが、3次元以上のデータも分類可能です。  

　K平均法の主な特徴を纏めると以下のとおりです。  

| K平均法の主な特徴　　　　　|
|:--------------------------|
| 教師なし学習　　　　　　　　|
| データの分類が可能　　　　　|
| Kを適切に設定する必要がある |
| クラスタの意味は人間が判断する |

　次節では、K平均法を使用して通信データをクラスタリングする手順を解説します。  

### 4.2. K平均法を用いた通信データ・クラスタリング
　本ブログでは、下記の機能を持つクラスタリングシステムを構築します。  

 * 通信データを複数のクラスタに分類
 * 各クラスタの成分を可視化

　本システムは、通信データを予め決められた数（K）のクラスタに分類します。そして、クラスタの意味を人間が分析し易いように、各クラスタの成分をグラフ化します。  

　以下、システムの構築手順を順を追って解説します。  

#### 4.2.1. 分析対象データの準備
　分析対象の通信データを準備します。  
　本来であれば、自社環境で収集したリアルな通信データを使用することが好ましいですが、ここでも「[KDD Cup 1999](http://kdd.ics.uci.edu/databases/kddcup99/kddcup99.html)」を使用します。  

　上記のサイトから「kddcup.data_10_percent.gz」をダウンロードし、42カラム目のラベルを全削除し、かつデータ量を150件程度に削減します（計算時間の短縮のため）。  
　すると、以下のようなファイル内容になります。  

```
duration,protocol_type,service,flag,src_bytes,dst_bytes,land,wrong_fragment,urgent,hot,num_failed_logins,logged_in,num_compromised,root_shell,su_attempted,num_root,num_file_creations,num_shells,num_access_files,num_outbound_cmds,is_host_login,is_guest_login,count,srv_count,serror_rate,srv_serror_rate,rerror_rate,srv_rerror_rate,same_srv_rate,diff_srv_rate,srv_diff_host_rate,dst_host_count,dst_host_srv_count,dst_host_same_srv_rate,dst_host_diff_srv_rate,dst_host_same_src_port_rate,dst_host_srv_diff_host_rate,dst_host_serror_rate,dst_host_srv_serror_rate,dst_host_rerror_rate,dst_host_srv_rerror_rate
0,tcp,http,SF,181,5450,0,0,0,0,0,1,0,0,0,0,0,0,0,0,0,0,8,8,0,0,0,0,1,0,0,9,9,1,0,0.11,0,0,0,0,0
0,tcp,http,SF,239,486,0,0,0,0,0,1,0,0,0,0,0,0,0,0,0,0,8,8,0,0,0,0,1,0,0,19,19,1,0,0.05,0,0,0,0,0
0,tcp,http,SF,235,1337,0,0,0,0,0,1,0,0,0,0,0,0,0,0,0,0,8,8,0,0,0,0,1,0,0,29,29,1,0,0.03,0,0,0,0,0
0,tcp,http,SF,219,1337,0,0,0,0,0,1,0,0,0,0,0,0,0,0,0,0,6,6,0,0,0,0,1,0,0,39,39,1,0,0.03,0,0,0,0,0
0,tcp,http,SF,217,2032,0,0,0,0,0,1,0,0,0,0,0,0,0,0,0,0,6,6,0,0,0,0,1,0,0,49,49,1,0,0.02,0,0,0,0,0
0,tcp,http,SF,217,2032,0,0,0,0,0,1,0,0,0,0,0,0,0,0,0,0,6,6,0,0,0,0,1,0,0,59,59,1,0,0.02,0,0,0,0,0
0,tcp,http,SF,212,1940,0,0,0,0,0,1,0,0,0,0,0,0,0,0,0,0,1,2,0,0,0,0,1,0,1,1,69,1,0,1,0.04,0,0,0,0

...snip...

162,tcp,telnet,SF,1567,2738,0,0,0,3,0,1,4,1,0,0,1,0,0,0,0,0,1,1,0,0,0,0,1,0,0,4,4,1,0,0.25,0,0,0,0,0
127,tcp,telnet,SF,1567,2736,0,0,0,1,0,1,0,0,0,0,1,0,0,0,0,0,83,1,0.99,0,0,0,0.01,0.08,0,5,5,1,0,0.2,0,0,0,0,0
321,tcp,telnet,RSTO,1506,1887,0,0,0,0,0,1,0,0,0,0,1,0,0,0,0,0,151,1,0.99,0,0.01,1,0.01,0.06,0,6,6,1,0,0.17,0,0,0,0.17,0.17
45,tcp,telnet,SF,2336,4201,0,0,0,3,0,1,1,1,0,0,0,0,0,0,0,0,2,1,0,0,0.5,0,0.5,1,0,7,7,1,0,0.14,0,0,0,0.14,0.14
176,tcp,telnet,SF,1559,2732,0,0,0,3,0,1,4,1,0,0,1,0,0,0,0,0,1,1,0,0,0,0,1,0,0,8,8,1,0,0.12,0,0,0,0.12,0.12
61,tcp,telnet,SF,2336,4194,0,0,0,3,0,1,1,1,0,0,0,0,0,0,0,0,1,1,0,0,0,0,1,0,0,9,9,1,0,0.11,0,0,0,0.11,0.11
47,tcp,telnet,SF,2402,3816,0,0,0,3,0,1,2,1,0,0,0,0,0,0,0,0,1,1,0,0,0,0,1,0,0,10,10,1,0,0.1,0,0,0,0.1,0.1
```

　このファイルは、各行が1つの通信データを表しており、「1～41カラム」は各データの特徴量を表しています。  
　※各特徴量の解説は[こちら](http://kdd.ics.uci.edu/databases/kddcup99/task.html)を参照してください。  

　この状態のファイルを「kddcup.data_small.csv」として保存します。  

#### 4.2.2. クラスタ数の決定
　K平均法では、クラスタ数（K）を予め決める必要があります。当然ながら、データを眺めるだけで適切なクラスタ数を求めることは不可能ですので、何らかの方法で目安を付けます。  
　そこで今回は、**シルエット分析**と呼ばれる手法を使用してクラスタ数の目安を付ける事にします。  

| シルエット分析（Silhouette Analysis）|
|:--------------------------|
| クラスタ内のサンプル密度（凝集度）を可視化し、クラスタ間の距離が離れている場合に最適なクラスタ数とする。|

　以下はクラスタ数を5に設定してシルエット分析した例を示しています。  

 <img src=".\img\cluster_silhouette5.png" height=400 width=400>

　横軸のSilhouette Coefficientは、サンプルが近隣のクラスタから離れていることを示します。値が1に近いほど他のクラスタと離れていることを意味します。また、シルエットの厚さはクラスタのサイズを示します。このことから、クラスタ数が適切な場合、各クラスタは同様の厚さになり、Silhouette Coefficientも1に近い値になります。  
　※なお、シルエット分析はscikit-learnで容易に実装可能です。サンプルコードは[こちら](http://scikit-learn.org/stable/auto_examples/cluster/plot_kmeans_silhouette_analysis.html)をご参照ください。  

　以下は、今回の分析対象データ「kddcup.data_small.csv」に対し、クラスタ数を2～6に変化させながら分析した結果を示しています。  

 * クラスタ数=2  
 <img src=".\img\cluster_silhouette2.png" height=400 width=400>

 * クラスタ数=3  
 <img src=".\img\cluster_silhouette3.png" height=400 width=400>

 * クラスタ数=4  
 <img src=".\img\cluster_silhouette4.png" height=400 width=400>

 * クラスタ数=5  
 <img src=".\img\cluster_silhouette5.png" height=400 width=400>

 * クラスタ数=6  
 <img src=".\img\cluster_silhouette6.png" height=400 width=400>

　この結果から分かるように、クラスタ数が5の場合に良いシルエットが現れているように見えます。よって、今回は「クラスタ数を5」にしてクラスタリングを行うことにします。  

　次節では実際にサンプルコードを実行し、テストデータがどのようにクラスタリングされるのか検証します。  

### 4.3. サンプルコード及び実行結果
#### 4.3.1. サンプルコード
　本ブログではPython3を使用し、簡易的な分析システムを実装しました。  
　本システムの大まかな処理フローは以下のとおりです。  

 1. 分析対象データのロード
 2. K平均法によるクラスタリング
 3. 分析結果の可視化

```
# -*- coding: utf-8 -*-
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
from sklearn.cluster import KMeans

# Cluster number using k-means.
CLUSTER_NUM = 5

# Load data.
df_kddcup = pd.read_csv('.\\dataset\\kddcup.data_small.csv')
df_kddcup = df_kddcup.iloc[:, [0, 7, 10, 11, 13, 35, 37, 39]]

# Normalization.
df_kddcup = (df_kddcup - df_kddcup.mean()) / df_kddcup.mean()

# Transpose of matrix.
kddcup_array = np.array([df_kddcup['duration'].tolist(),
                      df_kddcup['wrong_fragment'].tolist(),
                      df_kddcup['num_failed_logins'].tolist(),
                      df_kddcup['logged_in'].tolist(),
                      df_kddcup['root_shell'].tolist(),
                      df_kddcup['dst_host_same_src_port_rate'].tolist(),
                      df_kddcup['dst_host_serror_rate'].tolist(),
                      df_kddcup['dst_host_rerror_rate'].tolist(),
                      ], np.float)
kddcup_array = kddcup_array.T

# Clustering.
pred = KMeans(n_clusters=CLUSTER_NUM).fit_predict(kddcup_array)
df_kddcup['cluster_id'] = pred
print(df_kddcup)
print(df_kddcup['cluster_id'].value_counts())

# Visualization using matplotlib.
cluster_info = pd.DataFrame()
for i in range(CLUSTER_NUM):
    cluster_info['cluster' + str(i)] = df_kddcup[df_kddcup['cluster_id'] == i].mean()
cluster_info = cluster_info.drop('cluster_id')
kdd_plot = cluster_info.T.plot(kind='bar', stacked=True, title="Mean Value of Clusters")
kdd_plot.set_xticklabels(kdd_plot.xaxis.get_majorticklabels(), rotation=0)

print('finish!!')
```

#### 4.3.2. コード解説
　本節では、機械学習部分に着目して簡単にコードを解説します。  

　今回はK平均法の実装に、機械学習ライブラリの**scikit-learn**を使用しました。  
　scikit-learnの使用方法は[公式ドキュメント](http://scikit-learn.org/)を参照のこと。  

##### パッケージのインポート
```
import matplotlib.pyplot as plt
from sklearn.cluster import KMeans
```

　scikit-learnのK平均法パッケージ「`KMeans`」をインポートします。  
　このパッケージには、K平均法を行うための様々なクラスが収録されています。  

　また、クラスタリング結果を可視化するためのパッケージ「`matplotlib`」も併せてインポートします。  

##### クラスタ数の設定
```
CLUSTER_NUM = 5
```

　前節で述べたように、クラスタ数を「5」に設定します。

##### 分析対象データのロード
```
# Load data.
df_kddcup = pd.read_csv('.\\dataset\\kddcup.data_small.csv')
df_kddcup = df_kddcup.iloc[:, [0, 7, 10, 11, 13, 35, 37, 39]]

# Normalization.
df_kddcup = (df_kddcup - df_kddcup.mean()) / df_kddcup.mean()
```

　分析対象データ「kddcup.data_small.csv」からデータを取得します（`df_kddcup = df_kddcup.iloc[:, [0, 7, 10, 11, 13, 35, 37, 39]]`）。今回分析に使用する特徴量は、侵入検知と同様に以下にします。  

| Feature | Description |
|:------------|:------------|
| dst_host_serror_rate | SYNエラー率。 |
| dst_host_same_src_port_rate | 同一ポートに対する接続率。 |
| wrong_fragment | 誤りのあるfragment数。 |
| duration | ホストへの接続時間（sec）。 |
| logged_in | ログイン成功有無。 |
| root_shell | root shellの取得有無。 |
| dst_host_rerror_rate | REJエラー率。 |
| num_failed_logins | ログイン試行の失敗回数。 |

　また、分析精度を上げるために、各特徴量のデータ値を正規化します（`df_kddcup = (df_kddcup - df_kddcup.mean()) / df_kddcup.mean()`）。  

##### データの行列変換
```
kddcup_array = np.array([df_kddcup['duration'].tolist(),
                      df_kddcup['wrong_fragment'].tolist(),
                      df_kddcup['num_failed_logins'].tolist(),
                      df_kddcup['logged_in'].tolist(),
                      df_kddcup['root_shell'].tolist(),
                      df_kddcup['dst_host_same_src_port_rate'].tolist(),
                      df_kddcup['dst_host_serror_rate'].tolist(),
                      df_kddcup['dst_host_rerror_rate'].tolist(),
                      ], np.float)
kddcup_array = kddcup_array.T
```

　後述するクラスタリング実行用のメソッド`fit_predict`は引数にmatrixを取るため、`pandas`で読み込んだ分析対象データを`numpy`の行列に変換します。  

##### クラスタリングの実行
```
pred = KMeans(n_clusters=CLUSTER_NUM).fit_predict(kddcup_array)
```

　`KMeans(n_clusters=CLUSTER_NUM)`でK平均法モデルを定義します。  
　この際、引数「`n_clusters`」にクラスタ数を設定します。  

　`KMeans`の`fit_predict`メソッドの引数として行列に変換した分析対象データを渡すことで、クラスタリングが実行されます。  

##### クラスタリング結果の可視化
```
cluster_info = pd.DataFrame()
for i in range(CLUSTER_NUM):
    cluster_info['cluster' + str(i)] = df_kddcup[df_kddcup['cluster_id'] == i].mean()
cluster_info = cluster_info.drop('cluster_id')
kdd_plot = cluster_info.T.plot(kind='bar', stacked=True, title="Mean Value of Clusters")
kdd_plot.set_xticklabels(kdd_plot.xaxis.get_majorticklabels(), rotation=0)
```

　クラスタリング結果を、`matplotlib`の積み上げ棒グラフで出力します。  

#### 4.3.3. 実行結果

　このサンプルコードを実行すると、以下のグラフが表示されます。  

 <img src=".\img\clustering_result.png" height=500>

　分析対象データから5つのクラスタが作成され、また各クラスタの特徴量の平均値が色付きで可視化されていることが分かります。  
　クラスタリングでできるのはここまでであり、各クラスタの意味は人が分析して推定します。それでは、各クラスタを一つずつ見ていきましょう。  

 * cluster1  
 cluster1は紫と青の特徴量、すなわち「root_shell」「duration」の平均値が他クラスタより大きいことが分かります。ホストへの接続時間が長く、root権限が与えられるケースが多いことから、この通信データは「Buffer Overflow」が実行されたものと推定されます。  

 * cluster2  
 cluster2は桃色と茶色の特徴量、すなわち「dst_host_serror_rate」「dst_host_same_src_port_rate」の平均値が他クラスタより大きいことが分かります。SYNエラー率が高く、同一ポートに対する接続割合が多いことから、この通信データは「Nmapによる探索」または「SYN Flood」等が実行されたものと推定されます。  

 * cluster3  
 cluster3は灰色と緑色の特徴量、すなわち「dst_host_rerror_rate」「num_failed_logins」の平均値が他クラスタより大きいことが分かります。REJエラー率が高く、ログイン試行の失敗回数が多いことから、この通信データは「パスワード推測」が試行されたものと推定されます。  

 * cluster4  
 cluster4は橙色の特徴量、すなわち「wrong_fragment」の平均値が他クラスタより大きいことが分かります。誤りのあるフラグメントが多いことから、この通信データは「Teardrop」が試行されたものと推定されます。  

 * cluster0  
 順番が前後しましたが、cluster0の特徴量には偏りが殆どありません。よって、この通信データは正常通信のものと推定されます。  

　このように、クラスタリング結果を可視化することで、分析対象データには複数の攻撃の痕跡が含まれていることが分かりました。この分析結果を基にログの更なる深堀、または同時刻に記録された他のログを詳細に分析することで、攻撃の種類を特定することができるかもしれません。  

### 4.4. 動作条件
 * Python 3.6.1（Anaconda3）
 * pandas 0.20.3
 * numpy 1.11.3
 * matplotlib 2.0.2
 * scikit-learn 0.19.0