---
lab:
  title: 'ラボ: イメージ テンプレートを使用してカスタム セッション ホスト イメージを作成する'
  module: 'Module 1.5: Create and manage session host images'
---

# ラボ - イメージ テンプレートを使用してカスタム セッション ホスト イメージを作成する
# 受講生用ラボ マニュアル

## ラボの依存関係

- このラボで使用する Azure サブスクリプション。
- このラボで使用する Azure サブスクリプションの所有者ロールを持ち、その Azure サブスクリプションに関連付けられた Entra テナントにデバイスを参加させるのに十分なアクセス許可を持つ Microsoft Entra ユーザー アカウント。

## 推定所要時間

90 分 (イメージのビルドが完了するまでの待機時間は約 45 分)

## ラボのシナリオ

あなたは、Azure Virtual Desktop 環境の実装を計画しています。 Azure Virtual Desktop セッション ホストをデプロイする際は、カスタム仮想マシン イメージを使用する必要があります。

## 目標
  
このラボを完了すると、次のことができるようになります。

- イメージ テンプレートを使用して、Azure Virtual Desktop のカスタム セッション ホスト イメージを作成する

## ラボ ファイル

- なし

## 手順

### 演習 1: イメージ テンプレートを使用してカスタム セッション ホスト イメージを作成する

この演習の主なタスクは次のとおりです。

1. 必要なリソース プロバイダーを登録する
1. ユーザー割り当てマネージド ID を作成する
1. カスタムの Azure ロールベース アクセス制御 (RBAC) ロールを作成する
1. ホスト イメージのプロビジョニング関連リソースに対するアクセス許可を設定する
1. Azure Compute Gallery インスタンスとイメージ定義を作成する
1. カスタム イメージ テンプレートを作成する
1. カスタム イメージをビルドする
1. カスタム イメージを使用してセッション ホストをデプロイする

> **注**: カスタム イメージ テンプレートを作成するには、まず、次のようないくつかの前提条件を満たす必要があります。

- 必要なリソース プロバイダーをすべて登録する
- ユーザー割り当てマネージド ID を作成する
- カスタムの Azure ロールベース アクセス制御 (RBAC) ロールを使用して、ユーザー割り当てマネージド ID に必要なアクセス許可を付与する
- Azure Compute Gallery を使用してイメージを配布する場合は、イメージ定義と共にそのインスタンスを作成する必要がある

#### タスク 1: 必要なリソース プロバイダーを登録する

1. 必要に応じて、ラボ コンピューターから Web ブラウザーを起動して Azure portal に移動し、このラボで使用するサブスクリプションの所有者ロールを持つユーザー アカウントの資格情報を入力してサインインします。

    > **注**: ラボ セッション ウィンドウの右側にある [リソース] タブに表示されている `User1-` アカウントの資格情報を使用します。

1. Azure portal 上の Azure Cloud Shell で PowerShell セッションを開始します。

    > **注**: メッセージが表示されたら、**[概要]** ペインの **[サブスクリプション]** ドロップダウン リストで、このラボで使用している Azure サブスクリプションの名前を選択し、**[適用する]** を選択します。

1. [Azure Cloud Shell] ペインの PowerShell セッションで、次のコマンドを実行して、**Microsoft.DesktopVirtualization** リソース プロバイダーを登録します。

    ```powershell
    Register-AzResourceProvider -ProviderNamespace Microsoft.DesktopVirtualization
    Register-AzResourceProvider -ProviderNamespace Microsoft.VirtualMachineImages
    Register-AzResourceProvider -ProviderNamespace Microsoft.Storage
    Register-AzResourceProvider -ProviderNamespace Microsoft.Compute
    Register-AzResourceProvider -ProviderNamespace Microsoft.Network
    Register-AzResourceProvider -ProviderNamespace Microsoft.KeyVault
    Register-AzResourceProvider -ProviderNamespace Microsoft.ContainerInstance
    ```

    > **注**: 登録が完了するまで待つ必要はありません。 これには 5 分ほどかかる場合があります。

1. Azure Cloud Shell ウィンドウを閉じます。

#### タスク 2: ユーザー割り当てマネージド ID を作成する

