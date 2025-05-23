---
lab:
  title: 'ラボ: Azure portal を使用してホスト プールとセッション ホストをデプロイする (AD DS)'
  module: 'Module 2: Implement an AVD Infrastructure'
---

# ラボ - Azure portal を使用してホスト プールとセッション ホストをデプロイする (AD DS)
# 受講生用ラボ マニュアル

## ラボの依存関係

- このラボで使用する Azure サブスクリプション。
- このラボで使用する Azure サブスクリプションの所有者または共同作成者ロールを持つ Microsoft アカウントまたは Microsoft Entra アカウントと、Azure サブスクリプションに関連付けられた Microsoft Entra テナントのグローバル管理者ロールを持つ Microsoft アカウントまたは Microsoft Entra アカウント。
- 完了したラボ: **Azure Virtual Desktop (AD DS) のデプロイを準備する**

## 推定所要時間

60 分

## ラボのシナリオ

Active Directory ドメイン サービス (AD DS) 環境でホスト プールとセッション ホストを作成して構成する必要があります。

## 目標
  
このラボを完了すると、次のことができるようになります。

- AD DS ドメインに Azure Virtual Desktop 環境を実装する
- AD DS ドメインの Azure Virtual Desktop 環境を検証する

## ラボ ファイル

- なし

## 手順

### 演習 1: AD DS ドメインに Azure Virtual Desktop 環境を実装する
  
この演習の主なタスクは次のとおりです。

1. AD DS ドメインと Azure サブスクリプションを準備して、Azure Virtual Desktop ホスト プールをデプロイする
1. Azure Virtual Desktop ホスト プールをデプロイする
1. Azure Virtual Desktop ホスト プールのセッション ホストを管理する
1. Azure Virtual Desktop アプリケーション グループを構成する
1. Azure Virtual Desktop ワークスペースを構成する

#### タスク 1: AD DS ドメインと Azure サブスクリプションを準備して Azure Virtual Desktop ホスト プールをデプロイする

1. ラボ コンピューターから Web ブラウザーを起動して [Azure portal]( ) に移動し、このラボで使用するサブスクリプションの所有者ロールを持つユーザー アカウントの資格情報を入力してサインインします。
1. Azure portal で、「**仮想マシン**」を検索して選択し、**[仮想マシン]** ウィンドウで **[az140-dc-vm11]** を選択します。
1. **[az140-dc-vm11]** ウィンドウで **[接続]** を選択し、ドロップダウン メニューで **[Bastion]** を選択し、**[az140-dc-vm11 \| 接続]** ウィンドウの **[Bastion]** タブで **[Bastion を使用する]** を選択します。
1. プロンプトが表示されたら、次の資格情報を入力し、**[接続]** を選択します。

   |設定|値|
   |---|---|
   |[ユーザー名]|**Student**|
   |パスワード|**Pa55w.rd1234**|

1. **az140-dc-vm11** への Bastion セッション内で、管理者として **Windows PowerShell ISE** を起動します。
1. **az140-dc-vm11** への Bastion セッション内で、**[管理者: Windows PowerShell ISE]** コンソールで、次のように実行して、Azure Virtual Desktop ホストのコンピューター オブジェクトをホストする組織単位を作成します。

   ```powershell
   New-ADOrganizationalUnit 'WVDInfra' -path 'DC=adatum,DC=com' -ProtectedFromAccidentalDeletion $false
   ```

1. [**管理者: Windows PowerShell ISE**] コンソールで以下を実行して、**aduser1** アカウントのユーザー プリンシパル名を識別します。

   ```powershell
   (Get-AzADUser -DisplayName 'aduser1').UserPrincipalName
   ```

   > **注**: この手順で識別したユーザー プリンシパル名を記録します。 このラボで後ほど必要になります。

1. [**管理者: Windows PowerShell ISE**] コンソールで以下を実行して、**Microsoft.DesktopVirtualization** リソース プロバイダーを登録します。

   ```powershell
   Register-AzResourceProvider -ProviderNamespace Microsoft.DesktopVirtualization
   ```

