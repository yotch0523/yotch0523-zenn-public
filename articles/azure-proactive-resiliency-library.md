---
title: "Azure Proactive Resiliency Libraryとは何か"
emoji: "🛠️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Azure", "Resiliency"]
published: true
publication_name: "azpower"
---

# はじめに

こんにちは、AZPower 株式会社のクラウドインテグレーション部に所属している wai です。

みなさんは **Azure Proactive Resiliency Library(APRL)** をご存知でしょうか？

後に詳しく解説しますが、**Azure 上のワークロードの回復性（resiliency）を向上させるためのテクニックやツールを集約した、Microsoft が提供するライブラリ**です。しかし、Zenn をはじめとする各種プラットフォームで日本語の記事が見当たらないことから、まだ知名度はそれほど高くないようです。

少し話は逸れますが、Microsoft Learn では、 **Azure Well-Architected Framework（W-AF）** の五大柱の一つである「信頼性」を実現する上で、回復性が重要であることが言及されています。
https://learn.microsoft.com/ja-jp/azure/reliability/overview#what-is-reliability

ゆえに、実は Azure W-AF 内にも回復性に関するコンテンツが含まれているんですね。

では、なぜそれとは別に APRL が提供されているのでしょうか？
本記事では、APRL の位置付け、意義、そして具体的な活用方法について解説します。

みなさんのワークロードの回復性向上に少しでも役立てば幸いです。

# 本記事のポイント
- APRLの活用の仕方を解説します
  - 個別具体のケースに応じた推奨事項をチェックできます
  - 既存環境上のリソースに対して、推奨事項の準拠状況をツールによりチェック・分析できます