1. ラボ コンピューターの Azure portal が表示されている Web ブラウザーで、**マネージド ID** を検索して選択します。
1. **[マネージド ID]** ページで、 **[+ 作成]** を選択します。
1. **[ユーザー割り当てマネージド ID の作成]** ページの **[基本]** タブで、以下の設定を指定して、**[確認および作成]** を選択します。

    > **注**: **[名前]** 値を設定する際は、ラボ セッション ウィンドウの右側にある [リソース] タブに切り替えて、*User1-* と *@* 文字の間の文字列を特定します。 この文字列を使用して、*ランダム* プレースホルダーを置き換えます。

    |設定|値|
    |---|---|
    |サブスクリプション|このラボで使用している Azure サブスクリプションの名前|
    |リソース グループ|新しいリソース グループの名前 **az140-15a-RG**|
    |リージョン|Azure Virtual Desktop 環境をデプロイする Azure リージョンの名前|
    |Name|**az140**-*random*-**uami**|

1. **[確認および作成]** タブで、 **[作成]** を選択します。

    >**注**: ユーザー割り当てマネージド ID のプロビジョニングが完了するまで待つ必要はありません。 通常は数秒で完了します。

#### タスク 3: カスタムの Azure ロールベース アクセス制御 (RBAC) ロールを作成する

>**注**: カスタムの Azure ロールベース アクセス制御 (RBAC) ロールは、前のタスクで作成したユーザー割り当てマネージド ID に適切なアクセス許可を割り当てるために使用されます。

1. ラボ コンピューターの Azure portal が表示されている Web ブラウザーで、Azure Cloud Shell の PowerShell セッションを開始します。

1. Azure Cloud Shell ペインの PowerShell セッションで、次のコマンドを実行して、このラボに使用される Azure サブスクリプションの **Id** プロパティの値を特定し、それを **$subscriptionId** 変数に格納します。

    ```powershell
    $subscriptionId = (Get-AzSubscription).Id
    ```

1. 次のコマンドを実行して、割り当て可能なスコープ値を含む新しいカスタム ロールのロール定義を作成し、それを **$jsonContent** 変数に格納します (*random* プレースホルダーは、前のタスクで特定した文字列と同じものに置き換えてください)。

    ```powershell
    $jsonContent = @"
    {
      "Name": "Desktop Virtualization Image Creator (random)",
      "IsCustom": true,
      "Description": "Create custom image templates for Azure Virtual Desktop images.",
      "Actions": [
        "Microsoft.Compute/galleries/read",
        "Microsoft.Compute/galleries/images/read",
        "Microsoft.Compute/galleries/images/versions/read",
        "Microsoft.Compute/galleries/images/versions/write",
        "Microsoft.Compute/images/write",
        "Microsoft.Compute/images/read",
        "Microsoft.Compute/images/delete"
      ],
      "NotActions": [],
      "DataActions": [],
      "NotDataActions": [],
      "AssignableScopes": [
        "/subscriptions/$subscriptionId",
        "/subscriptions/$subscriptionId/resourceGroups/az140-15b-RG"
      ]
    }
    "@
    ```

1. 次のコマンドを実行して、**CustomRole.json** という名前のファイルに **$jsonContent** 変数の内容を格納します。

    ```powershell
    $jsonContent | Out-File -FilePath 'CustomRole.json'
    ```

1. 次のコマンドを実行して、カスタム ロールを作成します。

    ```powershell
    New-AzRoleDefinition -InputFile ./CustomRole.json
    ```

1. Azure Cloud Shell ウィンドウを閉じます。

#### タスク 4: ホスト イメージのプロビジョニング関連リソースに対するアクセス許可を設定する

1. ラボ コンピューターの Azure portal が表示されている Web ブラウザーで、**リソース グループ**を検索して選択し、**[リソース グループ]** ページで **[+ 作成]** を選択します。
1. **[リソース グループの作成]** ページの **[基本]** タブで、以下の設定を指定して、**[確認および作成]** を選択します。

    |設定|値|
    |---|---|
    |サブスクリプション|このラボで使用している Azure サブスクリプションの名前|
    |リソース グループ|新しいリソース グループの名前 **az140-15b-RG**|
    |リージョン|Azure Virtual Desktop 環境をデプロイする Azure リージョンの名前|

