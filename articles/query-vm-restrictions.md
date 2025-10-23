---
title: "【Azure】仮想マシンのデプロイ制限を調べる方法"
emoji: "🔍"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Azure", "Microsoft", "Bash", "PowerShell", "CLI"]
published: true
publication_name: "azpower"
---

こんにちは、AZPower 株式会社のクラウドインテグレーション部に所属している wai です。

皆さんはAzureに仮想マシンをデプロイしようとしたとき、クォータのせいでデプロイが失敗してしまった経験はありませんか？
単なる技術検証の中で起きたならまだ良いのですが、お客様とコミュニケーションを重ね、やっとの思いで設計が固まった末に起きたら少々大変なことになります。

そんな大惨事を防ぐ方法が実はあるので、今回はその方法を共有します。
本記事は5分程度でサクッと読めるので暇つぶしにでも読んでもらえると嬉しいです。

## 前提条件

- 作業端末にAzure CLIがインストールされていること
- 作業を行うMicrosoft Entra IDアカウントに調査を行うサブスクリプションに対する閲覧者権限が割り当てられていること

## 手順

では早速手順を共有します。

### ①Azure CLIでログイン

```az login```コマンドでログイン

![001](/images/query-vm-restrictions/001.png)

対話的認証を済ませるとコンテキストとするサブスクリプションの選択する必要があるので、調査対象のサブスクリプションを示す数字を入力しEnter。

不慣れな方は```az account show```コマンドでコンテキストとなったサブスクリプションを確認するところまでセットでお願いします。

### ②仮想マシンの制限調査コマンドを実行

この手順で目的は達成されます。
```az vm list-skus```コマンドを実行するのですが、オプションの付け方に少しコツがあるので下記のコードスニペットをしっかり見てください。
今回は代表として**Standard_NC4as_T4_v3**サイズの制限を調べてみようと思います。

```bash
az vm list-skus \
  -l japaneast \
  -s Standard_NC4as_T4_v3 \
  --query "[].{name:name,restrictions:restrictions}" \ # ココに注目✌
  -o json
```

queryオプションの中でrestrictionsというプロパティを引っ張ってきてください。この中にほしい情報が詰まっています。
すると現在コンテキストとなっているサブスクリプションでは下記のようなレスポンスが返却されました。

```json
[
  {
    "name": "Standard_NC4as_T4_v3",
    "restrictions": [
      {
        "reasonCode": "NotAvailableForSubscription",
        "restrictionInfo": {
          "locations": [
            "japaneast"
          ]
        },
        "type": "Location",
        "values": [
          "japaneast"
        ]
      },
      {
        "reasonCode": "NotAvailableForSubscription",
        "restrictionInfo": {
          "locations": [
            "japaneast"
          ],
          "zones": [
            "1",
            "2",
            "3"
          ]
        },
        "type": "Zone",
        "values": [
          "japaneast"
        ]
      }
    ]
  }
]
```

解読すると「Standard_NC4as_T4_v3サイズの仮想マシンは、現在のサブスクリプションでは japaneast リージョン全体、および japaneast の全ゾーン（1, 2, 3）で利用できない」という意味になります。

今回はリージョン全体で使えないという極端な例だったので可用性ゾーン1~3のすべてで利用できないという当たり前の結果でしたが、「東日本リージョンで使えるけど可用性ゾーン1で使えない」という状況となることもあります。

可用性ゾーン規模の冗長構成を施す要件がある場合、本制限によりそれが満たせないことが構築フェーズで明らかになっては手遅れとなることがありますので、ぜひ上記の手順で事前調査を行ってください。

## おわりに

今回はライトなボリュームでおさまりました。
いつものようにMCPサーバーで遊んでいたらたまたま本記事で扱ったコマンドにたどり着いたので、備忘録として残してみました。

VM以外のサービスでも類似したコマンドが提供されていると思うので、皆さんもぜひ探してみてくださいね。

恒例でマサカリ大歓迎です。
ではでは。

## 参考記事

<https://learn.microsoft.com/ja-jp/cli/azure/vm?view=azure-cli-latest#az-vm-list-skus>
