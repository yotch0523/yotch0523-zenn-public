---
title: "【Azure Migrate】レプリケートに利用するキャッシュストレージアカウントを手動でプロビジョニングする"
emoji: "📦"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Azure"]
published: true
publication_name: "azpower"
---

# はじめに

こんにちは、AZPower 株式会社のクラウドインテグレーション部に所属している wai です。

これまでは、Azure Migrate プロジェクトで初回のレプリケート実行時に、既定でキャッシュストレージアカウントが自動的に作成される仕様になっていました。詳しくは下記の公式ドキュメントをご覧ください。

https://learn.microsoft.com/ja-jp/azure/migrate/replicate-using-expressroute#identify-the-cache-storage-account

ところが、先日移行作業中にレプリケートを試した際、新規作成のオプションが選べない状態になっていることに気づきました。（後述しますが、この仕様になったりならなかったりと不思議な挙動が観測されています。）

![](/images/azure-migrate-cache-storage-article/000_old.png)
_従来の UI：既定でキャッシュストレージアカウントをプロビジョニングできるようになっていた_

この変更により、あらかじめ Storage Account を手動でプロビジョニングする必要が出てきます。ただし、その際にはいくつかの構成が必要です。しかし、Azure ポータルのガイダンスだけではその具体的な手順が分かりにくい部分がありました。そこで、備忘録も兼ねて本記事にその手順をまとめます。

Azure Migrate が何たるかについては下記の公式記事をご参照ください。
https://learn.microsoft.com/ja-jp/azure/migrate/migrate-services-overview
簡単に言うと、オンプレミス環境、その他パブリッククラウド上のサーバー・仮想マシンを Azure 上にデプロイするための支援を施してくれるサービスです。

# 前提条件

今回の記事での前提条件は以下の通りです：

- 今回は Azure Migrate への接続方法：パブリック（プライベートエンドポイントの利用はなし）
- ExpressRoute の利用：なし

また、本記事の趣旨や検証に直接関係はありませんが、実施環境の情報を念のため記載しておきます：

- ホスト OS：Windows Server 2022 Datacenter
- ゲスト OS の仮想化技術：Hyper-V
- ゲスト OS（検出対象マシン）：Windows Server 2016

さらに、冒頭で触れた不思議な挙動の一例として、「**_キャッシュストレージアカウント_**」と表示される場合もあれば、「**_レプリケーションストレージアカウント_**」と表記される場合もあります。混乱を避けるため、本記事では従来の呼称である「**_キャッシュストレージアカウント_**」を使用します。

# 結論

先に結論を書いておきます。
キャッシュストレージ用の Storage Account をプロビジョニングする際には、以下の構成が必要です：

1. **Azure Migrate に利用する Recovery Services コンテナーのシステム割り当てマネージド ID を有効化する**
2. **上記のシステム割り当てマネージド ID に、キャッシュストレージ用の Storage Account に対する*共同作成者ロール*、および*ストレージ BLOB データ共同作成者ロール*を割り当てる**

Azure Migrate のレプリケートの仕組みを把握している方であれば、上記の構成が必要であることに見当がつくかもしれません。
https://learn.microsoft.com/ja-jp/azure/migrate/hyper-v-migration-architecture

が、表示されるエラーメッセージの内容や、その中のヘルプリンクでプライベートエンドポイントの構成に関するドキュメントへ誘導されることから、混乱する人も出てくると思います（実際、私もそうでした）。

![](/images/azure-migrate-cache-storage-article/001_error_example.png)
_必要な構成が施されていない Storage Account が選択された際のエラーメッセージ_

![](/images/azure-migrate-cache-storage-article/003_unrelated_link.png)
_エラーメッセージ内の「方法を表示」リンクの遷移先_

さて、結論を先に述べたところで、以降は構成手順について解説していきます。

# 構成手順

## 0. キャッシュストレージ用の Storage Account を作成する

はじめに Storage Account を作成してください。
もしくは既存の Storage Account を使い回しても問題ありませんが、下記のリンクに記載されている要件を満たす必要があります：
https://learn.microsoft.com/ja-jp/azure/migrate/migrate-support-matrix-hyper-v-migration#replication-storage-account-requirements

## 1. Azure Migrate に利用する Recovery Services コンテナーのシステム割り当てマネージド ID を有効化する