1. **[確認および作成]** タブで、 **[作成]** を選択します。
1. **[リソース グループ]** ページを更新し、リソース グループの一覧で **az140-15b-RG** を選択します。
1. **[az140-15b-RG]** ページの縦型ナビゲーション メニューで、**[アクセス制御 (IAM)]** を選択します。
1. **[az140-15b-RG \| アクセス制御 (IAM)]** ページで **[+ 追加]** を選択し、ドロップダウン メニューで **[ロールの割り当ての追加]** を選択します。
1. **[ロールの割り当ての追加]** ページの **[ロール]** タブで、**[職務権限ロール]** タブが選択されていることを確認し、検索テキストボックスに「**デスクトップ仮想化 Image Creator** (*random*)」と入力し、結果の一覧で**デスクトップ仮想化 Image Creator** (*random*) を選択して、**[次へ]** を選択します。

    >**注**: *random* プレースホルダーは、新しいカスタム RBAC ロールを定義する際に使用した文字列と同じものに置き換えてください。

1. **[ロールの割り当ての追加]** ページの **[メンバー]** タブで、**[マネージド ID]** オプションを選択して、**[+ メンバーの選択]** をクリックし、**[マネージド ID の選択]** ペインの **[マネージド ID]** ドロップダウン リストで、**ユーザー割り当てマネージド ID** を選択します。ユーザー割り当てマネージド ID の一覧で **az140**-*random*-**uami** を選択し (*random * プレースホルダーは、新しいカスタム RBAC ロールを定義する際に使用した文字列と同じものを表します)、**[選択]** をクリックします。
1. **[ロールの割り当ての追加]** ページの **[メンバー]** タブに戻り、**[確認と割り当て]** を選択します。
1. **[確認と割り当て]** タブで、 **[確認と割り当て]** を選択します。 

#### タスク 5: Azure Compute Gallery インスタンスとイメージ定義を作成する

1. ラボ コンピューターの Azure portal が表示されている Web ブラウザーで、**Azure Compute Gallery** を検索して選択し、**[Azure Compute Gallery]** ページで **[+ 作成]** を選択します。
1. **[Azure Compute Gallery の作成]** ページの **[基本]** タブで、以下の設定を指定して、**[Next: Sharing method] (次へ: 共有方法)** を選択します。

    |設定|値|
    |---|---|
    |サブスクリプション|このラボで使用している Azure サブスクリプションの名前|
    |リソース グループ|**az140-15b-RG**|
    |Name|**az14015computegallery**|
    |リージョン|Azure Virtual Desktop 環境をデプロイする Azure リージョンの名前|

1. **[Azure コンピューティング ギャラリーの作成]** ページの **[共有]** タブで、既定のオプション **[ロールベースのアクセス制御 (RBAC)]** を選択したまま、**[確認および作成]** を選択します。
1. **[確認および作成]** タブで、 **[作成]** を選択します。

    >**注**:プロビジョニング プロセスが完了するまで待ちます。 これにかかる時間は 1 分未満です。

1. ラボ コンピューターから Azure portal が表示されている Web ブラウザーで、**[Azure コンピューティング ギャラリー]** を検索して選択し、**[Azure コンピューティング ギャラリー]** ページで **[az14015computegallery]** を選択します。 
1. **[az14015computegallery]** ページで **[+ 追加]** を選択し、ドロップダウン メニューで **[+ VM イメージ定義]** を選択します。 
1. **[VM イメージ定義の作成]** ページの **[基本]** タブで、以下の設定を指定して (その他の設定はすべて既定値のままにします)、**[次へ: バージョン]** を選択します。

    |設定|Value|
    |---|---|
    |リージョン|Azure Virtual Desktop 環境をデプロイする Azure リージョンの名前|
    |VM イメージ定義名|**az14015imagedefinition**|
    |OS の種類|**Windows**|
    |セキュリティの種類|**トラステッド起動がサポートされています**|
    |OS の状態|**一般化されたイメージ**|
    |発行元|**MicrosoftWindowsDesktop**|
    |プラン|**Windows-11**|
    |SKU|**win11-23h2-avd-m365**|

    > **注**: 第 1 世代仮想マシンは信頼済みおよび機密のセキュリティの種類ではサポートされていないため、VM の世代は自動的に第 2 世代に設定されます。

