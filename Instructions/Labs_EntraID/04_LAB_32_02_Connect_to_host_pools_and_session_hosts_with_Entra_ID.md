---
lab:
  title: 'ラボ: セッション ホストに接続する (Entra ID)'
  module: 'Module 3.2: Plan and implement user experience and client settings'
---

# ラボ - セッション ホストに接続する (Entra ID)
# 受講生用ラボ マニュアル

## ラボの依存関係

- このラボで使用する Azure サブスクリプション。
- このラボで使用する Azure サブスクリプションの所有者または投稿者ロールを持ち、その Azure サブスクリプションに関連付けられた Entra テナントにデバイスを参加させるのに十分なアクセス許可を持つ Microsoft Entra ユーザー アカウント。
- ラボ *Azure portal を使用してホスト プールとセッション ホストをデプロイする (Entra ID)* 完了
- ラボ *Azure portal を使用してホスト プールとセッション ホストを管理する (Entra ID)* 完了

## 推定所要時間

20 分

## ラボのシナリオ

Entra 参加済みセッション ホストを含む Azure Virtual Desktop 環境が既にあります。 Microsoft Entra に参加または登録されていない Windows 11 クライアントからそれらに接続することにより、それらの機能を検証する必要があります。

## 目標
  
このラボを完了すると、次のことができるようになります。

- Microsoft Entra に参加または登録されていない Windows クライアントから接続することにより、Microsoft Entra 参加済み Azure Virtual Desktop セッション ホストの機能を検証します。

## ラボ ファイル

- なし

## 手順

### 演習 1: Microsoft Entra 参加済みの Azure Virtual Desktop セッション ホストの機能を Windows 11 クライアントから接続することにより検証する
  
この演習の主なタスクは次のとおりです。

1. Azure Virtual Desktop ホスト プールの RDP プロパティを調整する
1. Windows 11 コンピューターに Microsoft リモート デスクトップ クライアントをインストールする
1. Azure Virtual Desktop ワークスペースに登録する
1. Azure Virtual Desktop アプリのテスト


#### タスク 1: Azure Virtual Desktop ホスト プールの RDP プロパティを調整する

