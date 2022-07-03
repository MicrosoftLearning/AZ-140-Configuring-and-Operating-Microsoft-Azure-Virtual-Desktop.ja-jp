---
lab:
  title: ラボ:Azure Virtual Desktop プロファイル (AD DS) を実装および管理する
  module: 'Module 4: Manage User Environments and Apps'
ms.openlocfilehash: 74bfd6ca118377b66f1a59947890efb821a43428
ms.sourcegitcommit: 0cbc3a5f6fd902e2253dae2575aff75bbc2fa99c
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/25/2022
ms.locfileid: "145883970"
---
# <a name="lab---implement-and-manage-azure-virtual-desktop-profiles-ad-ds"></a>ラボ - Azure Virtual Desktop プロファイル (AD DS) を実装および管理する
# <a name="student-lab-manual"></a>受講生用ラボ マニュアル

## <a name="lab-dependencies"></a>ラボの依存関係

- このラボで使用する Azure サブスクリプション。
- このラボで使用する Azure サブスクリプション内で所有者または共同作成者のロールを持つ Microsoft アカウントまたは Azure AD アカウント、この Azure サブスクリプションに関連付けられた Azure AD テナント内でグローバル管理者ロールを持つMicrosoft アカウントまたは Azure AD アカウント。
- 実施するラボ - **Azure Virtual Desktop (AD DS) のデプロイを準備する**
- 実施するラボ - **WVD (AD DS) 用のストレージを実装および管理する**

## <a name="estimated-time"></a>推定所要時間

30 分

## <a name="lab-scenario"></a>ラボのシナリオ

Active Directory ドメイン サービス (AD DS) 環境に Azure Virtual Desktop プロファイル管理を実装する必要があります。

## <a name="objectives"></a>目標
  
このラボを完了すると、次のことができるようになります。

- Azure Virtual Desktop 用に FSLogix ベースのプロファイルを実装する

## <a name="lab-files"></a>ラボ ファイル 

- なし

## <a name="instructions"></a>手順

### <a name="exercise-1-implement-fslogix-based-profiles-for-azure-virtual-desktop"></a>演習 1:Azure Virtual Desktop の FSLogix ベース プロファイルを実装する

この演習の主なタスクは次のとおりです。

1. Azure Virtual Desktop セッション ホスト VM で FSLogix ベースのプロファイルを構成する
1. Azure Virtual Desktop を使用して FSLogix ベースのプロファイルをテストする
1. ラボでデプロイされている Azure リソースを削除する

#### <a name="task-1-configure-fslogix-based-profiles-on-azure-virtual-desktop-session-host-vms"></a>タスク 1:Azure Virtual Desktop セッション ホスト VM で FSLogix ベースのプロファイルを構成する

