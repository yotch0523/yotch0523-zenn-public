---
title: "Azure Load Balancerによりルーティングされたバックエンドプールインスタンスを特定する"
emoji: "👋"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Azure", "Network", "Monitoring", "Observability"]
published: true
publication_name: "azpower"
---

こんにちは、AZPower 株式会社のクラウドインテグレーション部に所属している wai です。

Azure Load Balancer（ALB）とは、所謂 OSI 参照モデルのトランスポート層（L4）で動作する負荷分散サービスです。
本サービスを用いてアプリケーションの可用性を向上させている方も多いことと思います。

https://learn.microsoft.com/ja-jp/azure/load-balancer/load-balancer-overview

他の Azure リソースと同様に、当然 ALB も Azure Monitor ソリューションにより監視することができるのですが、具体的にどのバックエンドプールインスタンスにトラフィックがルーティングされたかまで把握することができるのはご存知でしょうか？
これを利用すると、特定のマシンに対するトラフィックがある条件に達した際にアラートを発報する、といった構成が可能になりますので、そういったニーズに対応する必要がある方の参考になれば幸いです。

## 前提条件

今回の記事の前提条件は下記の通りです：

- 検証環境（リージョン：東日本リージョン）

| リソース            | SKU・サイズ | リソース名 | 備考                                                     |
| ------------------- | ----------- | ---------- | -------------------------------------------------------- |
| Azure Load Balancer | Standard    | lb-main    | フロントエンドのパブリック LB                            |
| 仮想ネットワーク    | -           | vnet-main  | 10.0.0.0/16、default というサブネット(10.0.0.0/24)を持つ |
| 仮想マシン１        | D2s_v5      | vm-app1    | IIS がインストールされたシンプルな AP サーバー           |
| 仮想マシン 2        | D2s_v5      | vm-app2    | IIS がインストールされたシンプルな AP サーバー           |

![zenn_alb_architecture.png](/images/alb-backend-monitor/zenn_alb_architecture.png)

- ALB には、パブリック IP の 80 番ポートへの受信トラフィックを、default サブネット内にある 2 つの VM に負荷分散するよう設定された負荷分散規則が構成されている
- 東日本リージョンに Log Analytics ワークスペースが既にデプロイされている

同一の受け口に対するリクエストが二つの VM のいずれかにルーティングされるので、エンドポイントがどこかという情報だけでは振り分け先が判別できないですね。

続いてどのような構成が必要で、どのようなフォーマットで分析情報が取れるのかをこれから解説していきます。

## 結論

今回も時間がない方に向けて先にやるべきことを記載します：

1. **対象のリージョンにて仮想ネットワークフローログを作成する**
2. **Log Analytics ワークスペースにて*NTANetAnalytics*テーブルにクエリを投げる**

ネットワークフローログを作成し、必要な情報が Log Analytics ワークスペースに集積されるようにします。記録されたログに対してクエリを発行し、ALB のルーティング先バックエンドプールインスタンスを特定します。

以降は、その詳細な手順について解説していきます。

## 手順

### 1. 仮想ネットワークフローログの作成

そもそも仮想ネットワークフローログとは、仮想ネットワークを通過するトラフィックに関するログを記録する機能です。

https://learn.microsoft.com/ja-jp/azure/network-watcher/vnet-flow-logs-overview

本機能は Azure Network Watcher の機能の一つであるため、まずは Azure ポータルから「**_Network Watcher_**」を検索し、クリックしてください。

![image.png](/images/alb-backend-monitor/image.png)

すると、サイドバーに「**_フローログ_**」メニューがあるため、そちらクリック。

![image.png](/images/alb-backend-monitor/image1.png)

お馴染みの位置に「**_作成_**」ボタンがあるためそちらをクリック。

![image.png](/images/alb-backend-monitor/image2.png)

#### 1-1. 構成値：基本タブ

これより各種構成項目を埋めていきます。
先に全ての構成項目とその値を下表に示します。

##### 基本タブの構成項目

| 項目 | 構成値 |
| ------------------ | ---------------------------------------------- |
| サブスクリプション | 検証環境がデプロイされているサブスクリプション |
| フローログのタイプ | 仮想ネットワーク |
| ターゲットリソースの選択 | 仮想ネットワーク |
| ストレージアカウント | 分析情報の記録に利用できる適当な Storage Account |
| リテンション期間 | 適当な値、本検証では 30 日としています |

##### ターゲットリソース選択画面の構成項目

