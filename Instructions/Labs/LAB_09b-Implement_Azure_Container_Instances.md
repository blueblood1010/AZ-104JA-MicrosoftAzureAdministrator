﻿---
lab:
    title: '09b - Azure Container Instancesの実装'
    module: 'モジュール 09 - サーバーレス コンピューティング'
---

# ラボ 09b - Azure Container Instances を実装する
# 受講者用ラボ マニュアル

## ラボ シナリオ

Contoso は、仮想化されたワークロード用の新しいプラットフォームの使用を検討しています。この目的を達成するために使用できるコンテナー イメージの数を定義しました。コンテナー管理を最小限に抑えるため、Docker イメージのデプロイに Azure Container Instances を使用する方法を評価する必要があります。

## 目標

このラボでは、次の内容を学習します。

+ タスク 1: Docker イメージを Azure コンテナー インスタンス にデプロイする
+ タスク 2: Azure コンテナー インスタンスの機能を確認する

## 予想時間: 20分間

## 指示

### 演習 1

#### タスク 1: Docker イメージを Azure コンテナー インスタンス にデプロイする

このタスクでは、Web アプリケーションの新しいコンテナー インスタンスを作成します。 

1. [Azure portal](https://portal.azure.com) にログインします。

1. Azure portal で 「**コンテナー インスタンス**」 を検索し、「**コンテナー インスタンス**」 ブレードで 「**+ 追加**」 をクリックします。 

1. 「**コンテナー インスタンスの作成**」 ブレードの 「**基本**」 タブで 、次の設定を指定します (その他は既定値のままにします)。

    | 設定 | 値 |
    | ---- | ---- |
    | サブスクリプション | このラボで使用している Azure サブスクリプションの名前 |
    | リソース グループ | 新しいリソース グループ **az104-09b-rg1** の名前 |
    | コンテナー名 | **az104-9b-c1** |
    | リージョン | Azure Container Instances をプロビジョニングできるリージョンの名前 |
    | Image ソース | **Quickstart イメージ** |
    | イメージ | **microsoft/aci-helloworld (Linux)** |

1. 「**次へ:**」 をクリックする「**ネットワーク**」 をクリックし、「**コンテナー インスタンスの作成**」 ブレードの 「**ネットワーク**」 タブで 、次の設定を指定します (他は既定値のままにします)。

    | 設定 | 値 |
    | --- | --- |
    | DNS 名ラベル | グローバルに一意の有効な DNS ホスト名 |
	
    >**注**: コンテナーは、dns-name-label.region.azurecontainer.io でパブリックにアクセスできるようになります。**DNS 名ラベルは使用できません** というエラー メッセージを受け取った場合は、別の DNS 名ラベルを指定します。

1. 「**Next: Networking >**」 をクリックして、「**コンテナー インスタンスの作成**」 ブレードの 「**詳細**」 タブで、何も変更せずに設定をレビューし、「**レビュー + 作成**」 をクリックしてから、「**作成**」 をクリックします。 

    >**注**: デプロイが完了するまで待ちます。これには約 3 分ほどかかります。

    >**注**: 待機している間、この[サンプル アプリケーションに隠れたコード](https://github.com/Azure-Samples/aci-helloworld)を参照いただくこともできます (\app から参照可能)。 

#### タスク 2: Azure コンテナー インスタンスの機能を確認する

このタスクでは、コンテナー インスタンスのデプロイをレビューします。

1. デプロイ ブレードで、「**リソースに移動**」をクリックします。

1. コンテナー インスタンスの 「**概要**」 ブレードで、「**状態」** が "**実行中**" と表示されていることを確認 します。 

1. コンテナー インスタンスの **FQDN** の値をコピーし、新しいブラウザー タブを開き、対応する URL に移動します。

1. 「**Azure コンテナー インスタンスへようこそ**」 ページが表示されていることを確認します。

1. 新しいブラウザー タブを閉じ、Azure portal に戻り、コンテナー インスタンス ブレードの 「**設定**」 セクションで 「**コンテナー**」 をクリックし、「**ログ**」 をクリックします。 

1. ブラウザーでアプリケーションを表示すると生成される HTTP GET 要求のログを確認します。

#### リソースのクリーンアップ

   >**注**: 新しく作成された Azure リソースのうち、使用しないリソースは必ず削除してください。使用していないリソースを削除することで、予期しないコストが発生しなくなります。

1. Azure portalの 「**Cloud Shell**」 ペインで、**PowerShell** セッションを開始します。

1. 次のコマンドを実行して、このモジュールのラボ全体で作成されたすべてのリソース グループを一覧表示します。

   ```pwsh
   Get-AzResourceGroup -Name 'az104-09b*'
   ```

1. 次のコマンドを実行して、このモジュールのラボ全体で作成したすべてのリソース グループを削除します。

   ```pwsh
   Get-AzResourceGroup -Name 'az104-09b*' | Remove-AzResourceGroup -Force -AsJob
   ```

    >**メモ**: コマンドは非同期に実行されるため (-AsJob パラメーターによって決まります)、同じ PowerShell セッション内ですぐに別の PowerShell コマンドを実行できますが、リソース グループが実際に削除されるまでに数分かかります。

#### レビュー

このラボでは、次の内容を学習しました。

- Azure コンテナー インスタンス を使用した Docker イメージのデプロイ
- Azure コンテナー インスタンスの機能のレビュー