> **注**: 前のラボで実装した RDP 設定は、(シングル サインオンのサポートを介して) 最適なユーザー エクスペリエンスを提供しますが、これには「[　Microsoft Entra ID 認証を使用した Azure Virtual Desktop のシングル サインオンの構成](https://learn.microsoft.com/en-us/azure/virtual-desktop/configure-single-sign-on)」で説明されている追加の変更が必要です。 これらの変更がない場合、既定では、クライアント コンピューターが次のいずれかの条件を満たす場合に認証がサポートされます。

- セッション ホストと同じ Microsoft Entra テナントに Microsoft Entra 参加済みである
- セッション ホストと同じ Microsoft Entra テナントに Microsoft Entra ハイブリッド参加済みである
- セッション ホストと同じ Microsoft Entra テナントに Microsoft Entra 登録済みである

これらの条件はいずれもラボ コンピューターには適用されないので、カスタム RDP プロパティとして `targetisaadjoined:i:1` をホスト プールに追加する必要があります。

1. 必要に応じて、ラボ コンピューターから Web ブラウザーを起動して Azure portal に移動し、このラボで使用するサブスクリプションの所有者ロールを持つユーザー アカウントの資格情報を入力してサインインします。

    > **注**: ラボ セッション ウィンドウの右側にある [リソース] タブに表示されている `User1-` アカウントの資格情報を使用します。

1. Azure portal が表示されている Web ブラウザーの **[az140-21-hp1]** ページの垂直メニュー バーの **[設定]** セクションで、**[RDP プロパティ]** エントリを選択します。
1. **[az140-21-hp1 \| RDP プロパティ]** ページで、**[詳細]** タブを選択します。 
1. **[az140-21-hp1 \| RDP プロパティ]** ページの **[詳細]** タブの **[RDP プロパティ]** テキスト ボックスで、次の文字列を既存のコンテンツに追加します (この文字列を前置き文字列から区切る必要がある場合は、先頭のセミコロン文字 (`;`) を追加してください)。

    ```txt
    targetisaadjoined:i:1
    ```

1. **[RDP プロパティ]** テキスト ボックスで、既存のコンテンツ (末尾のセミコロン文字を含む) から次の文字列 (存在する場合) を削除します。

    ```txt
    enablerdsaadauth:i:value
    ```

1. **[az140-21-hp1 \| RDP プロパティ]** ページで、**[保存]** を選択します。

#### タスク 2: Windows 11 コンピューターに Microsoft リモート デスクトップ クライアントをインストールする

1. ラボ コンピューターから Web ブラウザーを起動し、[[Windows 用リモート デスクトップ クライアントを使用して Azure Virtual Desktop に接続]](https://learn.microsoft.com/en-us/azure/virtual-desktop/users/connect-windows) ページに移動し、**[リモート デスクトップ クライアント (MSI) をダウンロードしてインストール]** セクションまで下にスクロールして、[Windows 64 ビット](https://go.microsoft.com/fwlink/?linkid=2139369) リンクを選択します。 
1. エクスプローラーを開き、**ダウンロード** フォルダーに移動し、新しくダウンロードした MSI ファイルのインストールを開始します。 
1. **リモート デスクトップ セットアップ** ウィンドウで、プロンプトが表示されたら、ライセンス契約の条項に同意し、**このマシンのすべてのユーザーに対してインストールする**オプションを選択します。 プロンプトが表示されたら、ユーザー アカウント制御プロンプトを受け入れてインストールを続行します。
1. インストールが完了したら、**[セットアップの終了時にリモート デスクトップを起動する]** チェックボックスがオンになっていることを確認し、**[完了]** を選択して Microsoft リモート デスクトップ クライアントを起動します。

   > **注**: Windows 用[リモート デスクトップ アプリ](https://learn.microsoft.com/en-us/azure/virtual-desktop/users/connect-windows?pivots=rd-store)は、Microsoft Entra 参加済みセッション ホストへの接続をサポートしません。

#### タスク 3: Azure Virtual Desktop ワークスペースに登録する

1. ラボ コンピューターで、**[リモート デスクトップ]** クライアント ウィンドウに切り替え、**[登録]** を選択し、メッセージが表示されたら、ラボ インターフェイス ウィンドウの右側のペインにある **[リソース]** タブにある `User1` Entra ID ユーザー アカウントの資格情報でサインインします。

   > **注**: **AVD-DAG** プレフィックスを持つ Entra グループのメンバーであるユーザー アカウントを選択します。

   > **注**: または、**[リモート デスクトップ]** クライアント ウィンドウで、**[URL で登録]** を選択し、**[ワークスペースに登録]** ウィンドウの**メールまたはワークスペースの URL** に「**https://client.wvd.microsoft.com/api/arm/feeddiscovery**」と入力して、**[次へ]** を選択し、メッセージが表示されたら、Microsoft Entra 資格情報を使用してサインインします。

1. **[リモート デスクトップ]** ページに **[SessionDesktop]** アイコンのみが表示されていることを確認します。

   > **注**: これは、選択した Microsoft Entra ユーザー アカウントが、最初のラボ *Azure portal を使用してホスト プールとセッション ホストをデプロイする (Entra ID)* で、自動生成された **az140-21-hp1-DAG** デスクトップ アプリケーション グループに割り当てられたためです。

1. **[リモート デスクトップ]** ページで、**[SessionDesktop]** アイコンを右クリックし、ポップアップ メニューで **[設定]** を選択します。
1. **[SessionDesktop]** ペインで、**[既定の設定を使用]** スイッチをオフにします。
1. **[ディスプレイの設定]** セクションのドロップダウン メニューで、エントリ **[ディスプレイの選択]** を選択し、セッションに使用するディスプレイを選択します。
1. **[SessionDesktop]** ペインで、**[現在のディスプレイに合わせて最大化]**、**[ウィンドウ モードの場合はシングル ディスプレイ]**、**[セッションをウィンドウ サイズに合わせる]** など、残りのオプションを変更せずに確認します。 
1. **[SessionDesktop]** ペインを閉じます。 
1. **[リモート デスクトップ]** ページで、**[SessionDesktop]** アイコンをダブルクリックします。
1. サインインを求められたら、**[Windows セキュリティ]** ダイアログ ボックスに、このタスクでお使いの Azure Virtual Desktop 環境への接続に使用した最初の Microsoft Entra ユーザー アカウントのパスワードを入力します。

   > **注**: Azure Virtual Desktop では、1 つのユーザー アカウントで Microsoft Entra ID にサインインしてから、別のユーザー アカウントで Windows にサインインすることはできません。 2 つの異なるアカウントで同時にサインインすると、ユーザーが間違ったセッション ホストに再接続したり、Azure portal の情報が正しくないか表示されなかったり、アプリ アタッチまたは MSIX アプリ アタッチの使用中にエラー メッセージが表示されたりする可能性があります。

   > **注**: **[SessionDesktop]** ウィンドウが自動的に表示されます。

1. [リモート デスクトップ] セッション ウィンドウで、セッション内で完全な管理アクセス権があることを確認します (たとえば、タスク バーの **Windows** ロゴ アイコンを選択し、ポップアップ メニューから **[Windows PowerShell (管理者)]** 項目を選択します)。
1. [リモート デスクトップ] セッション ウィンドウで、タスク バーの Windows ロゴ アイコンを選択し、サインインに使用した Microsoft Entra ユーザー アカウントを表すアバター アイコンを選択し、ポップアップ メニューで **[サインアウト]** を選択します。

   > **注**: これにより、リモート デスクトップ セッションが自動的に終了します。 

1. **[リモート デスクトップ]** ウィンドウに戻り、**az140-21-ws1** ワークスペース エントリの右側にある省略記号 (`...`) アイコンを選択し、**[登録を解除]** を選択し、確認を求められたら **[続行]** を選択します。
1. **[リモート デスクトップ]** クライアント ウィンドウで、**[登録]** を選択し、メッセージが表示されたら、ラボ インターフェイス ウィンドウの右側のペインにある **[リソース]** タブで検索できる 2 番目の Entra ID ユーザー アカウントの資格情報でサインインします。

   > **注**: **AVD-RemoteApp** プレフィックスを持つ Entra グループのメンバーであるユーザー アカウントを選択します。

1. **[リモート デスクトップ]** ページに、コマンド プロンプト、Microsoft Word、Microsoft Excel、Microsoft PowerPoint などの 4 つのアイコンが表示されていることを確認します。 

   > **注**: これは、選択した Microsoft Entra ユーザー アカウントが、最初のラボ *Azure portal を使用してホスト プールとセッション ホストをデプロイする (Entra ID)* で、**az140-21-hp1-Office365-RAG** および **az140-21-hp1-Utilities-RAG** アプリケーション グループに割り当てられたためです。

1. [コマンド プロンプト] アイコンをダブルクリックします。 
1. サインインを求められたら、**[Windows セキュリティ]** ダイアログ ボックスに、お使いの Azure Virtual Desktop 環境への接続に使用した 2 番目の Microsoft Entra ユーザー アカウントのパスワードを入力します。
1. 直後に **[コマンド プロンプト]** ウィンドウが表示されることを確認します。 
1. [コマンド プロンプト] ウィンドウで、「**ホスト名**」と入力し、**Enter** キーを押して、コマンド プロンプトが実行されているコンピューターの名前を表示します。

   > **注**: 表示される名前が **sh-** プレフィックスで始まっていることを確認します。

1. コマンド プロンプトで、「**ログオフ**」と入力し、**Enter** キーを押して現在のリモート アプリ セッションからログオフします。
1. **[リモート デスクトップ]** ページの残りのアイコンをダブルクリックして、Microsoft Word、Microsoft Excel、Microsoft PowerPoint を起動します。
1. 各セッション ウィンドウを閉じます。