| 項目 | 構成値 |
| -- | -- |
| 仮想ネットワークを選択します | vnet-main |

※「仮想ネットワークを選択します」画面は、基本タブにて「ターゲットリソースの選択」で「仮想ネットワーク」を選択すると起動されます。

![image.png](/images/alb-backend-monitor/image3.png)
_基本タブ：サブスクリプション、フローログのタイプ、ターゲットリソースのタイプ_

![image.png](/images/alb-backend-monitor/image4.png)
_基本タブ > ターゲットリソース：ネットワークフローログを適用する仮想ネットワークを選択する_

![image.png](/images/alb-backend-monitor/image5.png)
_基本タブ：ストレージアカウント、リテンション期間_

仮想ネットワークフローログは Storage Account に対して出力されます。
この出力先として指定するにはいくつか条件がありますので、下記のドキュメントを参考にしてください。

https://learn.microsoft.com/ja-jp/azure/network-watcher/vnet-flow-logs-overview#considerations-for-virtual-network-flow-logs

実はこの構成だけでも欲しい情報を取得することはできるのですが、分析を行うには少々使い勝手が悪いので次の分析タブの構成を実施します。

#### 1-2. 構成値：分析タブ

続いて分析に関する構成を行います。
まず、構成項目の一覧を下表に示します。

##### 分析タブの構成項目

| 項目 | 構成値 |
| ------------------ | ---------------------------------------------- |
| Traffic Analytics を有効にする | チェック |
| Traffic Analytics の処理間隔 | 10 分ごと |
| サブスクリプション | 検証環境がデプロイされているサブスクリプション |
| Log Analytics ワークスペース | 前提条件記載の Log Analytics ワークスペース |

![image.png](/images/alb-backend-monitor/image6.png)

ここでの最大のポイントは **_Traffic Analytics_** の有効化です。

Traffic Analytics とは、Azure サブスクリプション全体のネットワークトラフィックに関する分析情報を提供する Azure Network Watcher のソリューションです。

https://learn.microsoft.com/ja-jp/azure/network-watcher/traffic-analytics?tabs=Americas

上記のドキュメントにも記載されているように、Traffic Analytics は仮想ネットワークフローログに記録されたデータを読み取ります。また、Traffic Analytics には Log Analytics ワークスペースが構成要素として含まれているため、Traffic Analytics を有効化することでフローログのデータを Log Analytics で分析・可視化できるようになります。

以上で必要な構成は完了です。（「**_タグ_**」タブでは任意の設定を行ってください）
「**_確認と作成_**」タブにて構成値を確認後、仮想ネットワークフローログを作成してください。

### 2. クエリ

全ての構成を実施後、ALB のパブリック IP アドレスに向けて 80 番ポートで何度かアクセスしてください。
すると、しばらくした後に Log Analytics ワークスペースに**_NTANetAnalytics_**テーブルが作成されるため、そのテーブルに対しクエリを投げます。

![image.png](/images/alb-backend-monitor/image7.png)

今回は下記のように、 DestLoadBalancer に対象の ALB のリソース名の文字列を含むものでフィルタする Kusto クエリを実行しました。

```kusto
NTANetAnalytics
| where DestLoadBalancer contains "lb-main"
```

![image.png](/images/alb-backend-monitor/image8.png)

すると上キャプチャのようにクエリ結果が出てきますが、少し右にスクロールすると Dest\*\*\*といったルーティング先に関する情報を格納したカラムが見えてきます。

![image.png](/images/alb-backend-monitor/image9.png)

DestVm を見ると、vm-app1、vm-app2 のどちらにルーティングされたのか一目瞭然ですね。目的は達成されました。

![image.png](/images/alb-backend-monitor/image10.png)

余談ですが、リクエスト元から先へのパケット数、バイト数といったデータも格納されています。
アラートに使えそうなデータがないか調べてみると楽しいと思います。

## おわりに

以上が本記事の内容です。
冒頭でも触れましたが、今回の構成は特定のバックエンドプールインスタンスへのトラフィックの動向を把握し、必要に応じてアラートやさらなる対策を講じる上で有効です。また、Traffic Analytics を活用することで、ネットワーク全体のトラフィックパターンを俯瞰し、さらなる最適化の可能性を模索することもできます。

恒例でマサカリ大歓迎です。
ではでは。

## 参考記事

[Azure Load Balancer を監視する](https://learn.microsoft.com/ja-jp/azure/load-balancer/monitor-load-balancer)