Azure Migrate プロジェクトの作成時に自動的に生成される Recovery Services コンテナーで、システム割り当てマネージド ID を有効化します。

![](/images/azure-migrate-cache-storage-article/006_recovery_sites_container_id_menu.png)
_Recovery Services コンテナーの Identity メニュー_

![](/images/azure-migrate-cache-storage-article/007_recovery_sites_container_said_enable.png)
_システム割り当てマネージド ID 構成画面_
「状態」項目で「オン」を選択後、「保存」ボタンをクリックしてください。
以上で完了です。

## 2. システム割り当てマネージド ID に、キャッシュストレージ用の Storage Account に対する適切なアクセス権を割り当てる

次に、システム割り当てマネージド ID に対して必要な Azure ロールを割り当てていきます。
本作業も前作業と同じ画面上で実施可能です

下記のように各種項目を設定してください。
一度の操作につき一つのロールしか割り当てられないので、合計二回割り当て操作を実施してください。

| 項目               | 設定値                                                |
| ------------------ | ----------------------------------------------------- |
| スコープ           | ストレージ                                            |
| サブスクリプション | [該当の Storage Account が存在するサブスクリプション] |
| リソース           | [該当の Storage Account]                              |
| 役割               | 共同作成者ロール                                      |

| 項目               | 設定値                                                |
| ------------------ | ----------------------------------------------------- |
| スコープ           | ストレージ                                            |
| サブスクリプション | [該当の Storage Account が存在するサブスクリプション] |
| リソース           | [該当の Storage Account]                              |
| 役割               | ストレージ BLOB データ共同作成者ロール                |

![](/images/azure-migrate-cache-storage-article/010_said_assign_button.png)
_システム割り当てマネージド ID 構成画面（有効化後は「Azure ロールの割り当て」ボタンが出現する）_

![](/images/azure-migrate-cache-storage-article/011_said_assign_role_contributor.png)
_ロール割り当ての追加画面_

> 補足
> 「役割」のセレクトボックス内にロールが一つも表示されないことがあります。
> これは、下キャプチャのように「リソース」セレクトボックスの読み込み処理が遅延していると起こります。
> ![](/images/azure-migrate-cache-storage-article/012_said_assign_role_loadingpng.png)
> 稀によくあるので、もしロールが選択できないときはのんびり待ちましょう。

お疲れ様でした、以上で構成手順はすべて完了です。

> 余談
> 同様のロールをユーザー割り当てマネージド ID に割り当て、Recovery Services コンテナーに割り当てた場合どうなるか検証したのですが、同様のエラーメッセージが表示されました。

# 再度レプリケート画面へ

ここまでのステップをこなした後に、レプリケート画面にて該当の Storage Account を選択してもエラーメッセージが表示されず、後続の構成操作を行うことができます。めでたしめでたし。
![](/images/azure-migrate-cache-storage-article/013_success.png)
_レプリケート画面のキャッシュストレージアカウント項目_

# おわりに

構成自体は割と簡単な内容であったと思います。
が、エラーメッセージを初めて見た時、「ん？Azure Migrate にアクセス権与えるの？Recovery Services コンテナじゃなく？」と少し不信感を覚えたものの、それに従って Azure Migrate が利用するサービスプリンシパルにロールを割り当てたりと無駄な遠回りをしてしまいました。今回は直感が正しかったですね。

ユーザー割り当てマネージド ID に同等の権限を与えた場合でもダメだった時は少し驚きました。どういう理屈なんだろう。
また、記事内でも触れましたが「キャッシュストレージアカウント」と「レプリケートストレージアカウント」との表記揺れや、既定でストレージアカウントをプロビジョニングする選択肢が出たり出なかったりした理由が今でも分かっていません。
両 UI が表示された時の条件は同じで、

- レプリケートを一度も行っていない状態
- 検出対象 VM は同様
- 評価アイテムも同様

この状態で時間を置いてレプリケート構成画面を何度も開き直していたら、ランダムでどちらも表示されました。一体...

とはいえ、今回新規作成を選択できない画面を引き当てたお陰で、構成方法を深掘りできたので私はとても運が良かったです。

簡単ではありますが、以上が本記事の内容です。マサカリ大歓迎です。
ではでは。
