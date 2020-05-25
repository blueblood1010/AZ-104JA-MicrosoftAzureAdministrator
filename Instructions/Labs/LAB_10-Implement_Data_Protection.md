﻿---
lab:
    title: '10 - データ保護を実装する'
    module: 'モジュール 10 - データ保護'
---

# ラボ 10 - 仮想マシンをバックアップする
# 受講者用ラボ マニュアル

## ラボ シナリオ

Azure 仮想マシンとオンプレミス コンピューターでホストされているファイルのバックアップと復元に Azure Recovery Services の使用を評価することになりました。また、Recovery Services のコンテナーに格納されているデータを、偶発的なデータ損失や悪意のあるデータ損失から保護する方法を特定する必要もあります。

## 目標

このラボでは、次の内容を学習します。

+ タスク 1: 課題環境をプロビジョニングする
+ タスク 2: Recovery Services コンテナーを作成する
+ タスク 3: Azure 仮想マシン レベルのバックアップを実装する
+ タスク 4: ファイルとフォルダのバックアップを実装する
+ タスク 5: Azure Recovery Services エージェントを使用してファイルの回復を実行する
+ タスク 6: Azure 仮想マシンのスナップショットを使用してファイルの回復を実行する (省略可能)
+ タスク 7: Azure Recovery Services の論理的な削除機能を確認する (任意)

## 予想時間: 50分間

## 指示

### 演習 1

#### タスク 1: ラボ環境をプロビジョニングする

このタスクでは、異なるバックアップ シナリオのテストに使用する 2 つの仮想マシンをデプロイします。