1. **[VM イメージ定義の作成]** ページの **[バージョン]** タブで、設定を変更せずに、**[次へ: 発行オプション]** を選択します。

    > **注**: この段階では、VM イメージ バージョンを作成しないでください。 これは、Azure Virtual Desktop によって行われます。

1. **[VM イメージ定義の作成]** ページの **[発行オプション]** タブで、設定を変更せずに、**[確認および作成]** を選択します。
1. **[VM イメージ定義の作成]** ページの **[確認および作成]** タブで、**[作成]** を選択します。

    > **注**:プロビジョニング プロセスが完了するまで待ちます。 通常、所要時間は 1 分以下です。

#### タスク 6: カスタム イメージ テンプレートを作成する

1. ラボ コンピューターから Azure portal が表示されている Web ブラウザーで **[Azure Virtual Desktop]** を検索して選択し、**[Azure Virtual Desktop]** ページにある垂直ナビゲーション メニューの **[管理]** セクションで **[カスタム イメージ テンプレート]** を選択し、**[Azure Virtual Desktop \| カスタム イメージ テンプレート]** ページで **[+ カスタム イメージ テンプレートの追加]** を選択します。 
1. **[カスタム イメージ テンプレートの作成]** ページの **[基本]** タブで、以下の設定を指定し、**[次へ]** を選択します。

    > **注**: **[マネージド ID]** プロパティを設定するときは、*ランダム* プレースホルダーを、この演習で前に識別したのと同じ文字列に置き換えてください。

    |設定|Value|
    |---|---|
    |テンプレート名|**az140-15b-imagetemplate**|
    |既存のテンプレートからのインポート|**いいえ**|
    |サブスクリプション|このラボで使用している Azure サブスクリプションの名前|
    |リソース グループ|**az140-15b-RG**|
    |Location|Azure Virtual Desktop 環境をデプロイする Azure リージョンの名前|
    |マネージド ID|**az140**-*random*-**uami**|

1. **[カスタム イメージ テンプレートの作成]** ページの **[ソース イメージ]** タブで次の設定を指定し、**[次へ]** を選択します。

    |設定|値|
    |---|---|
    |送信元の種類|**プラットフォーム イメージ (マーケットプレース)**|
    |イメージの選択|**Windows 11 Enterprise マルチセッション、バージョン 23H2 + Microsoft 365 Apps**|

1. **[カスタム イメージ テンプレートの作成]** ページの **[配布ターゲット]** タブで、以下の設定を指定して (その他の設定は既定値のままにします)、**[次へ]** を選択します。

    |設定|Value|
    |---|---|
    |Azure Compute Gallery|有効|
    |ギャラリー名|**az14015computegallery**|
    |ギャラリー イメージの定義|**az14015imagedefinition**|
    |ギャラリー イメージ バージョン|**1.0.0**|
    |実行の出力名|**az140-15-image-1.0.0**|
    |レプリケーションのリージョン|Azure Virtual Desktop 環境をデプロイする Azure リージョンの名前|
    |最新から除外|**いいえ**|
    |ストレージ アカウントの種類|**Standard_LRS**|

    > **注**: **[レプリケーションのリージョン]** プロパティを使用して、複数リージョンのビルドに対応できます。 **[最新から除外]** を **[はい]** に設定すると、VM の作成中に **ImageReference** 要素のバージョンとして**最新**が指定されたときに、このイメージ バージョンが使用されなくなります。