# 本記事の想定読者
- Azure W-AFの五大柱の一つ、「信頼性」とは何かを概ね理解している方
  - 原則を「完全に理解した」という状態になっている必要はありません、そのような人類は恐らくまだほとんどいません
  - まだの方は[こちら](https://learn.microsoft.com/ja-jp/azure/well-architected/reliability/principles)からおさらいしておくことをお勧めします
- 自社のワークロードの回復性を向上させたいが、具体的なアイデアが浮かばず手詰まりになっている方

# APRL とは

**Azure 上で稼働するワークロードの回復性を向上させるためのベストプラクティスや推奨事項、スクリプトを集めたライブラリです。**
https://azure.github.io/Azure-Proactive-Resiliency-Library-v2/welcome/
![image001.png](/images/azure-proactive-resiliency-library/image001.png)

本ライブラリのコンテンツを管理している GitHub リポジトリも公開されています。
https://github.com/Azure/Azure-Proactive-Resiliency-Library-v2

提供されているコンテンツのうち、実際に回復性の向上に活用できるものは以下の四つのカテゴリに分けられます。それぞれの詳細については後の各章で説明します。

1. **_Azure Resources_**
   **個々の Azure リソースごとの回復性を向上させるベストプラクティス**がまとめてられているコンテンツです。

2. **_Specilized Workloads_**
   **代表的なワークロードに対する回復性を向上させるベストプラクティス**がまとめられているコンテンツです。

3. **_Well-Architected Framework_**
   **Azure W-AF の信頼性に関するベストプラクティス**の概要と、その詳細がまとめられているコンテンツです。

4. **_Tools / Tools(PREVIEW)_**
   APRL にて提供されているツールの仕様、および利用方法の説明がまとめられているコンテンツです。
   ツールには Tools / Tools(PREVIEW)の二種類がありますが、**実行形式が異なる**こと、そして**後者に対しては生成されるファイルのフォーマットがブラッシュアップされている**、といった違いがあります。提供されている機能自体はどちらも同じです。
   どちらも **Azure 環境にデプロイされているリソースが、どの程度 Well-Architected Framework の信頼性のベストプラクティスに準拠しているかをチェックする**ことができます。

# W-AF との違い

APRL は Azure W-AF の信頼性のベストプラクティスに準拠したライブラリであり、そのため内容的に重複している部分も少なくありません。

一言で違いを表すと、**Azure W-AF はクラウドワークロードの設計原則を提供する**のに対し、**APRL はより具体的な構成のベストプラクティスや回復性向上のためのツールを提供する**ものだと考えると分かりやすいと思います。

Azure W-AF で原則を概ね理解した後に、具体的に利用を検討しているサービスに対してどんなテクノロジーを活用できるのかを調べたり、運用しているサービスの回復性を検証するためにツールを活用する、といったイメージを持っていただければよいかと思います。

# APRL の活用方法

それでは、ここからは各種コンテンツの活用方法を紹介していきます。

## Azure Resources

各 Azure リソースごとの回復性向上に繋がるベストプラクティスが紹介されています。

![Azure Resources](/images/azure-proactive-resiliency-library/image002.png)

例えば、この一覧の中に App Service に関するものがあるので、そのリンクをクリックします。

![App Service Page](/images/azure-proactive-resiliency-library/image003.png)

すると、App Service に関する設定のベストプラクティスが一覧表示されています。

![App Service Best Practices](/images/azure-proactive-resiliency-library/image004.png)

内容としては下記の通りです。

| 列名                     | 詳細                                                                                                   |
| ------------------------ | ------------------------------------------------------------------------------------------------------ |
| **Recommendation**       | **推奨事項の概要**<br>アンカーリンクとなっていて、同ページ内にある詳細が記載されたセクションへ移動する |
| **Impact**               | **回復性向上に寄与する度合い**<br>**Low** < **Medium** < **High** の３レベルで表現されている                       |
| **Category**             | **推奨事項のカテゴリ**                                                                                 |
| **Automation Available** | **推奨事項に準拠しているかどうかを自動的に調査できるかどうか**                                         |
| **In Azure Advisor**     | **Azure Advisor の推奨事項として列挙されるかどうか**                                                   |

### 詳細セクション

最後に各推奨事項の詳細セクションを覗いてみます。

![detail section](/images/azure-proactive-resiliency-library/image005.png)

各項目に関してはキャプチャで示してある通りなのですが、特に注目していただきたいのが下記の二項目です。

- **Learn More**
  該当の推奨事項に関して解説されている Microsoft Learn へのリンクが記載されています。遷移先で準拠するための構成手順が紹介されているので、スムーズに改善アクションに移ることができます。
- **ARG Quey**
  Azure Resource Graph で該当の推奨項目の準拠状況を調査するためのクエリが示されています。残念ながらこちらに関してはクエリで調査できるものとそうでないものとがあるので、ない場合は潔く Azure ポータルを覗きにいきましょう。

以上が Azure Resources の概要になります。

## Specialized Workloads

こちらは特定のワークロードにおいて回復性を向上させるベストプラクティスがまとめられています。

現時点（2025 年 2 月）では **AI** や **HPC**、**AVD**、**AVS**、**SAP** に関するものが提供されています。
試しに AVD のページを覗いてみます。

![Specialized Workloads](/images/azure-proactive-resiliency-library/image006.png)

各欄の概要は下表の通り

| 列名                   | 詳細                                                                                                                          |
| ---------------------- | ----------------------------------------------------------------------------------------------------------------------------- |
| **Recommendation**     | **推奨事項の概要**<br>後述の構成対象リソースの[Azure Resources](#azure-resources)推奨事項ページへと遷移するリンクとなっている |
| **Provider Namespace** | 推奨事項の構成対象であるリソースの**リソースプロバイダー名前空間**                                                            |
| **Resource Type**      | 推奨事項の構成対象である**リソースの種類**                                                                                    |

Recommendation 欄のリンクに関しては上表の通りで、例えば「Create a validation host pool」推奨事項のリンクをクリックすると、[hostPools リソースの Azure Resources コンテンツページ](https://azure.github.io/Azure-Proactive-Resiliency-Library-v2/azure-resources/DesktopVirtualization/hostPools/#Create-a-validation-host-pool)へと遷移します。

![hostPools Azure Resourcesコンテンツページ](/images/azure-proactive-resiliency-library/image007.png)

[Azure Resources の詳細セクション](#%E8%A9%B3%E7%B4%B0%E3%82%BB%E3%82%AF%E3%82%B7%E3%83%A7%E3%83%B3)で説明したように、Learn More リンクから Microsoft Learn ドキュメントへ遷移したり、ARG クエリを確認することができます。

### General Workload Guidance

実は Specialized Workloads ページを下に少しスクロールすると**General Workload Guidance**というセクションが設けられていることが分かります。

![General Workload Guidance](/images/azure-proactive-resiliency-library/image008.png)

これは、**該当ワークロードに関連する Azure リソースの推奨事項**がリストアップされています。
例えば、上キャプチャの「Configure AVD Insights workbook」リンクをクリックすると、Azure Monitor ソリューションの一つである**Azure Virtual Desktop Insights**のワークブックの詳細を記載したセクションへと遷移します。Azure Resources コンテンツと同様ですね。
本セクションもお見逃しのないように。

以上が Specialized Workloads の概要になります。

## Well-Archicted Framework

Azure W-AF の信頼性に関する推奨事項がまとめられています。APRL を利用している最中にふと原則に立ち返りたくなったらこちらから復習してみると良いでしょう。

## Tools / Tools(PREVIEW)

現在公開されているのは [Collector Script](#collector-script)、[Analyzer Script](#analyzer-script)、[Reports Generator Script](#reports-generator-script) の 3 つです。いずれも **Microsoft Well-Architected Reliablity Assessment(WARA)** に基づく信頼性評価の一環として提供されるものです。

プレビュー版はコマンドレットによる実行形式ですが、提供されているコマンドの種類はスクリプト版のそれと同様です。
今回は、代表して Tools、つまり**スクリプト版**の解説を行います。

:::message
後述のスクリプトは Azure Resource Manager(ARM)データのみを読み取るため、いわゆるキーやシークレットといった**リソースに格納されているデータを読み取ることはありません**。
そのため、提供されているツールを利用することにより情報漏洩が起きることはありませんのでご安心ください。
:::

![Important](/images/azure-proactive-resiliency-library/image009.png)

:::message alert
ただし、実行結果ファイルに**サブスクリプション ID**が出力されるので取り扱いには気をつけてください
:::

最後に、ツールの実行対象となるリソースは以下の通りです。

![Important](/images/azure-proactive-resiliency-library/image010.png)

| リソース                          | SKU/tier | 備考                                                                                                                       |
| --------------------------------- | -------- | -------------------------------------------------------------------------------------------------------------------------- |
| **Storage Account**               | 汎用 v2  |                                                                                                                            |
| **ユーザー割り当てマネージド ID** | -        | 下記のロールを割り当てる<br>・Storage Account に対する共同作成者ロール<br>・ストレージ BLOB データ共同作成者ロール |
| **App Service**                   | F1       | ユーザー割り当てマネージド ID を割り当てる                                                                                 |

Bicep テンプレートも載せておきます↓
:::details Bicep テンプレート

```bicep
param uamiName string
param location string = resourceGroup().location
param tags object = resourceGroup().tags
param storageAccountName string
param appServicePlanName string
param appServiceName string

var roleDefinitionIds = [
  'b24988ac-6180-42a0-ab88-20f7382dd24c'
  'ba92f5b4-2d11-453d-a403-e96b0029c9fe'
]

module uami 'br/public:avm/res/managed-identity/user-assigned-identity:0.4.0' = {
  name: 'uami-deployment'
  params: {
    name: uamiName
    location: location
    tags: tags
  }
}

module storageAccount 'br/public:avm/res/storage/storage-account:0.18.0' = {
  name: 'storageAccount-deployment'
  params: {
    name: storageAccountName
    location: location
    skuName: 'Standard_LRS'
    kind: 'StorageV2'
    accessTier: 'Hot'
    tags: tags
    roleAssignments: [for (roleDefinitionId, index) in roleDefinitionIds: {
      roleDefinitionIdOrName: roleDefinitionId
      principalId: uami.outputs.principalId
      principalType: 'ServicePrincipal'
    }]
  }
}

module appServicePlan 'br/public:avm/res/web/serverfarm:0.4.1' = {
  name: 'appServicePlan-deployment'
  params: {
    kind: 'app'
    location: location
    name: appServicePlanName
    skuName: 'F1'
    tags: tags
  }
}

module appService 'br/public:avm/res/web/site:0.14.0' = {
  name: 'appService-deployment'
  params: {
    kind: 'app'
    location: location
    name: appServiceName
    serverFarmResourceId: appServicePlan.outputs.resourceId
    tags: tags
    managedIdentities: {
      systemAssigned: false
      userAssignedResourceIds: [
        uami.outputs.resourceId
      ]
    }
    siteConfig: {
      alwaysOn: false
    }
  }
}
```

:::

[Azure Verified Modules(AVM)](https://azure.github.io/Azure-Verified-Modules/)を利用して各種リソースを宣言しており、最低限のパラメーターしか指定していません。

### 前準備

以降のスクリプトの実行に利用する情報を、あらかじめ PowerShell 変数にセットするのと、Azure PowerShell でご自身のユーザーアカウントのログインを済ませておいてください。

```powershell
$TenantId = '[ご自身の環境のテナントID]'
$ResourceGroups = @('/subscriptions/[ご自身の環境のサブスクリプションID]/resourceGroups/[実行対象リソースグループ名]')
Connect-AzAccount
```

### Collector Script

Collector Script は、**Azure 環境にデプロイされているリソースがベストプラクティスに則って構成されているかを調査する**ためのスクリプトです。具体的に収集されるデータは下記の通り

- **リソースの構成内容**
- **クローズされたサポートリクエストチケット**
- **アクティブな信頼性に関する Azure Advisor の推奨事項**
- **過去の Azure Service Health の廃止/停止通知**
- **Azure Service Health のアラート構成内容**

#### 前提条件

また、本ツールを利用するにあたっての前提条件は下記の通り

**【環境】**

- PowerShell 7
- Git
- Azure PowerShell モジュール

**【ユーザーアカウント】**

- 評価対象リソースに対する閲覧者ロール

#### 実行

準備が整ったところで、APRL の GitHub リポジトリから**1_wara_collector.ps1**スクリプトをダウンロードし、下記のコマンドで実行してください。

```powershell
.\1_wara_collector.ps1 -TenantId $TenantId -ResourceGroups $ResourceGroups
```

今回は検証対象が一つのリソースグループのみなので上記のような引数で実行しましたが、環境を丸ごと調査したい場合は**SubscriptionIds 引数**を指定するといった調整を加えてください。
:::message alert
リソース数が多いとその分時間がかかるため注意
:::

他にも[こちら](https://azure.github.io/Azure-Proactive-Resiliency-Library-v2/tools/collector/#resource-filtering)で説明されているように、タグの構成内容でフィルタリングすることもできるようなので、色々試してみてください。やはりタグ付けって大切ですね。

成功すると下キャプチャのように実行結果が JSON ファイルとして出力された旨のメッセージが表示されます。

![succeeded capture](/images/azure-proactive-resiliency-library/image011.png)

JSON ファイルの中身を確認すると、リソース数が少ないのにも関わらず中々のボリュームです。
![result json file](/images/azure-proactive-resiliency-library/image012.png)

これでは使い勝手が悪いので、[Analyzer Script](#analyzer-script) を実行することにより検証結果を分析し、より分かりやすい形式に変換していきます。

#### 補足：Runbook

Runbook とは、**Collector Script の検証対象リソースをより柔軟にフィルタするために用いるファイル**です。
正規表現による絞り込みのサポートなど、痒いところにも手が届くような仕様になっています。
仕様についての詳細は[こちら](https://azure.github.io/Azure-Proactive-Resiliency-Library-v2/tools/collector/runbooks/)の公式ドキュメントをご覧ください。
Collector Script 実行時に、Runbook JSON ファイルまでの絶対パス、もしくは相対パスを**RunbookFile**引数として渡してあげることで有効になります。

### Analyzer Script

Analyzer Script は、[Collector Script](#collector-script)により出力された JSON ファイルをもとに、**推奨事項の準拠状況を Excel ファイルへ変換する**ためのスクリプトです。
これにより、検出対象リソースの情報がより把握しやすい形で提供され、分析がしやすくなります。

#### 前提条件

本ツールを利用するに前提条件は下記の通りです。

**【環境】**

- **Windows OS⚠️**
- PowerShell 7
- Git
- **Excel がインストール済み**

:::message alert
上記の通り、本スクリプトは Windows OS でしか利用できませんのでご注意ください。
:::

#### 実行

では、APRL の GitHub リポジトリから**2_wara_data_analyzer.ps1**スクリプトをダウンロードし、下記のコマンドで実行してください。

```powershell
.\2_wara_data_analyzer.ps1 -JSONFile [出力されたJSONファイルまでのパス]
```

実行したスクリプトと同階層に Excel ファイルが出力されます。
この Excel ファイルには合計 7 つのシートが設けられていて、各シートの内容を簡単に解説します。

#### 1. Recomendations シート

![recomendations](/images/azure-proactive-resiliency-library/image015.png)

すべての推奨事項がリストアップされているシートです。

#### 2. ImpactedResources シート

![impacted resources](/images/azure-proactive-resiliency-library/image016.png)

各推奨事項に関連付けられているリソースの一覧が表示されているシートです。
本シートで注意すべきこととしては、**列挙されているもの全てが必ずしも非準拠とは限らない点にあります**。実は、Collector Script により自動的に検証可能な項目とそうでない項目があります。

A 列に「**APRL - Queries**」と記載のあるものは、**Collector Script の実行により自動的に検証ができている**ことを表します。
反対にそうでないものは、**自動検証ができなかった**ことを表しますので、**ユーザー側での確認が必要**です。A 列に表示されるメッセージの種類と、その意味する内容の詳細を知りたい方は[こちら](https://azure.github.io/Azure-Proactive-Resiliency-Library-v2/tools/analyzer/#action-plan-analysis)をご覧ください。

少々面倒に思うかもしれませんが、対象リソースを ARG でクエリ可能な場合、A 列にその KQL クエリが出力されるので、それを活用し調査を行うことができます。
そうでない場合は手動で Azure ポータルから調べていくしかありません。

上記の作業は、後に解説する[Reports Generator Script](#reports-generator-script)の実行の前に、ここで**レポートする必要のない推奨事項を間引いておく**ことを目的としています。

#### 3. Other-OutOfScope シート

![other](/images/azure-proactive-resiliency-library/image017.png)

スコープ内にデプロイされているものの、検証の対象外であるリソースがリストアップされているシートです。

#### 4. ResourceTypes シート

![resource types](/images/azure-proactive-resiliency-library/image018.png)

今回の検証対象となっているリソースの種類の内訳が集計されているシートです。

#### 5. Retirements シート

![retirements](/images/azure-proactive-resiliency-library/image019.png)

検証対象となったリソース、およびその構成内容のうち、廃止予定となっているものをリストアップしたシートです。
対象オブジェクトが見つからない場合、このシートは作成されません。

#### 6. PivotTable シート

![pivot table](/images/azure-proactive-resiliency-library/image020.png)

検証対象とリソース、および推奨事項のカテゴリの内訳が集計されているシートです。

#### 7. Charts シート

![charts](/images/azure-proactive-resiliency-library/image021.png)

PivotTable シートの集計内容を元にしたグラフが格納されているシートです。

また、今回は出力されませんでしたが、下記の二つのシートも作成される仕様のようです。

#### 8. Outages シート

Collector Script 実行時に検証対象として指定されていたものの、停止されていて検証を行えなかったサブスクリプションがリストアップされているシートです。

#### 9. Support Tickets シート

検証対象となったサブスクリプションにて、直近の半年間に渡って発行されたサポートチケットがリストアップされているシートです。

Analyzer Script により分析ができたら、最後にこれから解説する [Reports Generator Script](#reports-generator-script) を実行します。

### Reports Generator Script

準拠状況調査の最後に実施するのが Reports Generator Script です。
こちらは、**Analyzer Script により出力された Excel ファイルを読み込み、Word および PowerPoint 形式に変換する**ためのスクリプトです。
経営層に対し、より理解しやすいフォーマットで分析情報を共有する必要がある場合に本スクリプトが役に立ちます。

...と言いたかったのですが、残念ながら本スクリプトにより生成される Word ファイルと PowerPoint ファイルは**英語**表記となっているので、生成されたままでの状態で利用するのは日本では難しいというのが正直なところです。
しかし、昨今、自動的に和訳することができる便利なソフトウェアが揃っているのは不幸中の幸いかと思います。
後に解説するフォーマットなどを考慮し、業務において利用するかどうかを検討してください。

（おそらく、テンプレートの Word/PowerPoint ファイルを自社製のものにカスタマイズすればお望みの言語・フォーマットで生成できそうに思えますが、筆者の体力と時間の関係で別記事の対応とさせてください）

#### 前提条件

本ツールを利用するに前提条件は下記の通りです。

**【環境】**

- PowerShell 7
- **Excel、Word、PowerPoint がインストールされている**

#### 実行

APRL の GitHub リポジトリから下記の 3 つのファイルをダウンロードしてください。

- **3_wara_reports_generator.ps1**
- **Mandatory - Executive Summary presentation - Template.pptx**
- **Optional - Assessment Report - Template.docx**

上記 3 つのファイルを任意のディレクトリの同階層に配置した後に、下記のコマンドでスクリプトを実行してください。

```powershell
.\3_wara_reports_generator.ps1 -CustomerName '[任意の顧客名]' -WorkloadName '[任意のワークロード名]' -ExcelFile [Analyzer Scriptにより出力されたExcelファイルまでのパス]
```

:::message alert
- 一度本スクリプトを実行後に再度実行したい場合は、**直前の処理で生成された Word ファイル、PowerPoint ファイルを開いていないこと**を確認してください。開いている場合はスクリプトが失敗します。

- 実行する端末のスペックが低いといった理由で、スクリプトの実行により端末のリソースが逼迫することが懸念される場合は、**Heavy オプション**に **$true** を指定して実行することを推奨します。

- Microsoft Purview の秘密度ラベルにより暗号化されている場合、**本スクリプト実行前に一時的に解除**してください。
:::

以下、生成された Word ファイルと PowerPoint ファイルの一部をお見せします。

#### Word ファイル

![word home](/images/azure-proactive-resiliency-library/image026.png)
![word 1](/images/azure-proactive-resiliency-library/image027.png)
![word 2](/images/azure-proactive-resiliency-library/image028.png)

#### PowerPoint ファイル

![ppt home](/images/azure-proactive-resiliency-library/image022.png)
![ppt 1](/images/azure-proactive-resiliency-library/image023.png)
![ppt 2](/images/azure-proactive-resiliency-library/image024.png)
![ppt 3](/images/azure-proactive-resiliency-library/image025.png)

おしゃれ、そして英語。

長くなりましたが、以上が APRL で提供されているツールの解説でした。
今回は PowerShell スクリプトがどう書かれているのかを読んでみたかったこともありスクリプト版を解説しましたが、個人的には PREVIEW ではなくなったらコマンドレット版を利用していきたいと思います。

# 終わりに

以上が本記事の内容になります。
Azure Verified Modules のドキュメント内で見かけたのが APRL を知ったきっかけでした。個別具体のリソース・ケースごとの推奨事項が示されていて、設計時に活用できそうだというのが個人的な印象です。

また、推奨事項に関して一言補足です。
**回復性とコストは当然トレードオフの関係にある**ので、リッチにすればするほど利用料は高くなっていきます。**ワークロードのビジネス要件や重要性、特性に応じて取捨選択や優先順位付けをすることが重要です。**

そして、クラウドの進化に伴って Azure W-AF や APRL の内容も当然変わってくるので、定期的に評価を行い、継続的な改善を試みることも忘れないようにしてください。

恒例でマサカリ大歓迎です。
ではでは。

# 参考記事

https://github.com/Azure/Azure-Proactive-Resiliency-Library-v2
https://learn.microsoft.com/ja-jp/azure/architecture/guide/design-principles/
https://learn.microsoft.com/ja-jp/azure/well-architected/what-is-well-architected-framework