1. [Azure portal](https://portal.azure.com) にログインします。

1. Azure portal で右上にあるアイコンをクリックして **Azure Cloud Shell** を開きます。

1. **Bash** または **PowerShell** のいずれかを選択するように求められた場合、**PowerShell** を選択します。 

    >**注**: **Cloud Shell** を初めて起動し、「**ストレージがマウントされていません**」というメッセージが表示されたら、このラボで使用しているサブスクリプションを選択し、「**ストレージの作成**」 をクリックします。 

1. Cloud Shell ウィンドウのツール バーで、**ファイルのアップロード/ダウンロード** アイコンをクリックし、ドロップダウン メニューで **アップロード**をクリックし、ファイル **\\Allfiles\\Labs\\10\\az104-10-vms-template.json** と **\\Allfiles\\Labs\\10\\az104-10-vms-parameters.json** を Cloud Shell ホーム ディレクトリにアップロードします。

1. 「Cloud Shell」 ペインから次のコマンドを実行して、Virtual Machines をホストするリソース グループを作成します (`[Azure_region]` プレースホルダーを Azure Virtual Machines をデプロイする Azure リージョンの名前に置き換えます)。

   ```pwsh
   $location = '[Azure_region]'

   $rgName = 'az104-10-rg0'

   New-AzResourceGroup -Name $rgName -Location $location
   ```
1. 「Cloud Shell」 ペインから、次のコマンドを実行して最初の仮想ネットワークを作成し、アップロードしたテンプレートとパラメーター ファイルを使用して仮想マシンをデプロイします。

   ```pwsh
   New-AzResourceGroupDeployment `
      -ResourceGroupName $rgName `
      -TemplateFile $HOME/az104-10-vms-template.json `
      -TemplateParameterFile $HOME/az104-10-vms-parameters.json `
      -AsJob
   ```

1. Cloud Shell を最小化します。ただし、閉じないでください。

    >**注**: デプロイが完了するのを待たずに、次のタスクに進みます。デプロイに約 5 分かかる場合があります。

#### タスク 2: Recovery Services コンテナーを作成する

このタスクでは、Recovery Services コンテナーを作成します。

1. Azure portal で、「**Recovery Services コンテナー**」 を検索して選択し、「**Recovery Services コンテナー**」 ブレードで 「**+ 追加**」 をクリックします。

1. 「**Recovery Services コンテナーの作成**」 ブレードで、次の設定を指定します。

    | 設定 | 値 |
    | --- | --- |
    | サブスクリプション | このラボで使用している Azure サブスクリプションの名前 |
    | リソース グループ | 新しいリソース グループ 「**az104-10-rg1**」 の名前 |
    | 名前 | **az104-10-rsv1** |
    | リージョン | 前のタスクで 2 つの仮想マシンをデプロイしたリージョンの名前 |

    >**注**: 前のタスクで仮想マシンをデプロイしたリージョンと同じリージョンを指定してください。

1. 「**確認および作成**」 をクリックしてから、「**作成**」 をクリックします。

    >**注**: デプロイが完了するまで待ちます。デプロイにかかる時間は 1 分未満です。

1. デプロイが完了したら、「**リソースに移動**」 をクリックします。 

1. 「**az104-10-rsv1** Recovery Services コンテナー」 ブレードの 「**設定**」 セクションで、「**プロパティ**」 をクリックします。

1. 「**az104-10-rsv1 - プロパティ**」 ブレードで、「**バックアップ構成**」 ラベルの下にある 「**更新**」 リンクをクリックします。

1. 「**バックアップ構成**」 ブレードで、**ストレージ レプリケーションの種類**を**ローカル冗長**または **geo 冗長**のいずれかに設定できることがわかります。**Geo 冗長** の設定を既定のままにして、ブレードを閉じます。

    >**注**: この設定は、既存のバックアップ アイテムがない場合にのみ構成できます。

1. 「**az104-10-rsv1 - プロパティ**」 ブレードに戻り 、「**セキュリティ設定**」 ラベルの下にある 「**更新**」 リンクをクリックします。 

1. 「**セキュリティの設定**」 ブレードで、「**論理的な削除 (Azure 仮想マシンの場合)**」 が 「**有効**」 になっていることを確認します。

1. 「**セキュリティ設定**」 ブレードを閉じ、「**az104-10-rsv1** Recovery Services コンテナー」 ブレードに戻り、「**概要**」 をクリックします。

#### タスク 3: Azure 仮想マシン レベルのバックアップを実装する

このタスクでは、Azure 仮想マシン レベルのバックアップを実装します。

   >**注**: このタスクを開始する前に、この課題の最初のタスクで開始したデプロイが正常に完了していることを確認してください。

1. 「**az104-10-rsv1** Recovery Services コンテナー」 ブレードで、「**+ バックアップ**」 をクリックします。

1. 「**バックアップの目標**」 ブレードで、次の設定を指定します。

    | 設定 | 値 |
    | --- | --- |
    | ワークロードの実行場所 | **Azure** |
    | 何をバックアップしますか?  | **仮想マシン** |

1. 「**バックアップの目標**」 ブレードで、「**バックアップ**」 をクリックします。

1. 「**バックアップ ポリシー**」 で 「**既定ポリシー**」 設定を確認し、「**バックアップ ポリシーの選択**」 ドロップダウン リストで 「**新規作成**」 を選択します。

1. 新しいバックアップ ポリシーを次の設定で定義します (その他は既定値のままにします) 。

    | 設定 | 値 |
    | ---- | ---- |
    | ポリシーの名前 | **az104-10-backup-policy** |
    | 頻度 | **毎日** |
    | 時刻 | **12:00 AM** |
    | タイムゾーン | ローカル タイム ゾーンの名前 |
    | インスタント リカバリ スナップショットを保持する期間 | **2** 日間 |

1. 「**OK**」 をクリックして、ポリシーを作成します。これによって **バックアップするアイテム** のステップに自動的に切り替え、「**仮想マシンの選択**」 ブレードを開きます。

1. 「**仮想マシンの選択**」 ブレードで 「**az-104-10-vm0**」 を選択して 「**OK**」 をクリックし、「**バックアップ**」 ブレードで 「**バックアップを有効にする**」 をクリックします。

    >**注**: バックアップが有効になるのを待ちます。これには約 2 分かかります。 

1. 「**az104-10-rsv1** Recovery Services コンテナー」 ブレードに戻り、「**保護されたアイテム**」 セクションで 「**バックアップ アイテム**」 をクリックし、「**Azure 仮想マシン**」 エントリをクリックします。

1. 「**az104-10-vm0**」 の 「**Backup アイテム (Azure 仮想マシン)**」 ブレードで、「**Backup 事前チェック**」 エントリと 「**最後の Backup の状態**」 エントリの値を確認し、「**az104-10-vm0**」 エントリをクリックします。

1. 「**az104-10-vm0** バックアップ アイテム」 ブレードで 「**今すぐバックアップ**」 をクリックし、「**バックアップ保持期間**」 ドロップダウン リストの既定値をそのまま受け入れて、「**OK**」 をクリックします。

    >**注**: バックアップが完了するのを待たず、次のタスクに進みます。

#### タスク 4: ファイルとフォルダのバックアップを実装する

このタスクでは、Azure Recovery Services を使用してファイルとフォルダーのバックアップを実装します。

1. Azure portal で 「**仮想マシン**」 を検索して選択し、「**仮想マシン**」 ブレードで 「**az104-10-vm1**」 をクリックします。

1. 「**az104-10-vm1**」 ブレードで 「**接続**」 をクリックし、ドロップダウン メニューで 「**RDP**」 をクリックし、「**RDP で接続**」 ブレードで 「**RDP ファイルのダウンロード**」 をクリックし、プロンプトに従ってリモート デスクトップ セッションを開始します。

    >**注**: この手順は、Windows コンピューターからリモート デスクトップ経由で接続することを指します。Mac では、Mac App Store からリモート デスクトップ クライアントを使用でき、Linux コンピューターでは、オープンソースの RDP クライアント ソフトウェアを使用できます。

    >**注**: ターゲットの仮想マシンに接続する際は、警告メッセージを無視できます。

1. プロンプトが表示されたら、ユーザー名 "**Student**" とパスワード "**Pa55w.rd1234**" を使用してログインします。

1. Azure 仮想マシン 「**az104-10-vm1** へのリモート デスクトップ」 セッションで、「**サーバー マネージャー**」 ウィンドウの 「**ローカル サーバー**」 をクリックし、「**IE セキュリティ強化の構成**」 をクリック して、これを管理者向けに **オフ** にします。

1. Azure 仮想マシン 「**az104-10-vm1** へのリモート デスクトップ」 セッションで、Internet Explorer を起動して [Azure Portal](https://portal.azure.com) を参照し、自分の認証情報を使用してログインします。 

1. Azure portal で 「**回復サービス コンテナー**」 を検索して選択し、「**回復サービス コンテナー**」 で 「**az104-10-rsv1**」 をクリックします。

1. 「**az104-10-rsv1** Recovery Services コンテナー」 ブレードで、「**+ バックアップ**」 をクリックします。

1. 「**バックアップの目標**」 ブレードで、次の設定を指定します。

    | 設定 | 値 |
    | --- | --- |
    | ワークロードの実行場所 | **オンプレミス** |
    | 何をバックアップしますか?  | **ファイルとフォルダー** |

    >**注**: このタスクで使用している仮想マシンは Azure で実行されていますが、この環境を利用して、バックアップ機能がWindows Server オペレーティング システムを実行しているオンプレミス コンピューターに対しても適用できることを確認できます。

1. 「**バックアップの目標**」 ブレードで、「**インフラストラクチャの準備**」 をクリックします。

1. 「**インフラストラクチャの準備**」 ブレードで、「**Windows サーバーまたは Windows クライアント用エージェントのダウンロード**」 リンクをクリックします。

1. プロンプトが表示されたら 「**実行**」 をクリックして、既定の設定で 「**MARSAgentInstaller.exe**」 のインストールを開始します。 

    >**注**: 「**Microsoft Azure Recovery Services エージェント設定ウィザード**」 の 「**Microsoft アップデートのオプトイン**」 ページで、「**Microsoft アップデートを使用しない**」 インストール オプションを選択します。

1. 「**Microsoft Azure Recovery Services エージェント設定ウィザード**」 の 「**インストール**」 ページで、「**登録に進む**」 をクリックします。これによって **サーバーの登録ウィザード** が起動します。

1. Azure portal を表示する Internet Explorer ウィンドウに切り替え、「**インフラストラクチャの準備**」 ブレードで 「**ダウンロード済みまたは最新のリカバリ サーバー エージェントを使用している**」 チェック ボックスをオンにして、「**ダウンロード**」 をクリックします。

1. コンテナーの認証情報ファイルを開くか保存するかを確認するメッセージが表示されたら、「**保存**」 をクリックします。これによって、コンテナーの認証情報ファイルがローカルのダウンロード フォルダーに保存されます。

1. 「**サーバーの登録ウィザード**」 ウィンドウに戻り、「**コンテナー ID**」 ページで 「**参照**」 をクリックします。

1. 「**コンテナー認証情報の選択**」 ダイアログ ボックスで 「**ダウンロード**」 フォルダーを参照し、ダウンロードしたコンテナー認証情報ファイルをクリックして、「**開く**」をクリックします。

1. 「**コンテナー ID**」 ページで 「**次へ**」 をクリックします。

1. 「**サーバー登録ウィザード**」 の 「**暗号化の設定**」 ページ で、「**パスフレーズの生成**」 をクリックします。

1. 「**サーバー登録ウィザード**」 の 「**暗号化の設定**」 ページ で、「**パスフレーズを保存する場所の入力**」ドロップダウン リストの横にある 「**参照**」 ボタンをクリックします。

1. **「フォルダの参照」** ダイアログ ボックスで、**「ドキュメント」** フォルダを選択し、**「OK」** をクリックします。     

1. **「完了」**をクリックし、**Microsoft Azure バックアップ** の警告を確認して、 **「はい」** をクリックし、登録が完了するまで待ちます。     

    >**注**: 運用環境では、パスフレーズ ファイルは、バックアップ対象のサーバー以外の安全な場所に格納する必要があります。

1. **「サーバー登録ウィザード」** の**サーバー登録**のページで 、パスフレーズ ファイルの場所に関する警告を確認し、**「Microsoft Azure Recovery Services エージェントの起動」** チェックボックスがオンになっていることを確認し、**「閉じる」** をクリックします。        これにより、自動的に **Microsoft Azure バックアップ**コンソールが開きます。

1. **Microsoft Azure バックアップ** コンソールの **「操作」** ウィンドウで、**「バックアップのスケジュール」** をクリック します。     

1. **スケジュール バックアップ ウィザード**の 「**はじめに**」 ページで、「**次へ**」 をクリック します。     

1. 「**バックアップする項目の選択**」 ページで、「**アイテムの追加**」 をクリック します。   

1. **「項目の選択」** ダイアログ ボックスで、**[C:\\Windows\System32\\ドライバー\\etc\\]** を展開し、「**ホスト**」 を選択して **「OK」** をクリックします。       

1. **「バックアップの項目の選択」** ページで、**「次へ」** をクリックします。

1. **「バックアップ スケジュールの指定」** ページで、**「日」** オプションが選択されていることを確認し 、**「次の時刻 (最大許容時間は 1 日 3 回)」** ボックスの下にある最初のドロップダウン リスト ボックスで 「**4:30 AM**」 を選択し、**「次へ」** をクリック します。         

1. **リテンション期間ポリシーの選択** ページで、既定の設定値を受け入れてから、**次へ:** をクリックします。

1. **最初のバックアップ タイプの選択** ページで、既定の設定値を受け入れてから、**次へ:** をクリックします。

1. **確認**ページで、**「終了」** をクリックします。バックアップ スケジュールが作成されたら、**「閉じる」** をクリック します。 
  
1. **Microsoft Azure バックアップ** コンソールの 「操作」 ウィンドウで、**「今すぐバックアップ」** をクリック します。   

    >**注**: スケジュール バックアップを作成すると、オンデマンドでバックアップを実行するオプションが使用可能になります。

1. 「今すぐバックアップ」 ウィザードの **「バックアップ項目の選択」** ページで、**「ファイルとフォルダ」** オプションが選択されていることを確認し、**「次へ」** をクリック します。     

1. **「バックアップを保持する期限」** ページで、既定の設定値を受け入れてから、**「次へ」** をクリックします。

1. 「**確認**」 ページで、**「バックアップ」** をクリックします。

1. バックアップが完了したら、「**閉じる**」 をクリックして Microsoft Azure Backup を閉じます。

1. Azure portal が表示されている Internet Explorer ウィンドウに切り替え、「Recovery Services コンテナー」 ブレードに戻り、「**バックアップ アイテム**」 をクリックします。 

1. 「**az104-10-rsv1 - バックアップ アイテム**」 ブレードで、「**Azure バックアップ エージェント**」 をクリックします。

1. 「**バックアップ アイテム (Azure バックアップ エージェント)**」 ブレードで、「**az104-10-vm1**」 の **C:\\** ドライブを参照するエントリがあることを確認します。

#### タスク 5: Azure Recovery Services エージェントを使用してファイルの回復を実行する (省略可能)

このタスクでは、Azure Recovery Services エージェントを使用してファイルの復元を実行します。

1. 「**az104-10-vm1** へのリモート デスクトップ」 セッションでファイル エクスプローラーを開いて、[**C:\Windows\System32\\drivers\etc\\**] フォルダーに移動し、「**hosts**」 ファイルを削除します。

1. 「Microsoft Azure Backup」 ウィンドウに切り替え、「**データの回復**」 をクリックします。これによって 「**データ回復ウィザード**」 が起動します。

1. 「**データ回復ウィザード**」 の 「**はじめに**」 ページで、「**このサーバー (az104-10-vm1.)**」 オプションが選択されていることを確認して、「**次へ**」 をクリックします。

1. 「**回復モードの選択**」 ページで、「**個々のファイルとフォルダ**」 オプションが選択されていることを確認し、「**次へ**」 をクリックします。

1. 「**ボリュームと日付の選択**」 ページの 「**ボリュームの選択**」 ドロップダウン リストで [**C:\\**] を選択し、使用可能なバックアップの選択を既定のまま使用し、「**マウント**」 をクリックします。 

    >**注**: マウント操作が完了するまで待ちます。これにはおよそ2分かかる場合があります。

1. 「**ファイルの参照と回復**」 ページで、回復ボリュームのドライブの文字をメモし、robocopy の使用に関するヒントを確認します。

1. 「**スタート**」 ボタンをクリックして 「**Windows システム**」 フォルダを展開し、「**コマンド プロンプト**」 をクリックします。

1. コマンド プロンプトで次のコマンドを実行して、復元する **hosts** ファイルを元の場所にコピーします。`[recovery_volume]` は、以前に特定した回復ボリュームのドライブの文字に置き換えます。

   ```
   robocopy [recovery_volume]:\Windows\System32\drivers\etc C:\Windows\system32\drivers\etc hosts /r:1 /w:1
   ```

1. データの回復ウィザードに戻り、**ファイルの参照と回復** で **マウント解除** をクリックし、確認画面が表示されたら**はい**をクリックします。        

1. リモート デスクトップ セッションを終了します。

#### タスク 6: Azure 仮想マシンのスナップショットを使用してファイルの回復を実行する (省略可能)

このタスクでは、Azure 仮想マシン レベルのスナップショット ベースのバックアップからファイルを復元します。

1. 課題コンピューターで実行されているブラウザー ウィンドウに切り替え、Azure portal を表示します。

1. Azure portalで 「**仮想マシン**」 を検索して選択し、「**仮想マシン**」 ブレードで 「**az104-10-vm0**」 をクリックします。

1. **az104-10-vm0** ブレードで、「**接続**」 をクリックし、ドロップダウン メニューで 「**RDP**」 をクリックし、「**RDP ファイルに接続**」 ブレードで 「**RDP ファイルのダウンロード**」 をクリックし、プロンプトに従ってリモート デスクトップ セッションを開始します。

    >**注**: この手順は、Windows コンピューターからリモート デスクトップ経由で接続することを指します。Mac では、Mac App Store からリモート デスクトップ クライアントを使用でき、Linux コンピューターでは、オープンソースの RDP クライアント ソフトウェアを使用できます。

    >**注**: ターゲットの仮想マシンに接続する際は、警告メッセージを無視できます。

1. プロンプトが表示されたら、ユーザー名 **Student** とパスワード **Pa55w.rd1234** を使用してログインします。

1. **az104-10-vm0** の Azure 仮想マシンへのリモート デスクトップ セッション 内の 「**サーバー マネージャー**」 ウィンドウで、「**ローカル サーバー**」 をクリック し、「**IE セキュリティ強化の構成**」 をクリック して、管理者向けに**オフ**にします。

1. **az104-10-vm0** へのリモート デスクトップ セッションで、「**スタート**」 ボタンをクリックし、「**Windows システム**」 フォルダを展開 して、「**コマンド プロンプト**」 をクリックします。

1. コマンド プロンプトから、次のコマンドを実行して、**ホスト**ファイルを削除します。 

   ```
   del C:\Windows\system32\drivers\etc\hosts
   ```
 
   >**注**: このファイルは、このタスクの後半で、Azure 仮想マシン レベルのスナップショット ベースのバックアップから復元します。

1. **az104-10-vm0** Azure 仮想マシンへのリモート デスクトップ セッション内で、Internet Explorer を起動し、[Azure Portal](https://portal.azure.com) を参照して、資格情報を使用してサインインします。    

1. Azure portal で 「**回復サービス コンテナー**」 を検索して選択し、「**回復サービス コンテナー**」 で 「**az104-10-rsv1**」 をクリックします。

1. 「**az104-10-rsv1** Recovery Services コンテナー」 ブレードの 「**保護されたアイテム**」 セクションで、「**バックアップ アイテム**」 をクリックします。

1. 「**az104-10-rsv1 - バックアップ アイテム**」 ブレードで、「**Azure 仮想マシン**」 をクリックします。 

1. 「**バックアップ アイテム (Azure 仮想マシン)**」 ブレードで、「**az104-10-vm0**」 をクリックします。

1. 「**az104-10-vm0** バックアップ アイテム」 ブレードで、「**ファイルの回復**」 をクリックします。

    >**注**: アプリケーションの整合性スナップショットに基づいて、バックアップの開始直後にリカバリを実行するオプションがあります。

1. 「**ファイルの回復**」 ブレードで、既定の復旧ポイントを受け入れ、「**実行可能ファイルのダウンロード**」 をクリック します。

    >**注**: このスクリプトは、スクリプトを実行しているオペレーティング システム内のローカル ドライブとして、選択した復旧ポイントからディスクをマウントします。

1. 「**ダウンロード**」 をクリックし、**IaaSVMILRExeForWindows.exe** を実行するか保存するかを確認するメッセージが表示されたら、「**実行**」 をクリックします。

1. ポータルからパスワードを入力するように求められたら、「**ファイルの回復**」 ブレードの 「**スクリプトを実行するパスワード**」 テキスト ボックスからパスワードをコピーして、コマンド プロンプトに貼り付けて **Enter** キーを押します。

    >**注**: これにより、マウントの進行状況を表示する Windows PowerShell ウィンドウが開きます。

    >**注**: この時点でエラー メッセージが表示された場合は、Internet Explorer ウィンドウを更新し、最後の 3 つの手順を繰り返します。

1. マウント処理が完了するのを待ち、Windows PowerShell ウィンドウで情報メッセージを確認し、**Windows** をホストしているボリュームに割り当てられているドライブ文字をメモし、File Explorer を起動します。

1. File Explorer で、前の手順で識別したオペレーティング システム ボリュームのスナップショットをホストするドライブ文字に移動し、その内容を確認します。

1. **コマンド プロンプト** ウィンドウに切り替えます。

1. コマンド プロンプトから、次のコマンドを実行して、復元する **ホスト**ファイルを元の場所にコピーします (`[os_volume]` を、以前に指定したオペレーティング システム ボリュームのドライブ文字に置き換えます)。 

   ```
   robocopy [os_volume]:\Windows\System32\drivers\etc C:\Windows\system32\drivers\etc hosts /r:1 /w:1
   ```

1. Azure portal の **「ファイルの回復** ブレードに戻り、**「ディスクのマウント解除」**をクリックします。

1. リモート デスクトップ セッションを終了します。

#### タスク 7: Azure Recovery Services のソフトデリート機能を確認する

1. 課題コンピューターの Azure Portal で、**Recovery Servicesコンテナー**を検索して選択 し、 **Recovery Services コンテナー**で **az104-10-rsv1** をクリックします。     

1. 「**az104-10-rsv1** Recovery Services コンテナー」 ブレードの 「**保護されたアイテム**」 セクションで、「**バックアップ アイテム**」 をクリックします。

1. 「**az104-10-rsv1 - バックアップ アイテム**」 ブレードで、「**Azure バックアップ エージェント**」 をクリックします。

1. 「**バックアップ項目 (Azure バックアップ エージェント)**」 ブレードで、**az104-10-vm1** のバックアップを表すエントリをクリックします。 

1. 「**az104-10-vm1. の C:\\**」 ブレードで、「**az104-10-vm1.**」 リンクをクリックします。

1. **az104-10-vm1.** 上保護されたサーバー ブレードで、「**削除**」 をクリックします。

1. 「**削除**」 ブレードで、次の設定を指定します。

    | 設定 | 値 |
    | --- | --- |
    | サーバー名を入力してください | **az104-10-vm1.** |
    | 理由 | **Recycling Dev/Test server** |
    | コメント | **az104 10 lab** |

    >**注**: サーバー名を入力するときは、末尾に必ずピリオドを含めます。

1. 「**このサーバに関連付けられた 1 つのバックアップ アイテムのバックアップ データがあります。"確認" をクリックするとすべてのクラウド バック アップデータが完全に削除されることを私は理解しています。**ラベルの隣のチェックボックスを有効にします。**このアクションは元に戻せません。このサブスクリプションの管理者に、この削除を通知する警告が送信される場合があります。**」というラベルの横にあるチェックボックスを有効にして、「**削除**」 をクリックします。

1. 「**az104-10-rsv1 - バックアップ アイテム**」 ブレードに戻り、「**Azure 仮想マシン**」 をクリックします。

1. 「**az104-10-rsv1 - バックアップ アイテム**」 ブレードで、「**Azure 仮想マシン**」 をクリックします。 

1. 「**バックアップ アイテム (Azure 仮想マシン)**」 ブレードで、「**az104-10-vm0**」 をクリックします。

1. 「**az104-10-vm0** バックアップ アイテム」 ブレードで、「**バックアップの停止**」 をクリックします。 

1. 「**バックアップの停止**」 ブレードで 「**バックアップ データの削除**」 を選択し、次の設定を指定して 「**バックアップの停止**」 をクリックします。

    | 設定 | 値 |
    | --- | --- |
    | バックアップ アイテムの名前を入力する | **az104-10-vm0** |
    | 理由 | **その他** |
    | コメント | **az104 10 の課題** |

1. 「**az104-10-rsv1 - バックアップ アイテム**」 ブレードに戻り、「**更新**」 をクリックします。

    >**注**: 「**Azure 仮想マシン**」 のエントリは、まだ **1** つのバックアップ アイテムを一覧に表示しています。

1. 「**Azure 仮想マシン**」 エントリをクリックし、「**バックアップ アイテム (Azure 仮想マシン)**」 ブレードで 「**az104-10-vm0**」 エントリをクリックします。

1. **az104-10-vm0** バックアップ アイテム ブレードで、削除したバックアップの 「**削除を取り消す**」 オプションがあることを確認します。 

    >**注**: この機能は、Azure 仮想マシンのバックアップにおいて既定で有効になっている、論理削除機能によって提供されています。

1. 「**az104-10-rsv1** Recovery Services コンテナー ブレードに戻り、「**設定**」 セクションで 「**プロパティ**」 をクリックします。

1. 「**az104-10-rsv1 - プロパティ**」 ブレードで、「**セキュリティ設定**」 ラベルの下にある 「**更新**」 リンクをクリックします。 

1. 「**セキュリティ設定**」 ブレードで、「**論理的な削除 (Azure 仮想マシンの場合)**」 を無効にして 「**保存**」 をクリックします。

    >**注**: これは、すでに論理削除の状態にあるアイテムには影響しません。

1. 「**セキュリティ設定**」 ブレードを閉じ、「**az104-10-rsv1** Recovery Services コンテナー」 ブレードに戻り、「**概要**」 をクリックします。

1. 「**az104-10-vm0** バックアップ アイテム」 ブレードに戻り、「**削除の取り消し**」 をクリックします。 

1. 「**az104-10-vm0 の削除を取り消す**」 ブレードで、「**削除の取り消し**」 をクリックします。 

1. 削除の取り消し操作が完了するのを待ち、必要に応じてブラウザーのページを更新し、「**az104-10-vm0** バックアップ アイテム」 ブレードに戻って 「**バックアップ データの削除**」 をクリックします。

1. 「**バックアップ データの削除**」 ブレードで次の設定を指定し、「**削除**」 をクリックします。

    | 設定 | 値 |
    | --- | --- |
    | バックアップ アイテムの名前を入力する | **az104-10-vm0** |
    | 理由 | **Others** |
    | コメント | **az104 10 lab** |

#### リソースのクリーンアップ

   >**注**: 新しく作成された Azure リソースのうち、使用しないリソースは必ず削除してください。使用していないリソースを削除することで、予期しないコストが発生しなくなります。

1. Azure portalの 「**Cloud Shell**」 ウインドウで、「**PowerShell**」 セッションを開始します。

1. 次のコマンドを実行して、このモジュールのラボ全体で作成されたすべてのリソース グループを一覧表示します。

   ```pwsh
   Get-AzResourceGroup -Name 'az104-10*'
   ```

1. 次のコマンドを実行して、このモジュールのラボ全体で作成したすべてのリソース グループを削除します。

   ```pwsh
   Get-AzResourceGroup -Name 'az104-10*' | Remove-AzResourceGroup -Force -AsJob
   ```

  > **注**: または、**AzureBackupRG_** プレフィックスが付いている自動生成リソース グループを削除することも可能です (これによる追加の課金は発生しません)。
  
  >**注意**: このコマンドは非同期処理で実行されるため (-AsJob パラメーターによって決まります)、同じ PowerShell セッション内ですぐに別の PowerShell コマンドを実行できますが、リソース グループが実際に削除されるまでに数分かかります。

#### レビュー

このラボで実行した内容は以下のとおりです。

- ラボ環境を構成しました。
- Recovery Services コンテナーを作成しました。
- Azure 仮想マシン レベルのバックアップを実施しました。
- ファイルとフォルダーのバックアップを実施しました。
- Azure Recovery Services エージェントを使用してファイルの回復を実施しました。
- Azure 仮想マシン のスナップショットを使用してファイルの回復を実施しました。
- AzureRecovery Services の論理削除の機能を確認しました。