1. **[カスタム イメージ テンプレートの作成]** ページの **[ビルド プロパティ]** タブで、次の設定を指定し (他の設定は既定値のまま)、**[次へ]** を選択します。

    |設定|Value|
    |---|---|
    |ビルド タイムアウト|**120**|
    |ビルド VM サイズ|**Standard_DC2s_v3**|
    |OS ディスク サイズ (GB)|**127**|
    |ステージング グループ|**az140-15c-RG**|
    |VNet|設定しないままにします|

    > **注**: **ステージング グループ**は、イメージをビルドしてログを格納するためにリソースをステージングするのに使用されるリソース グループです。 指定しない場合、名前は自動的に生成されます。 **VNet** 名が設定されていない場合は、一時的な名前と、ビルドの作成に使用される VM のパブリック IP アドレスが作成されます。

    > **重要**: 指定したビルド VM サイズに対して十分な数の使用可能な vCPU があることを確認します。 そうでない場合は、別のサイズを選択するか、クォータの引き上げを要求します。

1. **[カスタム イメージ テンプレートの作成]** ページの **[カスタマイズ]** タブで、**[+ 組み込みスクリプトの追加]** を選択します。 
1. **[組み込みスクリプトの選択]** ペインで、オペレーティング システム固有のスクリプト、Azure Virtual Desktop スクリプト、MSIX アプリ アタッチ スクリプト、アプリケーション スクリプト、Windows 更新プログラム関連スクリプトにグループ化された使用可能なオプションを確認し、次のエントリを選択します。

   - **タイム ゾーン リダイレクト**: クライアントがセッション ホスト上のセッション内でそのタイム ゾーンを使用できるようにします
   - **ストレージ センサーを無効にする**: 空きディスク領域の不足状態を誤って検出することで、ストレージ センサーがセッション ホストに悪影響を与えないようにします
   - **クライアントとサーバー上での画面キャプチャのブロック**を使用して**画面キャプチャ保護を有効にする**: スクリーンショットと画面共有でリモート コンテンツをブロックまたは非表示にします

