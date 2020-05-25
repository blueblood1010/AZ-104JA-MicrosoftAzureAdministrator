﻿---
lab:
    title: '02a - サブスクリプションと RBAC の管理'
    module: 'モジュール 02 - ガバナンスとコンプライアンス'
---

# ラボ 02a - サブスクリプションと RBAC を管理する
# 受講者用ラボ マニュアル

## ラボ シナリオ

Contoso の Azure リソースの管理を強化するために、次の機能を実装する必要があります。

- Contoso のすべての Azure サブスクリプションを含む管理グループを作成する

- 管理グループ内のすべてのサブスクリプションに対するサポート要求を、指定された Azure Active Directory ユーザーに提出するアクセス許可を付与する。そのユーザーのアクセス許可は、次の場合に限定する必要があります。 

    - サポート リクエスト チケットの作成
    - リソース グループの表示 

## 目標

この課題では、次の内容を学習します。

+ タスク 1: 管理グループの導入
+ タスク 2: カスタム KPI ロールの作成 
+ タスク 3: RBAC ロールの割り当て


## 予想時間: 30分間

## 指示

### 演習 1

#### タスク 1: 管理グループの導入

このタスクでは、管理グループを作成および構成します。 

1. [Azure portal](https://portal.azure.com) にログインします。

1. **管理グループ**を検索して選択し、「**管理グループ**」 ブレードで 「**+ 管理グループの追加**」 をクリック します。

1. 次の設定で管理グループを作成してください:

    | 設定 | 値 |
    | --- | --- |
    | 管理グループ ID | **az104-02-mg1**|
    | 管理グループの表示名 | **az104-02-mg1**|

1. 管理グループの一覧で、新しく作成された管理グループのエントリをクリックし、その**detail**をクリックします。

1. 「**az104-02-mg1**」 ブレードで、「**+ サブスクリプションの追加**」 をクリックし、このラボで使用しているサブスクリプションを管理グループに追加します。

    >**注**: Azure サブスクリプションの ID をクリップボードにコピーします。これは、次のタスクで必要になります。

#### タスク 2: カスタム KPI ロールの作成

このタスクでは、カスタム RBAC ロールの定義を作成します。

1. ラボのコンピューターから <**\\Allfiles\\Labs\\02\\az104-02a-customRoleDefinition.json** ファイルをメモ帳で開き、その内容を確認します。

   ```json
   {
      "Name": "サポート要求共同作成者 (カスタム)",
      "IsCustom": true,
      "Description": "サポート要求を作成できます",
      "Actions": [
          "Microsoft.Resources/subscriptions/resourceGroups/read",
          "Microsoft.Support/*"
      ],
      "NotActions": [
      ],
      "AssignableScopes": [
          "/providers/Microsoft.Management/managementGroups/az104-02-mg1",
          "/subscriptions/SUBSCRIPTION_ID"
      ]
   }
   ```

1. JSON ファイルの `SUBSCRIPTION_ID` プレースホルダーを、クリップボードにコピーしたサブスクリプション ID に置き換えて、変更を保存します。

1. Azure portal で、「**Cloud Shell**」 ウィンドウを開くには、検索テキスト ボックスの右側にあるツール バー アイコンをクリックします。

1. **Bash** または **PowerShell** のいずれかを選択するように求められた場合、**PowerShell** を選択します。。 

    >**注**: **Cloud Shell** を初めて起動し、「**ストレージがマウントされていません**」というメッセージが表示されたら、このラボで使用しているサブスクリプションを選択し、「**ストレージの作成**」 をクリックします。 

1. Cloud Shell ウィンドウのツール バーで、 「**ファイルのアップロード/ダウンロード**」 アイコンをクリックし、ドロップダウン メニューの 「**アップロード**」 をクリックして、ファイル **\\Allfiles\\Labs\\02\az104-02a-customRoleDefinition.json** を Cloud Shell ホーム ディレクトリにアップロードします。

1. Cloud Shell ペインから、次の操作を実行して、カスタム ロール定義を作成します。

   ```pwsh
   New-AzRoleDefinition -InputFile $HOME/az104-02a-customRoleDefinition.json
   ```

1. Cloud Shell パネルを閉じます。

#### タスク 3: RBAC ロールの割り当て

このタスクでは、Azure Active Directory ユーザーを作成し、前のタスクで作成した RBAC ロールをそのユーザーに割り当て、ユーザーが RBAC ロール定義で指定されたタスクを実行できることを確認します。

1. Azure portal で、「**Azure Active Directory**」 ブレードで 「Azure Active Directory」 を検索して選択し、「**ユーザー**」をクリックして、「**+ 新しいユーザー**」 をクリックします。

1. 次の設定で新しいユーザーを作成します (その他は既定のままにします)。

    | 設定 | 値 |
    | --- | --- |
    | ユーザー名 | **az104-02-aaduser1**|
    | 名前 | **az104-02-aaduser1**|
    | パスワードを作成します | enabled |
    | 初期パスワード | **Pa55w.rd124** |

    >**注**: **クリップボードにフル** ユーザー名 **をコピーします**。 このラボの後半で使用します。

1. Azure portal で、 **az104-02-mg1** 管理グループに戻り、 **detail**をクリックします。

1. 「**アクセス制御 (IAM)**」をクリックし、「**+ 追加**」、「**ロールの割り当て**」 の順にクリックし、新しく作成したユーザー アカウントに「**Support Request Contributor (Custom)**」ロールを割り当てます。

1. **InPrivate** ブラウザー ウィンドウを開き、新しく作成したユーザー アカウントを使用して [Azure portal](https://portal.azure.com)にサインインします。パスワードを更新するように求められたら、ユーザーのパスワードを変更します。

    >**注**: ユーザー名を入力するのではなく、クリップボードの内容を貼り付けることができます。

1. **InPrivate** ブラウザー ウィンドウの Azure portal で、「**リソース グループ**」を検索して選択し、az104-02-aaduser1 ユーザーがすべてのリソース グループを表示できることを確認します

1. **InPrivate** ブラウザー ウィンドウの Azure portal で、「**すべてのリソース**」 を検索して選択し、az104-02-aaduser1 ユーザーがリソースを表示できないことを確認します。

1. **InPrivate** ブラウザー ウィンドウの Azure portal で、**ヘルプ + サポート** を検索して選択し、**+ 新しいサポート要求** をクリックします。 

1. **InPrivate**ブラウザー ウィンドウの 「 **ヘルプ + サポート -> 新しいサポート要求**」 ブレードの 「**基本**」 タブで、「 **サービスとサブスクリプションの制限 (クォータ)**」 の問題の種類を選択し、このラボで使用しているサブスクリプションが 「**サブスクリプション**」 ドロップダウン リストに表示されていることを確認します。

    >**注**: この課題で使用しているサブスクリプションが 「**サブスクリプション**」 ボックスの一覧に表示されている場合、サブスクリプション固有のサポート リクエストが可能なアカウントが存在します。

    >**注**: 「**サービスとサブスクリプションの制限 (クォータ)**」 オプションが表示されない場合は、Azure portal からサインアウトしてログインし直します。

1. サポート リクエストの作成を続行しないでください。代わりに、Azure portal から az104-02-aaduser1 ユーザーとしてサインアウトし、InPrivate ブラウザー ウィンドウを閉じます。

#### リソースのクリーンアップ

   >**注**: 新しく作成した Azure リソースのうち、使わないリソースは必ず削除してください。 

   >**注**: このラボで作成したリソースで余分なコストは発生しませんが、未使用のリソースを削除すると予期しない料金は発生しなくなります。

1. Azure ポータルで 「**Azure Active Directory**」を検索して選択し、Azure Active Directory ブレードで、**ユーザー**をクリックします。

1. 「 **ユーザー | すべてのユーザー** 」 ブレードで "**az104-02-aaduser1**"をクリックします。

1. 「**az104-02-aaduser1 | プロファイル**」 ブレードで、「**オブジェクト ID**」 属性 の値をコピー します。

1. Azure portal で **Cloud Shell** 内の **PowerShell** セッションを開始します。

1. 「Cloud Shell」 ペインで、次の手順を実行してカスタム ロール定義の割り当てを削除します (`[object_ID]` プレースホルダーを、前にコピーした **az104-02-aaduser1** の Azure Active Directory ユーザー アカウントの**object ID**属性の値で置き換えます)。

   ```pwsh
   $scope = (Get-AzRoleAssignment -RoleDefinitionName 'Support Request Contributor (Custom)').Scope

   Remove-AzRoleAssignment -ObjectId '[object_ID]' -RoleDefinitionName 'Support Request Contributor (Custom)' -Scope $scope
   ```

1. Cloud Shell ペインから次を実行して、カスタム ロールの定義を削除します。

   ```pwsh
   Remove-AzRoleDefinition -Name 'Support Request Contributor (Custom)' -Force
   ```

1. Azure portal で、**Azure Active Directory** の 「**ユーザー - すべてのユーザー**」 ブレードに戻り、**az104-02-aaduser1** ユーザー アカウントを削除します。

1. Azure portal で、**az104-02-mg1** 管理グループに移動し、**詳細**を表示します。

1. Azure サブスクリプションを表すエントリの右側にある**省略記号**アイコンを右クリックし、「**移動**」 をクリックします。

1. 「**移動**」 ブレードで、サブスクリプションが最初に含まれている管理グループを選択し、「**保存**」 をクリックします。 

1. 「**管理グループ**」 ブレードに戻り、**az104-02-mg1** 管理グループの右側にある**省略記号**アイコンを右クリックし、「**削除**」 をクリックします。

#### レビュー

この課題では、次の内容を学習します。

- 管理グループの導入
- カスタム RBAC ロールの作成 
- RBAC ロールの割り当て