1. **az140-dc-vm11** への Bastion セッション内で、Microsoft Edge を起動して、[Azure portal](https://portal.azure.com) に移動します。 ダイアログが表示されたら、このラボで使用するサブスクリプションの所有者ロールをもつユーザー アカウントの Microsoft Entra 資格情報を指定してサインインします。
1. **az140-dc-vm11** への Bastion セッション内で、Azure portal の Azure portal ページの上部にある [**リソース、サービス、ドキュメントの検索**] テキスト ボックスを使用して、「**仮想ネットワーク**」を検索して移動し、[**仮想ネットワーク**] ブレードで [**az140-adds-vnet11**] を選択します。 
1. **[az140-adds-vnet11]** ウィンドウで **[サブネット]** を選択し、**[サブネット]** ウィンドウで **[+ サブネット]** を選択し、**[サブネットの追加]** ウィンドウで、次の設定を指定し (他のすべての設定は既定値のままにします)、**[保存]** をクリックします。

   |設定|値|
   |---|---|
   |名前|**hp1-Subnet**|
   |開始アドレス|**10.0.1.0**|
   |サイズ|**/24 (256 アドレス)**|

#### タスク 2: Azure Virtual Desktop ホスト プールをデプロイする

1. **az140-dc-vm11** への Bastion セッション内で、Azure portal を表示している Microsoft Edge ウィンドウで「**Azure Virtual Desktop**」を検索して選択し、**[Azure Virtual Desktop]** ブレードで **[ホスト プール]** を選択し、**[Azure Virtual Desktop \|ホスト プール]** ブレードで **[+ 作成]** を選択します。 
1. **[ホスト プールの作成]** ブレードの **[基本]** タブで、次の設定を指定し、**[次へ: 仮想マシン >]** を選択します (他の設定は既定値のままにします)。

   |設定|値|
   |---|---|
   |サブスクリプション|このラボで使用している Azure サブスクリプションの名前|
   |リソース グループ|**az140-21-RG** という名前の**新しい**リソース グループを作成します|
   |ホスト プール名|**az140-21-hp1**|
   |場所|このラボの最初の演習でリソースをデプロイした Azure リージョンの名前または類似したリージョンの名前 |
   |検証環境|**いいえ**|
   |優先するアプリ グループの種類|**デスクトップ**|
   |ホスト プールの種類|**プールされた**|
   |負荷分散アルゴリズム|**幅優先**|
   |最大セッションの制限|**12**|

   > **注**: ユーザーが RemoteApp アプリとデスクトップ アプリの両方を公開している場合、優先するアプリ グループの種類によって、フィードに表示されるアプリ グループが決まります。

1. **[ホスト プールの作成]** ブレードの **[仮想マシン]** タブで、次の設定を指定し、**[次へ: ワークスペース >]** を選択します (他の設定は既定値のままにします)。

   |設定|Value|
   |---|---|
   |[仮想マシンの追加]|**はい**|
   |リソース グループ|**既定はホスト プールと同じ**|
   |名前のプレフィックス|**az140-21-p1**|
   |仮想マシンのタイプ|Azure 仮想マシン|
   |仮想マシンの場所|前のラボでリソースをデプロイした Azure リージョンの名前|
   |可用性のオプション|**インフラストラクチャ冗長は必要ありません**|
   |セキュリティの種類|**トラステッド起動の仮想マシン**|
   |Image|**Windows 11 Enterprise マルチセッション + Microsoft 365 Apps、バージョン 22H2**|
   |[仮想マシンのサイズ]|**Standard DC2s_v3**|
   |[Number of VMs](VM の数)|**2**|
   |OS ディスクの種類|**Standard SSD**|
   |OS ディスク サイズ|**既定のサイズ (128 GB)**|
   |ブート診断|**マネージド ストレージ アカウントで有効にする (推奨)**|
   |仮想ネットワーク|**az140-adds-vnet11**|
   |Subnet|**hp1-Subnet (10.0.1.0/24)**|
   |ネットワーク セキュリティ グループ|**Basic**|
   |パブリック インバウンド ポート|**いいえ**|
   |参加したいディレクトリを選択する|**Active Directory**|
   |AD ドメイン参加 UPN|**student@adatum.com**|
   |パスワード|**Pa55w.rd1234**|
   |[パスワードの確認入力]|**Pa55w.rd1234**|
   |特定のドメインまたはユニット|**はい**|
   |参加するドメイン|**adatum.com**|
   |組織単位のパス|**OU=WVDInfra,DC=adatum,DC=com**|
   |ユーザー名|**Student**|
   |パスワード|**Pa55w.rd1234**|
   |[パスワードの確認入力]|**Pa55w.rd1234**|

1. **[ホスト プールの作成]** ブレードの **[ワークスペース]** タブで、次の設定を確認し、**[確認と作成]** を選択します。

   |設定|Value|
   |---|---|
   |デスクトップ アプリ グループを登録する|**いいえ**|

1. **[ホストプールの作成]** ブレードの **[確認および作成]** タブで、**[作成]** を選択します。

   > **注**: デプロイが完了するまで待ちます。 これには 10 から 15 分ほどかかる場合があります。

#### タスク 3: Azure Virtual Desktop ホスト プール セッション ホストを管理する

1. **az140-dc-vm11** への Bastion セッション内で、Azure portal を表示している Web ブラウザー ウィンドウで、「**Azure Virtual Desktop**」を検索して選択し、**[Azure Virtual Desktop]** ブレードの垂直メニューバーの **[管理]** セクションで、**[ホスト プール]** を選択します。
1. **[Azure Virtual Desktop \| ホスト プール]** ブレードで、ホスト プールの一覧から **[az140-21-hp1]** を選択します。
1. **[az140-21-hp1]** ブレードの垂直メニューバーの **[管理] セクション**で、**[セッション ホスト]** を選択し、プールが 2 つのホストで構成されていることを確認します。 
1. **[az140-21-hp1 \| セッション ホスト]** ブレードで、**[+ 追加]** を選択します。
1. **[仮想マシンをホスト プールに追加]** ブレードの **[基本]** タブで、事前構成された設定を確認し、**[次へ: 仮想マシン]** を選択します。
1. **[仮想マシンをホスト プール ブレードに追加]** ブレードの **[Virtual Machines]** タブで、次の設定を指定し、**[確認および作成]** を選択します (他は既定値のままにします)。

   |設定|値|
   |---|---|
   |リソース グループ|**az140-21-RG**|
   |名前のプレフィックス|**az140-21-p1**|
   |仮想マシンの場所|最初の 2 つのセッション ホスト VM をデプロイした Azure リージョンの名前|
   |可用性のオプション|**インフラストラクチャ冗長は必要ありません**|
   |セキュリティの種類|**トラステッド起動の仮想マシン**|
   |Image|**Windows 11 Enterprise マルチセッション + Microsoft 365 Apps、バージョン 22H2**|
   |[Number of VMs](VM の数)|**1**|
   |OS ディスクの種類|**Standard SSD**|
   |OS ディスク サイズ|**既定のサイズ (128 GB)**|
   |ブート診断|**マネージド ストレージ アカウントで有効にする (推奨)**|
   |仮想ネットワーク|**az140-adds-vnet11**|
   |Subnet|**hp1-Subnet (10.0.1.0/24)**|
   |ネットワーク セキュリティ グループ|**Basic**|
   |パブリック インバウンド ポート|**いいえ**|
   |参加したいディレクトリを選択する|**Active Directory**|
   |AD ドメイン参加 UPN|**student@adatum.com**|
   |パスワード|**Pa55w.rd1234**|
   |[パスワードの確認入力]|**Pa55w.rd1234**|
   |特定のドメインまたはユニット|**はい**|
   |参加するドメイン|**adatum.com**|
   |組織単位のパス|**OU=WVDInfra,DC=adatum,DC=com**|   
   |ユーザー名|**Student**|
   |パスワード|**Pa55w.rd1234**|
   |[パスワードの確認入力]|**Pa55w.rd1234**|

   > **注**: このように、既存のプールにセッション ホストを追加するときに、VM のイメージとプレフィックスを変更できます。 一般に、プール内のすべての VM を置き換える予定がない限り、これは推奨されません。 

1. **[仮想マシンをホスト プール ブレードに追加]** ブレードの **[確認および作成]** タブで、**[作成]** を選択します

   > **注**: デプロイが完了するまで待ってから、次のタスクに進んでください。 これには 10 分ほどかかる場合があります。 

#### タスク 4: Azure Virtual Desktop アプリケーション グループを構成する

1. **az140-dc-vm11** への Bastion セッション内で、Azure portal を表示する Web ブラウザー ウィンドウで「**Azure Virtual Desktop**」を検索して選択し、**[Azure Virtual Desktop]** ブレードで **[アプリケーション グループ]** を選択します。
1. **[Azure Virtual Desktop \| アプリケーション グループ]** ブレードで、既存の自動生成された **az140-21-hp1-DAG** デスクトップ アプリケーション グループに注意して、それを選択します。 
1. **[az140-21-hp1-DAG]** ブレードで **[割り当て]** を選択します。
1. **[az140-21-hp1-DAG \| 割り当て]** ブレードで **[+ 追加]** を選択します。
1. **[Microsoft Entra ユーザーまたはユーザー グループの選択]** ブレードで、**[グループ]** を選択したら、**[az140-wvd-pooled]** を選択し、**[選択]** をクリックします。
1. **[Azure Virtual Desktop \| アプリケーション グループ]** ブレードに戻り、**[+ 作成]** を選択します。 
1. **[アプリケーション グループの作成]** ブレードの **[基本]** タブで、次の設定を指定して、**[次へ: アプリケーション >]** を選択します。

   |設定|値|
   |---|---|
   |サブスクリプション|このラボで使用している Azure サブスクリプションの名前|
   |リソース グループ|**az140-21-RG**|
   |ホスト プール|**az140-21-hp1**|
   |アプリケーション グループの種類|**リモート アプリ**|
   |アプリケーション グループ名|**az140-21-hp1-Office365-RAG**|

1. **[アプリケーション グループの作成]** ブレードの **[アプリケーション]** タブで、**[+ アプリケーションの追加]** を選択します。
1. **[アプリケーションの追加]** ブレードで、次の設定を指定し、**[確認と追加]** を選択し、**[追加]** を選択します。

   |設定|Value|
   |---|---|
   |アプリケーション ソース|**[スタート] メニュー**|
   |アプリケーション|**Word**|
   |説明|**Microsoft Word**|
   |コマンド ライン必須|**いいえ**|

1. **[アプリケーション グループの作成]** ブレードの **[アプリケーション]** タブに戻り、**[+ アプリケーションの追加]** を選択します。
1. **[アプリケーションの追加]** ブレードで、次の設定を指定し、**[確認と追加]** を選択し、**[追加]** を選択します。

   |設定|Value|
   |---|---|
   |アプリケーション ソース|**[スタート] メニュー**|
   |アプリケーション|**Excel**|
   |説明|**Microsoft Excel**|
   |コマンド ライン必須|**いいえ**|

1. **[アプリケーション グループの作成]** ブレードの **[アプリケーション]** タブに戻り、**[+ アプリケーションの追加]** を選択します。
1. **[アプリケーションの追加]** ブレードで、次の設定を指定し、**[確認と追加]** を選択し、**[追加]** を選択します。

   |設定|Value|
   |---|---|
   |アプリケーション ソース|**[スタート] メニュー**|
   |アプリケーション|**PowerPoint**|
   |説明|**Microsoft PowerPoint**|
   |コマンド ライン必須|**いいえ**|

1. **[アプリケーション グループの作成]** ブレードの **[アプリケーション]** タブに戻り、**[次へ: 割り当て >]** を選択します。
1. [**アプリケーション グループの作成**] ブレードの [**割り当て**] タブで、[**+ Microsoft Entra ユーザーまたはユーザー グループの追加**] を選択します。
1. **[Microsoft Entra ユーザーまたはユーザー グループの選択]** ブレードで、**[グループ]** を選択したら、**[az140-wvd-remote-app]** を選択し、**[選択]** をクリックします。
1. **[アプリケーション グループの作成]** ブレードの **[割り当て]** タブに戻り、**[次へ: ワークスペース >]** を選択します。
1. **[ワークスペースの作成]** ブレードの **[ワークスペース]** タブで、次の設定を指定し、**[確認および作成]** を選択します。

   |設定|Value|
   |---|---|
   |アプリケーション グループの登録|**いいえ**|

1. **[アプリケーション グループの作成]** ブレードの **[確認および作成]** タブで、**[作成]** を選択します。

   > **注**: アプリケーション グループが作成されるまで待ちます。 これにかかる時間は 1 分未満です。 

   > **注**: 次に、アプリケーション ソースとしてのファイル パスに基づいてアプリケーション グループを作成します。

1. **az140-dc-vm11** への Bastion セッション内で、「**Azure Virtual Desktop**」を検索して選択し、**[Azure Virtual Desktop]** ブレードで **[アプリケーション グループ]** を選択します。
1. **[Azure Virtual Desktop \| アプリケーション グループ]** ブレードで、**[+ 作成]** を選択します。 
1. **[アプリケーション グループの作成]** ブレードの **[基本]** タブで、次の設定を指定して、**[次へ: アプリケーション >]** を選択します。

   |設定|値|
   |---|---|
   |サブスクリプション|このラボで使用している Azure サブスクリプションの名前|
   |リソース グループ|**az140-21-RG**|
   |ホスト プール|**az140-21-hp1**|
   |アプリケーション グループの種類|**RemoteApp**|
   |アプリケーション グループ名|**az140-21-hp1-Utilities-RAG**|

1. **[アプリケーション グループの作成]** ブレードの **[アプリケーション]** タブで、**[+ アプリケーションの追加]** を選択します。
1. **[アプリケーションの追加]** ブレードの **[基本]** タブで次の設定を指定し、**[次へ]** を選択します。

   |設定|Value|
   |---|---|
   |アプリケーション ソース|**ファイル パス**|
   |アプリケーション パス|**C:\Windows\system32\cmd.exe**|
   |アプリケーション識別子|**コマンド プロンプト**|
   |[表示名]|**コマンド プロンプト**|
   |説明|**Windows コマンド プロンプト**|
   |コマンド ライン必須|**いいえ**|

1. **[アイコン]** タブで、次の設定を指定し、**[確認と追加]** を選択し、**[追加]** を選択します。

   |設定|Value|
   |---|---|
   |Icon path (アイコン パス)|**C:\Windows\system32\cmd.exe**|
   |Icon Index (アイコン インデックス)|0|

1. **[アプリケーション グループの作成]** ブレードの **[アプリケーション]** タブに戻り、**[次へ: 割り当て >]** を選択します。
1. [**アプリケーション グループの作成**] ブレードの [**割り当て**] タブで、[**+ Microsoft Entra ユーザーまたはユーザー グループの追加**] を選択します。
1. **[Microsoft Entra ユーザーまたはユーザー グループの選択]** ブレードで、**[グループ]** を選択したら、**[az140-wvd-remote-app]** と **[az140-wvd-admins]** を選択し、**[選択]** をクリックします。
1. **[アプリケーション グループの作成]** ブレードの **[割り当て]** タブに戻り、**[次へ: ワークスペース >]** を選択します。
1. **[ワークスペースの作成]** ブレードの **[ワークスペース]** タブで、次の設定を指定し、**[確認および作成]** を選択します。

   |設定|Value|
   |---|---|
   |アプリケーション グループの登録|**いいえ**|

1. **[アプリケーション グループの作成]** ブレードの **[確認および作成]** タブで、**[作成]** を選択します。

#### タスク 5: Azure Virtual Desktop ワークスペースを構成する

1. **az140-dc-vm11** への Bastion セッション内で、Azure portal を表示している Microsoft Edge ウィンドウで、「**Azure Virtual Desktop**」を検索して選択し、**[Azure Virtual Desktop]** ブレードで **[ワークスペース]** を選択します。
1. **[Azure Virtual Desktop \| ワークスペース]** ブレードで、**[+ 作成]** を選択します。 
1. **[ワークスペースの作成]** ブレードの **[基本]** タブで、次の設定を指定して、**[次へ: アプリケーション グループ >]** を選択します。

   |設定|値|
   |---|---|
   |サブスクリプション|このラボで使用している Azure サブスクリプションの名前|
   |リソース グループ|**az140-21-RG**|
   |ワークスペース名|**az140-21-ws1**|
   |フレンドリ名|**az140-21-ws1**|
   |場所|このラボの最初の演習でリソースをデプロイした Azure リージョンの名前または類似したリージョンの名前|

1. **[ワークスペースの作成]** ブレードの **[アプリケーション グループ]** タブで、次の設定を指定します。

   |設定|Value|
   |---|---|
   |アプリケーション グループを登録する|**はい**|

1. **[ワークスペースの作成]** ブレードの **[ワークスペース]** タブで、**[+ アプリケーション グループの登録]** を選択します。
1. **[アプリケーション グループの追加]** ブレードで、**[az140-21-hp1-DAG]**、**[az140-21-hp1-Office365-RAG]**、**[az140-21-hp1-Utilities-RAG]** の各エントリの横にあるプラス記号を選択し、**[選択]** をクリックします。 
1. **[ワークスペースの作成]** ブレードの **[アプリケーション グループ]** タブに戻り、**[確認および作成]** を選択します。
1. **[ワークスペースの作成]** ブレードの **[確認および作成]** タブで、**[作成]** を選択します。

### 演習 2: Azure Virtual Desktop 環境の検証
  
この演習の主なタスクは次のとおりです。

1. Windows 10 コンピューターに Microsoft リモート デスクトップ クライアント (MSRDC) をインストールする
1. Azure Virtual Desktop ワークスペースに登録する
1. Azure Virtual Desktop アプリのテスト

#### タスク 1: Microsoft リモート デスクトップ クライアント (MSRDC) を Windows 10 コンピューターにインストールする

1. **az140-dc-vm11** への Bastion セッション内で、Azure portal を表示しているブラウザー ウィンドウで、「**仮想マシン**」を検索して選択し、**[仮想マシン]** ブレードで **[az140-cl-vm11]** エントリを選択します。
1. **[az140-cl-vm11]** ブレードの **[操作]** セクションで、**[コマンドの実行]** を選択します。 
1. **[az140-cl-vm11 \| コマンドの実行]** ブレードで、**[EnableRemotePS]** を選択し、**[実行]** を選択します。 

   > **注**: コマンドが完了するのを待ってから、次の手順に進みます。 これには 1 分ほどかかる場合があります。 ドメイン プロファイル以外の、使用中のパブリック プロファイルをアドレス指定したエラーが赤い文字のテキストで表示される場合は、無視して、次の手順に進んでください。

1. **az140-dc-vm11** への Bastion セッション内で、**[管理者: Windows PowerShell ISE]** スクリプト ペインで、次のように実行して、**ADATUM\\az140-wvd-users** のすべてのメンバーを、Windows 10 が実行されている Azure VM **az140-cl-vm11** の **[リモート デスクトップ ユーザー]** グループに追加します。これは、ラボ「**Azure Virtual Desktop (AD DS) のデプロイを準備する**」でデプロイしたものです。

   ```powershell
   $computerName = 'az140-cl-vm11'
   Invoke-Command -ComputerName $computerName -ScriptBlock {Add-LocalGroupMember -Group 'Remote Desktop Users' -Member 'ADATUM\az140-wvd-users'}
   ```

1. ラボ コンピューターから、Azure portal を表示している Web ブラウザーの画面を表示しているラボ コンピューターに切り替え、「**仮想マシン**」を検索して選択し、**[仮想ネットワーク]** ブレードから **[az140-cl-vm11]** エントリを選択します。
1. **[az140-cl-vm11]** ブレードで **[接続]** を選択し、ドロップダウン メニューで **[Bastion 経由で接続する]** を選択します。
1. プロンプトが表示されたら、次の資格情報を入力し、**[接続]** を選択します。

   |設定|値|
   |---|---|
   |ユーザー名|**Student@adatum.com**|
   |パスワード|**Pa55w.rd1234**|

   > **注** 最初のサインインが完了するまでに 5 から 10 分かかる場合があります。

1. **az140-cl-vm11** への Bastion セッション内で、Microsoft Edge を起動し、[[Windows デスクトップ クライアントのダウンロード]](https://go.microsoft.com/fwlink/?linkid=2068602) ページに移動し、プロンプトが表示されたら、**[実行]** を選択してインストールを開始します。 **[リモート デスクトップ セットアップ]** ウィザードの **[インストールのスコープ]** ページで、**[このコンピューターのすべてのユーザー用にインストール]** オプションを選択し、**[インストール]** をクリックします。 ユーザー アカウント制御によって管理者の資格情報の入力を求められたら、ユーザー名 **ADATUM\\Student** で、**Pa55w.rd1234** をパスワードとして使用して認証します。
1. インストールが完了したら、**[セットアップの終了時にリモート デスクトップを起動する]** チェックボックスがオンになっていることを確認し、**[完了]** をクリックしてリモート デスクトップ クライアントを起動します。

#### タスク 2: Azure Virtual Desktop ワークスペースに登録する

1. **[リモート デスクトップ]** クライアント ウィンドウで、**[サブスクライブ]** を選択し、プロンプトが表示されたら、**aduser1** の資格情報を使用してサインインします。ここでは、このラボの前半で特定された userPrincipalName 属性を指定し、このアカウントの作成時に設定したパスワードを入力します。

   > **注**:または、[**リモート デスクトップ**] クライアント ウィンドウで [**URL でサブスクライブ**] を選択し、[**ワークスペースのサブスクライブ**] ウィンドウの [**メールまたはワークスペース URL**] に「**https://client.wvd.microsoft.com/api/arm/feeddiscovery**」と入力して、[**次へ**] を選択します。ダイアログが表示されたら、**aduser1** の資格情報を使用してサインインします (userPrincipalName 属性をユーザー名として使用し、このアカウントの作成時に設定したパスワードを使用します)。 

1. **[リモート デスクトップ]** ページに SessionDesktop が表示されていることを確認し、さらに、自動生成されワークスペースに公開された az140-21-hp1-DAG デスクトップ アプリケーション グループに SessionDesktop が含まれており、グループ メンバーシップを介してユーザー アカウント **aduser1** に関連付けられていることを確認します。 

   > **注**: 現在、ホスト プールの **[優先するアプリ グループの種類]** が **[デスクトップ]** に設定されているため、これは想定内の結果です。

#### タスク 3: Azure Virtual Desktop アプリのテスト

1. **az140-cl-vm11** への Bastion セッション内の **[リモート デスクトップ クライアント]** ウィンドウのアプリケーションのリストで、**[SessionDesktop]** をダブルクリックし、リモート デスクトップ セッションが起動することを確認します。 

   > **注**: 最初はアプリケーションが起動するまでに数分かかる場合がありますが、その後、アプリケーションの起動は大幅に速くなります。

   > **注**: **[Microsoft Teams へようこそ]** サインイン プロンプトが表示されたら、閉じてください。

1. **[既定のデスクトップ]** セッションで、**[スタート]** を右クリックし、**[ファイル名を指定して実行]** を選択し、**[ファイル名を指定して実行]** ダイアログ ボックスの **[開く]** テキストボックスに「**cmd**」と入力して、**[OK]** を選択します。 
1. **[セッション デスクトップ]** セッション内のコマンド プロンプトで、｢**hostname**」と入力し、**Enter** キーを押して、リモート デスクトップ セッションが実行されているコンピューターの名前を表示します。
1. 表示される名前が **az140-21-p1-0**、**az140-21-p1-1**、**az140-21-p1-2** のいずれかであることを確認します。
1. コマンド プロンプトで、「**logoff**」と入力し、**Enter** キーを押して、セッション デスクトップからログオフします。

   > **注**: 次に、**[優先するアプリ グループの種類]** を **[RemoteApp]** に設定して変更します。

1. ラボ コンピューターから、**az140-dc-vm11** への Bastion セッションに切り替えます。
1. **az140-dc-vm11** への Bastion セッション内で、Azure portal を表示している Web ブラウザー ウィンドウで、「**Azure Virtual Desktop**」を検索して選択し、**[Azure Virtual Desktop]** ブレードの垂直メニューバーの **[管理]** セクションで、**[ホスト プール]** を選択します。
1. **[Azure Virtual Desktop \| ホスト プール]** ブレードで、ホスト プールの一覧から **[az140-21-hp1]** を選択します。
1. **[az140-21-hp1]** ブレードの垂直メニュー バーの **[設定]** セクションで、**[プロパティ]** を選択し、**[優先するアプリ グループの種類]** で **[リモート アプリ]** を選択して、**[保存]** を選択します。 
1. ラボ コンピューターから、**az140-dc-vm11** への Bastion セッションに切り替えます。
1. **az140-cl-vm11** への Bastion セッション内の **[リモート デスクトップ]** クライアント ウィンドウで、右上隅にある省略記号を選択し、ドロップダウン メニューで **[更新]** を選択します。
1. **[リモート デスクトップ]** ページに個々のアプリが表示されていることを確認し、さらに、作成およびワークスペースに公開した 2 つのアプリケーション グループにこれらのアプリが含まれており、グループ メンバーシップを介してユーザー アカウント **aduser1** に関連付けられていることを確認します。 

   > **注**: 現在、ホスト プールの **[優先するアプリ グループの種類]** が **[RemoteApp]** に設定されているため、これは想定内の結果です。

1. **az140-cl-vm11** への Bastion セッション内の **[リモート デスクトップ クライアント]** ウィンドウのアプリケーションのリストで、**[コマンド プロンプト]** をダブルクリックし、**[コマンド プロンプト]** ウィンドウが起動することを確認します。 認証を求められたら、**aduser1** ユーザー アカウントの作成時に設定したパスワードを入力して、**[Remember me]** チェックボックスを選択し、**[OK]** を選択します。
1. コマンド プロンプトで、「**ホスト名**」と入力し、**Enter** キーを押して、コマンド プロンプトが実行されているコンピューターの名前を表示します。

   > **注**: 表示される名前が **az140-cl-vm11** ではなく、**az140-21-p1-0**、**az140-21-p1-1**、**az140-21-p1-2** であることを確認します。

1. コマンド プロンプトで、「**ログオフ**」と入力し、**Enter** キーを押して現在のリモート アプリ セッションからログオフします。

### 演習 3: ラボでプロビジョニングされた Azure VM を停止して割り当てを解除する

この演習の主なタスクは次のとおりです。

1. ラボでプロビジョニングされた Azure VM を停止して割り当てを解除する

>**注**: この演習では、関連するコンピューティング料金を最小限に抑えるために、このラボでプロビジョニングされた Azure VM の割り当てを解除します

#### タスク 1: ラボでプロビジョニングされた Azure VM の割り当てを解除する

1. ラボ コンピューターに切り替え、Azure portal が表示されている Web ブラウザー ウィンドウで、[**Cloud Shell**] ウィンドウの **PowerShell** シェル セッションを開きます。
1. [Cloud Shell] ウィンドウの PowerShell セッションから、次を実行して、このラボで作成されたすべての Azure VM を一覧表示します。

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG'
   ```

1. [Cloud Shell] ウィンドウの PowerShell セッションから、次を実行して、このラボで作成したすべての Azure VM を停止して割り当てを解除します。

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Stop-AzVM -NoWait -Force
   ```

   >**注**: このコマンドは非同期で実行されるため (-NoWait パラメーターによって決定されます)、同じ PowerShell セッション内で直後に別の PowerShell コマンドを実行できますが、Azure VM が実際に停止されて割り当てが解除されるまでに数分かかります。