1. **[組み込みスクリプトの選択]** ペインで、**[保存]** を選択します。

    > **注**: 独自のスクリプトを追加することもできます。 たとえば、[[タイム ゾーン リダイレクト]](https://raw.githubusercontent.com/Azure/RDS-Templates/master/CustomImageTemplateScripts/CustomImageTemplateScripts_2024-03-27/TimezoneRedirection.ps1)、[[ストレージ センサーを無効にする]](https://raw.githubusercontent.com/Azure/RDS-Templates/master/CustomImageTemplateScripts/CustomImageTemplateScripts_2024-03-27/DisableStorageSense.ps1)、[[画面キャプチャ保護を有効にする]](https://raw.githubusercontent.com/Azure/RDS-Templates/master/CustomImageTemplateScripts/CustomImageTemplateScripts_2024-03-27/ScreenCaptureProtection.ps1) など、組み込みのスクリプトを参照することを検討してください。

1. **[カスタム イメージ テンプレートの作成]** ページの **[カスタマイズ]** タブで、**[次へ]** を選択します。
1. **[カスタム イメージ テンプレートの作成]** ページの **[タグ]** タブで、**[次へ]** を選択します。
1. **[カスタム イメージ テンプレートの作成]** ページの **[確認および作成]** タブで、**[作成]** を選択します。

    > **注**: テンプレート VM が作成されるまで待ちます。 この処理には数分かかる場合があります。 **[Azure Virtual Desktop \| カスタム イメージ テンプレート]** ページを更新して、テンプレートの状態を確認します。

#### タスク 7: カスタム イメージをビルドする

> **注**: 待機時間がかなり長いため、このラボの残りのタスクは省略可能です。 

1. ラボ コンピューターから Azure portal が表示されている Web ブラウザーで、**[Azure Virtual Desktop \| カスタム イメージ テンプレート]** ページの **[az140-15b-imagetemplate]** を選択します。
1. **az140-15b-imagetemplate** ページで、**[ビルドの開始]** を選択します。

    > **注**: ビルド が作成されるまで待ちます。 ビルド プロセスを完了する実際の時間は異なる場合がありますが、ラボの手順で提供されている設定では、45 分以内に完了する必要があります。 ページを数分ごとに更新し、**az140-15b-imagetemplate** ページの **[Essentials]** セクションの **[ビルドの実行状態]** の値を監視します。 

    > **注**: ビルドの実行状態は、**[実行中] - [構築中]** から **[実行中] - [配布中]** に変更され、最後に **[成功]** に変わる必要があります。

    > **注**: ビルドの完了を待っている間に、ステージング リソース グループ **az140-15c-RG** の内容を確認します。このリソースには、ビルド仮想マシン、仮想ネットワーク、ネットワーク セキュリティ グループ、キー コンテナー、スナップショット、コンテナー インスタンス、ストレージ アカウントなどのビルド リソースが自動的にプロビジョニングされます。 

1. ラボ コンピューターから Azure portal が表示されている Web ブラウザーで、**[リソース グループ]** を検索して選択し、**[リソース グループ]** ページで **[az140-15c-RG]** を選択します。
1. **[az140-15c-RG]** ページの **[リソース]** セクションで、自動プロビジョニングされたリソースをメモします。
1. **az140-15b-imagetemplate** ページに戻り、ビルドの進行状況を監視します。 

    > **注**: または、**アクティビティ ログ**を使って、ビルド プロセスの完了を追跡することもできます。 注目すべきアクションは、**VM イメージ テンプレートを実行して出力を生成する**ことです。 その状態は、ある時点で **[承認済み]** から **[成功]** に変更されます。

1. ビルドが完了したら、ラボ コンピューターから Azure portal が表示されている Web ブラウザーで、**[Azure コンピューティング ギャラリー]** を検索して選択し、**[Azure コンピューティング ギャラリー]** ページで **[az14015computegallery]** を選択します。 
1. **[az14015computegallery]** の **[定義]** タブで、**[az14015imagedefinition]** を選択します。
1. **az14015imagedefinition** ページの **[バージョン]** タブで、**1.0.0 (最新バージョン)** のイメージに関する情報を確認します。

#### タスク 8: カスタム イメージを使用してセッション ホストをデプロイする

> **注**: 必要に応じて、作成したカスタム イメージを使用して、Azure Virtual Desktop セッション ホストのデプロイの初期段階をステップ実行することを検討してください。 

1. ラボ コンピューターから Azure portal が表示されている Web ブラウザーで、**[仮想ネットワーク]** を検索して選択し、**[仮想ネットワーク]** ページで **[作成 +]** を選択します。
1. **[仮想ネットワークの作成]** ページの **[基本]** タブで、以下の設定を指定し、 **[次へ]** を選択します。

    |設定|値|
    |---|---|
    |サブスクリプション|このラボで使用している Azure サブスクリプションの名前|
    |リソース グループ|新しいリソース グループ **az140-15d-RG** の名前|
    |仮想ネットワーク名|**az140-vnet15d**|
    |リージョン|Azure Virtual Desktop 環境をデプロイする Azure リージョンの名前|

1. **[セキュリティ]** タブで、既定の設定をそのまま使用し、 **[次へ]** を選択します。
1. **[IP アドレス]** タブで、以下の設定を指定します。

    |設定|値|
    |---|---|
    |IP アドレス空間|**10.30.0.0/16**|

1. **デフォルト** サブネット エントリの横にある編集 (鉛筆) アイコンを選択し、 **Edit** ペインで次の設定を指定し (他の設定は既存の値のまま)、**保存**を選択します。

    |設定|値|
    |---|---|
    |名前|**hp1-Subnet**|
    |開始アドレス|**10.30.1.0**|
    |プライベート サブネットを有効にする (デフォルトの送信アクセスなし)|無効|

1. **[IP アドレス]** タブに戻り、**[確認および作成]** を選択し、**[作成]** を選択します。

    > **注**:プロビジョニング プロセスが完了するまで待ちます。 通常、所要時間は 1 分以下です。

1. ラボ コンピューターから Azure portal が表示されている Web ブラウザーで **[Azure Virtual Desktop]** を検索して選択し、**[Azure Virtual Desktop]** ページにある垂直ナビゲーション メニューの **[管理]** セクションで **[ホスト プール]** を選択し、**[Azure Virtual Desktop \| ホスト プール]** ページで **[+ 作成]** を選択します。 
1. **[ホスト プールの作成]** ページの **[基本]** タブで、次の設定を指定し、**[次へ: セッション ホスト]** を選択します (他は既定値のままにします)。

    |設定|値|
    |---|---|
    |サブスクリプション|このラボで使用している Azure サブスクリプションの名前|
    |リソース グループ|**az140-15d-RG**|
    |ホスト プール名|**az140-15-hp1**|
    |Location|Azure Virtual Desktop 環境をデプロイする Azure リージョンの名前|
    |検証環境|**いいえ**|
    |優先するアプリ グループの種類|**デスクトップ**|
    |ホスト プールの種類|**プールされた**|
    |セッション ホスト構成を作成する|**いいえ**|
    |負荷分散アルゴリズム|**幅優先**|

    > **注**: 幅優先の負荷分散アルゴリズムを使用する場合、最大セッション制限パラメーターは省略可能です。

1. **[ホスト プールの作成]** ページの **[セッション ホスト]** タブで、以下の設定を指定します (他の設定は既定値のままにします)。

    > **注**: **名前のプレフィックス**値を設定するときは、ラボ セッション ウィンドウの右側にある [リソース] タブに切り替え、*User1-* と *@* 文字の間の文字の文字列を識別します。 この文字列を使用して、*ランダム* プレースホルダーを置き換えます。

    |設定|Value|
    |---|---|
    |[仮想マシンの追加]|**はい**|
    |リソース グループ|**既定はホスト プールと同じ**|
    |名前のプレフィックス|**sh0**_random_|
    |仮想マシンのタイプ|**Azure 仮想マシン**|
    |仮想マシンの場所|Azure Virtual Desktop 環境をデプロイする Azure リージョンの名前|
    |可用性のオプション|**インフラストラクチャ冗長は必要ありません**|
    |セキュリティの種類|**トラステッド起動の仮想マシン**|

1. **[ホスト プールの作成]** ページの **[仮想マシン]** タブで、**[イメージ]** ドロップダウン リストの下にある **[すべてのイメージを表示]** を選択します。
1. **[イメージの選択]** ページで **[共有イメージ]** を選択し、イメージの一覧で **az14015imagedefinition** を選択します。 
1. **[ホスト プールの作成]** ページの **[仮想マシン]** タブに戻り、以下の設定を指定して、**[次へ: ワークスペース >]** を選択します (他の設定は既定値のままにします)。

    |設定|Value|
    |---|---|
    |[仮想マシンのサイズ]|**Standard DC2s_v3**|
    |[Number of VMs](VM の数)|**1**|
    |OS ディスクの種類|**Standard SSD**|
    |OS ディスク サイズ|**既定サイズ**|
    |ブート診断|**マネージド ストレージ アカウントで有効にする (推奨)**|
    |仮想ネットワーク|az140-vnet15d|
    |Subnet|**hp1-Subnet**|
    |ネットワーク セキュリティ グループ|**Basic**|
    |パブリック インバウンド ポート|**いいえ**|
    |参加したいディレクトリを選択する|**Microsoft Entra ID**|
    |Intune に VM を登録する|**いいえ**|
    |ユーザー名|**Student**|
    |Password|あらかじめ登録された Administrator アカウントのパスワードとして使用される任意の十分に複雑な文字列|
    |[パスワードの確認入力]|前に指定した文字列と同じ文字列|

    > **注**: パスワードの長さは 12 文字以上で、小文字、大文字、数字、特殊文字の組み合わせで構成されている必要があります。 詳細については、[Azure VM の作成時のパスワード要件](https://learn.microsoft.com/en-us/azure/virtual-machines/windows/faq#what-are-the-password-requirements-when-creating-a-vm-)に関する情報をご覧ください。

1. **[ホスト プールの作成]** ページの **[ワークスペース]** タブで、次の設定を確認し、**[確認および作成]** を選択します。

    |設定|Value|
    |---|---|
    |デスクトップ アプリ グループを登録する|**いいえ**|

1. **[ホストプールの作成]** ページの **[確認および作成]** タブで、**[作成]** を選択します。

    > **注**: デプロイが完了するまで待ちます。 これには 10 から 15 分ほどかかる場合があります。