1. ラボのコンピューターから Web ブラウザーを起動し、[Azure portal](https://portal.azure.com) に移動し、このラボで使用するサブスクリプションの所有者の役割を持つユーザーアカウントの認証情報を提供してサインインします。
1. Azure portal で、「**仮想マシン**」を検索して選択し、 **[仮想マシン]** ブレードで **[az140-21-p1-0]** を選択します。
1. **[az140-21-p1-0]** ブレードで、 **[開始]** を選択し、仮想マシンの状態が **[実行中]** に変わるまで待ちます。
1. **[az140-21-p1-0]** ブレードで **[接続]** を選択し、ドロップダウン メニューで **[Bastion]** を選択し、 **[az140-21-p1-0 \| 接続]** ブレードの **[Bastion]** タブで **[Bastion を使用する]** を選択します。
1. プロンプトが表示されたら、次の認証情報を入力します。

   |設定|値|
   |---|---|
   |[ユーザー名]|**student@adatum.com**|
   |パスワード|**Pa55w.rd1234**|

1. **az140-21-p1-0** へのリモート デスクトップ セッション内で、Microsoft Edge を起動して、[Azure portal](https://portal.azure.com) に移動します。 プロンプトが表示されたら、このラボで使用しているサブスクリプションで所有者の役割を持つユーザーアカウントの資格情報を使用してサインインします。
1. **az140-21-p1-0** へのリモート デスクトップ セッション内で、Azure portal を表示している Microsoft Edge の画面の [Cloud Shell] ペイン内にある PowerShell セッションを起動します。 
1. Cloud Shell ペインの PowerShell セッションから、以下を実行して、このラボで使用する Azure VM をホストする Azure Virtual Desktop セッションを開始します。

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Start-AzVM
   ```

   >**注**:Azure VM が実行中になるのを待ってから、次の手順に進みます。

1. **az140-21-p1-0** へのリモート デスクトップ セッション内で、Microsoft Edge を起動し、[FSLogix ダウンロード ページ](https://aka.ms/fslogix_download)を参照し、FSLogix 圧縮インストール バイナリをダウンロードして、これを **C:\\Allfiles\\Labs\\04** フォルダー (必要に応じて、フォルダーを作成する) に抽出し、**x64\\Release** サブフォルダーに移動して、**FSLogixAppsSetup.exe** ファイルをダブルクリックし、**Microsoft FSLogix Apps セットアップ** ウィザードを実行して、既定の設定で Microsoft FSLogix Apps をインストールします。

   > **注**:イメージがすでに含まれている場合、FXLogic のインストールは不要です。

3. **az140-21-p1-0** へのリモート デスクトップ セッション内で、管理者として **Windows PowerShell ISE** を起動し、 **[管理者: Windows PowerShell ISE]** スクリプト ペインで、次のように実行して、PowerShellGet モジュールの最新バージョンをインストールします (確認を求められたら **[はい]** を選択します)。

   ```powershell
   [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
   Install-Module -Name PowerShellGet -Force -SkipPublisherCheck
   ```

1. **[管理者: Windows PowerShell ISE]** コンソールで、次のように実行して、最新バージョンの Az PowerShell モジュールをインストールします (確認を求められたら、 **[すべてはい]** を選択します)。

   ```powershell
   Install-Module -Name Az -AllowClobber -SkipPublisherCheck
   ```

1. **[管理者: Windows PowerShell ISE]** コンソールで、次のように実行して、実行ポリシーを変更します。

   ```powershell
   Set-ExecutionPolicy RemoteSigned -Force
   ```

1. **[管理者: Windows PowerShell ISE]** コンソールで、次のように実行して、Azure サブスクリプションにサインインします。

   ```powershell
   Connect-AzAccount
   ```

1. プロンプトが表示されたら、このラボで使用しているサブスクリプションで所有者の役割を持つユーザーアカウントの資格情報を使用してサインインします。
1. **az140-21-p1-0** へのリモート デスクトップ セッション内で、 **[管理者: Windows PowerShell ISE]** スクリプト ペインで、次のように実行して、このラボで以前に構成した AzureStorage アカウントの名前を取得します。

   ```powershell
   $resourceGroupName = 'az140-22-RG'
   $storageAccountName = (Get-AzStorageAccount -ResourceGroupName $resourceGroupName)[0].StorageAccountName   
   ```

1. **az140-21-p1-0** へのリモート デスクトップ セッション内で、 **[管理者: Windows PowerShell ISE]** スクリプト ペインで、次のように実行して、プロファイル レジストリ設定を構成します。

   ```powershell
   $profilesParentKey = 'HKLM:\SOFTWARE\FSLogix'
   $profilesChildKey = 'Profiles'
   $fileShareName = 'az140-22-profiles'
   New-Item -Path $profilesParentKey -Name $profilesChildKey –Force
   New-ItemProperty -Path $profilesParentKey\$profilesChildKey -Name 'Enabled' -PropertyType DWord -Value 1
   New-ItemProperty -Path $profilesParentKey\$profilesChildKey -Name 'VHDLocations' -PropertyType MultiString -Value "\\$storageAccountName.file.core.windows.net\$fileShareName"
   ```

1. **az140-21-p1** へのリモート デスクトップ セッション内で、 **[スタート]** を右クリックし、右クリック メニューで **[ファイル名を指定して実行]** を選択し、 **[ファイル名を指定して実行]** ダイアログ ボックスの **[開く]** テキストボックスに次のように入力し、 **[OK]** を選択して **[ローカル ユーザーとグループ]** コンソールを起動します。

   ```cmd
   lusrmgr.msc
   ```

1. **ローカル ユーザーとグループ** コンソールで、名前が **FSLogix** 文字列で始まる 4 つのグループに注意してください。

   - FSLogix ODFC 除外リスト
   - FSLogix ODFC 包含リスト
   - FSLogix プロファイル除外リスト
   - FSLogix プロファイル包含リスト

1. **[ローカル ユーザーとグループ]** コンソールのグループのリストで、 **[FSLogix プロファイ包含リスト]** グループをダブルクリックし、 **\\Everyone** グループが含まれていることを確認し、 **[OK]** を選択して、グループの **[プロパティ]** ウィンドウを閉じます。 
1. **ローカル ユーザーとグループ** コンソールのグループのリストで、**FSLogix プロファイル除外リスト** グループをダブルクリックし、既定ではグループ メンバーが含まれていないことに注意し、 **[OK]** を選択してグループの **[プロパティ]** ウィンドウを閉じます。 

   > **注**:一貫したユーザー エクスペリエンスを提供するには、すべての Azure Virtual Desktop セッション ホストに FSLogix コンポーネントをインストールして構成する必要があります。 このタスクは、ラボ環境の他のセッション ホストで無人で実行します。 

1. **az140-21-p1-0** へのリモート デスクトップ セッション内で、 **[管理者: Windows PowerShell ISE]** スクリプト ペインで、次のように実行して、FSLogix コンポーネントを **az140-21-p1-1** および **az140-21-p1-2** セッション ホストにインストールします。

   ```powershell
   $servers = 'az140-21-p1-1', 'az140-21-p1-2'
   foreach ($server in $servers) {
      $localPath = 'C:\Allfiles\Labs\04\x64'
      $remotePath = "\\$server\C$\Allfiles\Labs\04\x64\Release"
      Copy-Item -Path $localPath\Release -Destination $remotePath -Filter '*.exe' -Force -Recurse
      Invoke-Command -ComputerName $server -ScriptBlock {
         Start-Process -FilePath $using:localPath\Release\FSLogixAppsSetup.exe -ArgumentList '/quiet' -Wait
      } 
   }
   ```

   > **注**:スクリプトの実行が完了するのを待ちます。 これには 2 分ほどかかる場合があります。

1. **az140-21-p1-0** へのリモート デスクトップ セッション内で、 **[管理者: Windows PowerShell ISE]** スクリプト ペインで、次のように実行して、**az140-21-p1-1** および **az140-21-p1-1** セッション ホストのプロファイル レジストリ設定を構成します。

   ```powershell
   $profilesParentKey = 'HKLM:\SOFTWARE\FSLogix'
   $profilesChildKey = 'Profiles'
   $fileShareName = 'az140-22-profiles'
   foreach ($server in $servers) {
      Invoke-Command -ComputerName $server -ScriptBlock {
         New-Item -Path $using:profilesParentKey -Name $using:profilesChildKey –Force
         New-ItemProperty -Path $using:profilesParentKey\$using:profilesChildKey -Name 'Enabled' -PropertyType DWord -Value 1
         New-ItemProperty -Path $using:profilesParentKey\$using:profilesChildKey -Name 'VHDLocations' -PropertyType MultiString -Value "\\$using:storageAccountName.file.core.windows.net\$using:fileShareName"
      }
   }
   ```

   > **注**:FSLogix ベースのプロファイル機能をテストする前に、テストに使用する **ADATUM\\aduser1** アカウントの、ローカルにキャッシュされたプロファイルを、前のラボで使用した Azure Virtual Desktop セッション ホストから削除する必要があります。

1. **az140-21-p1-0** へのリモート デスクトップ セッション内で、 **[管理者: Windows PowerShell ISE]** スクリプト ペインで、次のように実行して、セッションホストとして機能するすべての Azure VM でローカルにキャッシュされた **ADATUM\\aduser1** アカウントのプロファイルを削除します。

   ```powershell
   $userName = 'aduser1'
   $servers = 'az140-21-p1-0','az140-21-p1-1', 'az140-21-p1-2'
   Get-CimInstance -ComputerName $servers -Class Win32_UserProfile | Where-Object { $_.LocalPath.split('\')[-1] -eq $userName } | Remove-CimInstance
   ```

#### <a name="task-2-test-fslogix-based-profiles-with-azure-virtual-desktop"></a>タスク 2:Azure Virtual Desktop を使用して FSLogix ベースのプロファイルをテストする

1. ラボ コンピューターから、Azure portal を表示している Web ブラウザーの画面を表示しているラボ コンピューターに切り替え、「**仮想マシン**」を検索して選択し、 **[仮想ネットワーク]** ブレードから **[az140-cl-vm11]** エントリを選択します。
1. **[az140-cl-vm11]** ブレードで **[接続]** を選択し、ドロップダウン メニューで **[Bastion]** を選択し、 **[az140-cl-vm11 \| 接続]** ブレードの **[Bastion]** タブで **[Bastion を使用する]** を選択します。
1. プロンプトが表示されたら、次の資格情報を入力し、 **[接続]** を選択します。

   |設定|値|
   |---|---|
   |[ユーザー名]|**Student@adatum.com**|
   |パスワード|**Pa55w.rd1234**|

1. **az140-cl-vm11** へのリモート デスクトップ セッション内で、 **[スタート]** をクリックし、 **[スタート]** メニューで、 **[リモート デスクトップ]** をクリックして、リモート デスクトップ クライアントを起動します。
1. **az140-cl-vm11** へのリモート デスクトップ セッション内で、 **[リモート デスクトップ]** クライアント ウィンドウの **[サブスクライブ]** を選択し、**aduser1** の資格情報を使用してサインインします。

 >**注** サブスクライブを求めらない場合は、以前のサブスクリプションの登録の解除が必要な場合があります。
3. アプリケーションの一覧で、 **[コマンド プロンプト]** をダブルクリックし、プロンプトが表示されたら、**aduser1** アカウントのパスワードを入力し、 **[コマンド プロンプト]** ウィンドウが正常に開くことを確認します。
4. **[コマンド プロンプト]** ウィンドウの左上隅にある **[コマンド プロンプト]** アイコンを右クリックし、ドロップダウン メニューで **[プロパティ]** を選択します。
5. **[コマンド プロンプトのプロパティ]** ダイアログ ボックスで、 **[フォント]** タブを選択し、サイズとフォント設定を変更して、 **[OK]** を選択します。
6. **[コマンド プロンプト]** ウィンドウで、**logoff** と入力し、**Enter** キーを押して、リモート デスクトップ セッションからサインアウトします。
7. **az140-cl-vm11** へのリモート デスクトップ セッション内で、 **[リモート デスクトップ]** クライアント ウィンドウのアプリケーションの一覧で、[az140-21-ws1] の下にある **[SessionDesktop]** をダブルクリックし、リモート デスクトップ セッションが起動することを確認します。 
8. **SessionDesktop** セッション内で、 **[スタート]** を右クリックし、右クリック メニューで **[ファイル名を指定して実行]** を選択し、 **[ファイル名を指定して実行]** ダイアログ ボックスの **[開く]** テキストボックスに **cmd** と入力し、 **[OK]** を選択して **コマンド プロンプト** ウィンドウを起動します。
9. **コマンド プロンプト** ウィンドウの設定が、このタスクの前半で構成した設定と一致していることを確認します。
10. **SessionDesktop** セッション内で、すべてのウィンドウを最小化し、デスクトップを右クリックし、右クリック メニューで **[新規]** を選択し、カスケード メニューで **[ショートカット]** を選択します。 
11. **[ショートカットの作成]** ウィザードの **[どの項目にショートカットを作成しますか?]** ページで、 **[項目の場所を入力してください]** テキスト ボックスに「**メモ帳**」と入力し、 **[次へ]** を選択します。
12. **[ショートカットの作成]** ウィザードの **[ショートカットの名前を付ける]** ページで、 **[このショートカットの名前を入力してください]** テキスト ボックスに「**メモ帳**」と入力し、 **[完了]** を選択します。
13. **SessionDesktop** セッション内で、 **[スタート]** を右クリックし、右クリック メニューで **[シャットダウン] または [サインアウト]** を選択してから、カスケード メニューで **[サインアウト]** を選択します。
14. **az140-cl-vm11** へのリモート デスクトップ セッションに戻り、 **[リモート デスクトップ]** クライアント ウィンドウのアプリケーションの一覧で、 **[SessionDesktop]** をダブルクリックし、新しいリモート デスクトップ セッションを開始します。 
15. **SessionDesktop** セッション内で、**メモ帳** のショートカットがデスクトップに表示されることを確認します。
16. **SessionDesktop** セッション内で、 **[スタート]** を右クリックし、右クリック メニューで **[シャットダウン] または [サインアウト]** を選択してから、カスケード メニューで **[サインアウト]** を選択します。
17. ラボ コンピューターに切り替え、Azure portal を表示している Microsoft Edge ウィンドウで、 **[ストレージ アカウント]** ブレードに移動し、前の演習で作成したストレージ アカウントを表すエントリを選択します。
18. ストレージ アカウント ブレードの **[ファイル サービス]** セクションで **[ファイル共有]** を選択し、ファイル共有のリストで **[az140-22-profiles]** を選択します。 
19. **[az140-22-profiles]** ブレードで、そのコンテンツに、**ADATUM\\aduser1** アカウントのセキュリティ識別子 (SID) と、それに続く **_aduser1** サフィックスの組み合わせで構成される名前のフォルダーが含まれていることを確認します。
20. 前の手順で特定したフォルダーを選択し、 **"Profile_aduser1.vhd"** という名前の 1 つのファイルが含まれていることに注意してください。

### <a name="exercise-2-stop-and-deallocate-azure-vms-provisioned-and-used-in-the-lab"></a>演習 2:ラボでプロビジョニングおよび使用された Azure VM を停止および割り当て解除する

この演習の主なタスクは次のとおりです。

1. ラボでプロビジョニングおよび使用された Azure VM を停止および割り当て解除する

>**注**:この演習では、このラボでプロビジョニングおよび使用した Azure VM を割り当て解除し、対応するコンピューティング料金を最小化します

#### <a name="task-1-deallocate-azure-vms-provisioned-and-used-in-the-lab"></a>タスク 1:ラボでプロビジョニングおよび使用された Azure VM を割り当て解除する

1. ラボ コンピューターに切り替え、Azure portal を表示している Web ブラウザーの画面で、**Cloud Shell** ペイン内の **PowerShell** シェル セッションを開きます。
1. Cloud Shell ペインの PowerShell セッションから、以下を実行して、このラボで作成および使用されたすべての Azure VM を一覧表示します。

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG'
   ```

1. Cloud Shell ペインの PowerShell セッションから、以下を実行して、このラボで作成および使用されたすべての Azure VM を停止して割り当てを解除します。

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Stop-AzVM -NoWait -Force
   ```

   >**注**:コマンドは非同期で実行されるので (-NoWait パラメーターによって決定されます)、同じ PowerShell セッション内で直後に別の PowerShell コマンドを実行できますが、Azure VM が実際に停止されて割り当てが解除されるまでに数分かかります。
