---
title: Visual Studio および Git と ASP.NET Core を組み合わせた Azure への継続的配置
author: rick-anderson
description: Visual Studio で ASP.NET Core Web アプリを作成し、それを Azure App Service に配置する方法について説明します。Git を利用し、継続的に配置します。
ms.author: riande
ms.custom: devx-track-csharp, mvc
ms.date: 12/06/2018
no-loc:
- appsettings.json
- ASP.NET Core Identity
- cookie
- Cookie
- Blazor
- Blazor Server
- Blazor WebAssembly
- Identity
- Let's Encrypt
- Razor
- SignalR
uid: host-and-deploy/azure-apps/azure-continuous-deployment
ms.openlocfilehash: 561f41dd9d50df0651c060fac4f75d9d59244029
ms.sourcegitcommit: acfe51c35497a204f75c2a61125c9408c04493e6
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/10/2021
ms.locfileid: "102605647"
---
# <a name="continuous-deployment-to-azure-with-visual-studio-and-git-with-aspnet-core"></a>Visual Studio および Git と ASP.NET Core を組み合わせた Azure への継続的配置

作成者: [Erik Reitan](https://github.com/Erikre)

[!INCLUDE [Azure App Service Preview Notice](../../includes/azure-apps-preview-notice.md)]

このチュートリアルでは、Visual Studio で ASP.NET Core Web アプリを作成し、それを Visual Studio から Azure App Service に継続的に配置する方法について説明します。

[Azure Pipelines を使用した初めてのパイプラインの作成](/azure/devops/pipelines/get-started-yaml)に関する記事を参照してください。記事では、Azure DevOps Services を使用した [Azure App Service](/azure/app-service/app-service-web-overview) 向けの継続的デリバリー (CD) ワークフローの構成方法が示されています。 Azure Pipelines (Azure DevOps Services サービス) を利用すると、Azure App Service でホストされているアプリの更新プログラムを公開するための堅牢な配置パイプラインの設定が簡単になります。 Azure Portal で、このパイプラインのビルド、テスト実行、ステージング スロットへの配置、運用への配置を構成できます。

> [!NOTE]
> このチュートリアルを完了するには、Microsoft Azure アカウントが必要です。 アカウントを持っていない場合は、[MSDN サブスクライバー特典を有効にする](https://azure.microsoft.com/pricing/member-offers/credit-for-visual-studio-subscribers/?WT.mc_id=A261C142F)か、[無料試用版にサインアップ](https://azure.microsoft.com/free/dotnet/?WT.mc_id=A261C142F)してください。

## <a name="prerequisites"></a>必須コンポーネント

このチュートリアルでは、次のソフトウェアがインストールされていることを前提としています。

* [Visual Studio](https://visualstudio.microsoft.com)
* [!INCLUDE [](~/includes/net-core-sdk-download-link.md)]
* [Git](https://git-scm.com/downloads) for Windows

## <a name="create-an-aspnet-core-web-app"></a>ASP.NET Core Web アプリを作成する

1. Visual Studio を起動します。

1. **[ファイル]** メニューで **[新規作成]**  >  **[プロジェクト]** の順に選択します。

1. **[ASP.NET Core Web アプリケーション]** プロジェクト テンプレートを選択します。 このテンプレートは、 **[インストール済み]**  >  **[テンプレート]**  >  **[Visual C#]**  >  **[.NET Core]** の下にあります。 プロジェクトに `SampleWebAppDemo` という名前を付けます。 **[新しい Git リポジトリの作成]** オプションを選択し、 **[OK]** をクリックします。

   ![[新しいプロジェクト] ダイアログ](azure-continuous-deployment/_static/01-new-project.png)

1. **[New ASP.NET Core Project]\(新しい ASP.NET Core プロジェクト\)** ダイアログで、ASP.NET Core の **[空]** テンプレートを選択し、 **[OK]** をクリックします。

   ![[新しい ASP.NET Core プロジェクト] ダイアログ](azure-continuous-deployment/_static/02-web-site-template.png)

> [!NOTE]
> .NET Core の最新のリリースは 2.0 です。

### <a name="running-the-web-app-locally"></a>Web アプリのローカル実行

1. Visual Studio でアプリの作成が完了したら、 **[デバッグ]**  >  **[デバッグの開始]** の順に選択し、アプリを実行します。 または、**F5** キーを押します。

   Visual Studio と新しいアプリの初期化に時間がかかる場合があります。 完了すると、ブラウザーに実行中のアプリが表示されます。

   ![ブラウザー ウィンドウの実行中のアプリに 'Hello World!' と表示されます。](azure-continuous-deployment/_static/04-browser-runapp.png)

1. 実行中の Web アプリを確認したら、ブラウザーを閉じ、Visual Studio のツール バーにある "デバッグの停止" アイコンを選択してアプリを停止します。

## <a name="create-a-web-app-in-the-azure-portal"></a>Azure Portal で Web アプリを作成する

Azure Portal で次の手順を実行して Web アプリを作成します。

1. [Azure Portal](https://portal.azure.com) にログインします。

1. Portal インターフェイスの左上にある **[新規作成]** を選択します。

1. **[Web + モバイル]**  >  **[Web アプリ]** の順に選択します。

   ![Microsoft Azure Portal:新しいボタン:Marketplace での Web + モバイル:おすすめアプリでの Web アプリのボタン](azure-continuous-deployment/_static/05-azure-newwebapp.png)

1. **[Web アプリ]** ブレードに **[App Service の名前]** の一意の名前を入力します。

   ![[Web アプリ] ブレード](azure-continuous-deployment/_static/06-azure-newappblade.png)

   > [!NOTE]
   > **[App Service の名前]** の名前は一意にする必要があります。 名前を入力する場合、この規則が適用されます。 別の値を入力する場合は、このチュートリアルで **SampleWebAppDemo** が出てくるたびに、その値を置き換えてください。

   **[Web アプリ]** ブレードではまた、既存の **[App Service プラン/場所]** を選択するか、新しい App Service プラン/場所を作成できます。 新しいプランを作成した場合、価格レベル、場所、その他のオプションを選択します。 App Service プランに関する詳細については、「[Azure App Service プランの概要](/azure/app-service/azure-web-sites-web-hosting-plans-in-depth-overview)」を参照してください。

1. **[作成]** を選択します。 Web アプリがプロビジョニングされ、起動します。

   ![Azure Portal: サンプル Web アプリ デモ 01 の [要点] ブレード](azure-continuous-deployment/_static/07-azure-webappblade.png)

## <a name="enable-git-publishing-for-the-new-web-app"></a>新しい Web アプリに対して Git 公開を有効にする

Git は分散型バージョン管理システムであり、これを利用して Azure App Service Web アプリを展開できます。 Web アプリ コードはローカルの Git リポジトリに格納されます。また、コードはリモート リポジトリにプッシュすることによって Azure に展開されます。

1. [Azure Portal](https://portal.azure.com) にログインします。

1. **[App Services]** を選択し、Azure サブスクリプションに関連付けられているアプリ サービスの一覧を表示します。

1. このチュートリアルの前のセクションで作成した Web アプリを選択します。

1. **[デプロイ]** ブレードで、 **[デプロイ オプション]**  >  **[ソースの選択]**  >  **[ローカル Git リポジトリ]** の順に選択します。

   ![[設定] ブレード:[展開ソース] ブレード:[ソースの選択] ブレード](azure-continuous-deployment/_static/deployment-options.png)

1. **[OK]** を選択します。

1. Web アプリや他の App Service アプリを公開するための展開の資格情報を設定していない場合は、ここで設定してください。

   * **[設定]**  >  **[デプロイ資格情報]** の順に選択します。 **[デプロイ資格情報の設定]** ブレードが表示されます。
   * ユーザー名とパスワードを作成します。 後で Git を設定するときに使用するパスワードを保存します。
   * **[保存]** を選択します。

1. **[Web アプリ]** ブレードで、 **[設定]**  >  **[プロパティ]** の順に選択します。 展開するリモート Git リポジトリの URL が **[GIT URL]** の下に表示されます。

1. **GIT URL** の値をコピーします。後で使用します。

   ![Azure Portal: アプリケーションの [プロパティ] ブレード](azure-continuous-deployment/_static/09-azure-giturl.png)

## <a name="publish-the-web-app-to-azure-app-service"></a>Azure App Service に Web アプリを発行する

このセクションでは、Visual Studio でローカル Git リポジトリを作成し、そのリポジトリから Azure にプッシュし、Web アプリを展開します。 実行する必要がある手順は、次のとおりです。

* ローカル リポジトリを Azure に展開できるように、GIT URL 値を使用してリモート リポジトリ設定を追加します。
* プロジェクトの変更をコミットします。
* ローカル リポジトリから Azure のリモート リポジトリにプロジェクトの変更をプッシュします。

1. **ソリューション エクスプローラー** で **[ソリューション 'SampleWebAppDemo']** を右クリックし、 **[コミット]** を選択します。 **チーム エクスプローラー** が表示されます。

   ![[チーム エクスプローラー接続] タブ](azure-continuous-deployment/_static/10-team-explorer.png)

1. **チーム エクスプローラー** で、 **[ホーム]** (ホーム アイコン)、 **[設定]**  >  **[リポジトリ設定]** の順に選択します。

1. **[リポジトリ設定]** の **[リモート]** セクションで、 **[追加]** を選択します。 **[リモートの追加]** ダイアログ ボックスが表示されます。

1. リモートの **[名前]** を **[Azure-SampleApp]** に設定します。

1. **[フェッチ]** の値を先にこのチュートリアルで Azure からコピーした **Git URL** に設定します。 これは **.git** で終わる URL であることにご注意ください。

   ![[リモートの編集] ダイアログ](azure-continuous-deployment/_static/11-add-remote.png)

   > [!NOTE]
   > あるいは、**コマンド ウィンドウ** からリモート リポジトリを指定できます。**コマンド ウィンドウ** を開き、プロジェクト ディレクトリに移動し、コマンドを入力します。 例:
   >
   > `git remote add Azure-SampleApp https://me@sampleapp.scm.azurewebsites.net:443/SampleApp.git`

1. **[ホーム]** (ホーム アイコン)、 **[設定]**  >  **[グローバル設定]** の順に選択します。 名前と電子メール アドレスが設定されていることを確認します。 必要に応じて **[更新]** を選択します。

1. **[ホーム]**  >  **[変更]** の順に選択し、 **[変更]** ビューに戻ります。

1. 「**Initial Push #1**」のようなコミット メッセージを入力し、 **[コミット]** を選択します。 このアクションによりローカルで *コミット* されます。

   ![[チーム エクスプローラー接続] タブ](azure-continuous-deployment/_static/12-initial-commit.png)

   > [!NOTE]
   > あるいは、**コマンド ウィンドウ** から変更をコミットできます。**コマンド ウィンドウ** を開き、プロジェクト ディレクトリに移動し、git コマンドを入力します。 例:
   >
   > `git add .`
   >
   > `git commit -am "Initial Push #1"`

1. **[ホーム]**  >  **[同期]**  >  **[アクション]**  >  **[コマンド プロンプトを開く]** の順に選択します。 コマンド プロンプトでプロジェクト ディレクトリが開きます。

1. コマンド ウィンドウで次のコマンドを入力します。

   `git push -u Azure-SampleApp main`

1. Azure で先に作成した Azure **デプロイ資格情報** パスワードを入力します。

   このコマンドによりローカル プロジェクト ファイルを Azure にプッシュするプロセスが開始されます。 上記のコマンドからの出力は、展開が成功したというメッセージで終わります。

   ```
   remote: Finished successfully.
   remote: Running post deployment command(s)...
   remote: Deployment successful.
   To https://username@samplewebappdemo01.scm.azurewebsites.net:443/SampleWebAppDemo01.git
   * [new branch]      main -> main
   Branch main set up to track remote branch main from Azure-SampleApp.
   ```

   > [!NOTE]
   > プロジェクトのコラボレーションが必要な場合は、Azure にプッシュする前に [GitHub](https://github.com) にプッシュすることを検討してください。
 
### <a name="verify-the-active-deployment"></a>アクティブ デプロイを検証する

ローカル環境から Azure への Web アプリの転送が成功していることを確認します。

[Azure Portal](https://portal.azure.com) で、Web アプリを選択します。 **[デプロイ]**  >  **[デプロイ オプション]** の順に選択します。

![Azure Portal: [設定] ブレード:[デプロイ] ブレードにデプロイ成功が表示されています](azure-continuous-deployment/_static/13-verify-deployment.png)

## <a name="run-the-app-in-azure"></a>Azure でアプリを実行する

Web アプリが Azure に展開されたら、アプリを実行します。

その方法は 2 つあります。

* Azure Portal で、Web アプリ用の Web アプリ ブレードを探します。 **[参照]** を選択して、既定のブラウザーでアプリを表示します。
* ブラウザーを開き、Web アプリの URL を入力します。 例 : `http://SampleWebAppDemo.azurewebsites.net`

## <a name="update-the-web-app-and-republish"></a>Web アプリを更新し、再発行する

ローカル コードを変更した後は、再発行します。

1. Visual Studio の **ソリューション エクスプローラー** で、*Startup.cs* ファイルを開きます。

1. `Configure` メソッドで、次のように表示されるように `Response.WriteAsync` メソッドを変更します。

   ```csharp
   await context.Response.WriteAsync("Hello World! Deploy to Azure.");
   ```

1. 変更を *Startup.cs* に保存します。

1. **ソリューション エクスプローラー** で **[ソリューション 'SampleWebAppDemo']** を右クリックし、 **[コミット]** を選択します。 **チーム エクスプローラー** が表示されます。

1. `Update #2` のようなコミット メッセージを入力します。

1. **[コミット]** ボタンを押し、プロジェクトの変更をコミットします。

1. **[ホーム]**  >  **[同期]**  >  **[アクション]**  >  **[プッシュ]** の順に選択します。

> [!NOTE]
> あるいは、**コマンド ウィンドウ** から変更をプッシュできます。**コマンド ウィンドウ** を開き、プロジェクト ディレクトリに移動し、git コマンドを入力します。 例:
> 
> `git push -u Azure-SampleApp main`

## <a name="view-the-updated-web-app-in-azure"></a>Azure で更新後の Web アプリを表示する

更新後の Web アプリを表示します。Azure Portal の Web アプリ ブレードから **[参照]** を選択するか、ブラウザーを開き、Web アプリの URL を入力します。 例 : `http://SampleWebAppDemo.azurewebsites.net`

## <a name="additional-resources"></a>その他の技術情報

* [Azure Pipelines による最初のパイプラインの作成](/azure/devops/pipelines/get-started-yaml)
* [プロジェクト Kudu](https://github.com/projectkudu/kudu/wiki)
* <xref:host-and-deploy/visual-studio-publish-profiles>
