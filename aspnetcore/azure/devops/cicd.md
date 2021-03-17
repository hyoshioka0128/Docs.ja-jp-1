---
title: 継続的インテグレーションおよび継続的デプロイ - ASP.NET Core と Azure を使用した DevOps
author: CamSoper
description: ASP.NET Core と Azure を使用した DevOps での継続的インテグレーションおよび継続的デプロイ
ms.author: scaddie
ms.date: 10/24/2018
ms.custom: devx-track-csharp, mvc, seodec18
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
uid: azure/devops/cicd
ms.openlocfilehash: 18b2c6ce27132844402f88b2817a07e3588d81c1
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/10/2021
ms.locfileid: "102586268"
---
# <a name="continuous-integration-and-deployment"></a>継続的インテグレーションと継続的配置

前の章では、Simple Feed Reader アプリ用にローカル Git リポジトリを作成しました。 この章では、そのコードを GitHub リポジトリに発行してから、Azure Pipelines を使用して Azure DevOps Services パイプラインを構築します。 パイプラインを使用すれば、アプリの継続的なビルドとデプロイが可能になります。 GitHub リポジトリへのコミットによって、ビルドと Azure Web アプリのステージング スロットへのデプロイがトリガーされます。

このセクションでは、次のタスクを実行します。

* アプリのコードを GitHub に発行する
* ローカル Git デプロイの接続を解除する
* Azure DevOps 組織を作成する
* Azure DevOps Services でチーム プロジェクトを作成する
* ビルド定義の作成
* リリース パイプラインを作成する
* GitHub への変更をコミットし、Azure に自動的にデプロイする
* Azure Pipelines パイプラインを調べる

## <a name="publish-the-apps-code-to-github"></a>アプリのコードを GitHub に発行する

1. ブラウザー ウィンドウを開き、`https://github.com` に移動します。
1. ヘッダーの **[+]** ドロップダウンをクリックし、 **[新しいリポジトリ]** を選択します。

    ![GitHub の [新しいリポジトリ] オプション](media/cicd/github-new-repo.png)

1. **[所有者]** ドロップダウンでご自分のアカウントを選択し、 **[リポジトリ名]** テキストボックスに「*simple-feed-reader*」と入力します。
1. **[リポジトリの作成]** ボタンをクリックします。
1. ご利用のローカル コンピューターのコマンド シェルを開きます。 *simple-feed-reader* Git リポジトリが格納されているディレクトリに移動します。
1. 既存の *origin* リモートを *upstream* に名前変更します。 次のコマンドを実行します。

    ```console
    git remote rename origin upstream
    ```

1. GitHub にあるリポジトリのご自分のコピーを指す新しい *origin* リモートを追加します。 次のコマンドを実行します。

    ```console
    git remote add origin https://github.com/<GitHub_username>/simple-feed-reader/
    ```

1. 新しく作成した GitHub リポジトリにご自分のローカル Git リポジトリを発行します。 次のコマンドを実行します。

    ```console
    git push -u origin main
    ```

1. ブラウザー ウィンドウを開き、`https://github.com/<GitHub_username>/simple-feed-reader/` に移動します。 GitHub リポジトリにご自分のコードが表示されることを確認します。

## <a name="disconnect-local-git-deployment"></a>ローカル Git デプロイの接続を解除する

次の手順に従って、ローカル Git デプロイを削除します。 その機能は Azure Pipelines (Azure DevOps サービス) に置き換えられ、強化されます。

1. [Azure portal](https://portal.azure.com/) を開き、*staging (mywebapp\<unique_number\>/staging)* Web アプリに移動します。 この Web アプリは、ポータルの検索ボックスに 「*staging*」と入力すれば、すぐに見つけることができます。

    ![Web アプリの検索用語 staging](media/cicd/portal-search-box.png)

1. **デプロイ センター** をクリックします。 新しいパネルが表示されます。 **[接続解除]** をクリックして、前の章で追加したローカル Git ソース管理構成を削除します。 **[はい]** ボタンをクリックして削除操作を確定します。
1. *mywebapp<unique_number>* App Service に移動します。 繰り返しますが、ポータルの検索ボックスを使用すれば、App Service をすばやく見つけることができます。
1. **デプロイ センター** をクリックします。 新しいパネルが表示されます。 **[接続解除]** をクリックして、前の章で追加したローカル Git ソース管理構成を削除します。 **[はい]** ボタンをクリックして削除操作を確定します。

## <a name="create-an-azure-devops-organization"></a>Azure DevOps 組織を作成する

1. ブラウザーを開き、[Azure DevOps 組織作成ページ](https://go.microsoft.com/fwlink/?LinkId=307137)に移動します。
1. **[覚えやすい名前を選んでください]** テキストボックスに一意の名前を入力して、ご自分の Azure DevOps 組織にアクセスするための URL を作成します。
1. コードは GitHub リポジトリでホストされているため、 **[Git]** ラジオ ボタンを選択します。
1. **[Continue]** をクリックします。 しばらく待つと *MyFirstProject* という名前のアカウントとチームプロジェクトが作成されます。

    ![Azure DevOps 組織作成ページ](media/cicd/vsts-account-creation.png)

1. Azure DevOps の組織とプロジェクトが使用できる状態であることを示す確認メールを開きます。 次に示す **[プロジェクトの開始]** ボタンをクリックします。

    ![[プロジェクトの開始] ボタン](media/cicd/vsts-start-project.png)

1. ブラウザーが開き、 *\<account_name\>.visualstudio.com* が表示されます。 *[MyFirstProject]* リンクをクリックして、プロジェクトの DevOps パイプラインの構成を開始します。

## <a name="configure-the-azure-pipelines-pipeline"></a>Azure Pipelines パイプラインを構成する

3 つのステップを個別に実行する必要があります。 以下の 3 つのセクションの手順を完了すると、運用 DevOps パイプラインが生成されます。

### <a name="grant-azure-devops-access-to-the-github-repository"></a>GitHub リポジトリへのアクセス権を Azure DevOps に付与する

1. **[または外部リポジトリからコードを構築する]** アコーディオンを展開します。 **[ビルドのセットアップ]** ボタンをクリックします。

    ![[ビルドのセットアップ] ボタン](media/cicd/vsts-setup-build.png)

1. **[ソースの選択]** セクションから **[GitHub]** オプションを選択します。

    ![ソースの選択 - GitHub](media/cicd/vsts-select-source.png)

1. ご利用の GitHub リポジトリに Azure DevOps がアクセスできるようにするには、事前に承認が必要です。 **[接続名]** テキストボックスに「 *<GitHub_username> GitHub connection*」と入力します。 次に例を示します。

    ![GitHub 接続名](media/cicd/vsts-repo-authz.png)

1. ご利用の GitHub アカウントで 2 要素認証が有効になっている場合は、個人用アクセス トークンが必要です。 その場合は、 **[GitHub 個人用アクセス トークンで認証する]** リンクをクリックします。 詳細については、[公式の GitHub 個人用アクセストークン作成手順](https://help.github.com/articles/creating-a-personal-access-token-for-the-command-line/)に関するページを参照してください。 必要な権限のスコープは *repo* のみです。 それ以外の場合は、 **[OAuth を使用して承認]** ボタンをクリックします。
1. メッセージが表示されたら、ご自分の GitHub アカウントにサインインします。 次に、[承認] を選択して、ご自分の Azure DevOps 組織へのアクセスを許可します。 成功した場合は、新しいサービス エンドポイントが作成されます。
1. **[リポジトリ]** ボタンの横にある省略記号ボタンをクリックします。 一覧から、 *<GitHub_username>/simple-feed-reader* リポジトリを選択します。 **[選択]** ボタンをクリックします。
1. **[手動のビルドとスケジュールされたビルドの既定のブランチ]** ドロップダウンから既定のブランチ (*main*) を選択します。 **[Continue]** をクリックします。 テンプレートの選択ページが表示されます。

### <a name="create-the-build-definition"></a>ビルド定義を作成する

1. テンプレートの選択ページの [検索] ボックスに「*ASP.NET Core*」と入力します。

    ![テンプレート ページでの ASP.NET Core の検索](media/cicd/vsts-template-selection.png)

1. テンプレートの検索結果が表示されます。 **[ASP.NET Core]** テンプレートにマウス ポインターを合わせ、 **[適用]** ボタンをクリックします。
1. ビルド定義の **[タスク]** タブが表示されます。 **[トリガー]** タブをクリックします。
1. **[継続的インテグレーションを有効にする]** ボックスをオンにします。 **[ブランチ フィルター]** セクションで、 **[種類]** ドロップダウンが *[Include]* に設定されていることを確認します。 **[ブランチ仕様]** ドロップダウンを *main* に設定します。

    ![[継続的インテグレーションを有効にする] に関する設定](media/cicd/vsts-enable-ci.png)

    これらの設定を行うと、GitHub リポジトリの既定のブランチ (*main*) に何らかの変更がプッシュされたときに、ビルドがトリガーされます。 継続的インテグレーションのテストについては、「[GitHub への変更をコミットし、Azure に自動的にデプロイする](#commit-changes-to-github-and-automatically-deploy-to-azure)」セクションで行います。

1. **[保存してキューに登録]** ボタンをクリックし、 **[保存]** オプションを選択します。

    ![[保存] ボタン](media/cicd/vsts-save-build.png)

1. 次のモーダル ダイアログが表示されます。

    ![ビルド定義を保存する - モーダル ダイアログ](media/cicd/vsts-save-modal.png)

    既定のフォルダーである *\\* を使用して、 **[保存]** ボタンをクリックします。

### <a name="create-the-release-pipeline"></a>リリース パイプラインの作成

1. チーム プロジェクトの **[リリース]** タブをクリックします。 **[新しいパイプライン]** ボタンをクリックします。

    ![[リリース] タブ - [新しい定義] ボタン](media/cicd/vsts-new-release-definition.png)

    テンプレート選択ウィンドウが表示されます。

1. テンプレート選択ページで、[検索] ボックスに「*App Service*」と入力します。

    ![リリース パイプライン テンプレートの検索ボックス](media/cicd/vsts-release-template-search.png)

1. テンプレートの検索結果が表示されます。 **[スロットを使用した Azure App Service の配置]** テンプレートにマウス ポインターを合わせ、 **[適用]** ボタンをクリックします。 リリース パイプラインの **[パイプライン]** タブが表示されます。

    ![リリース パイプラインの [パイプライン] タブ](media/cicd/vsts-release-definition-pipeline.png)

1. **[成果物]** ボックスの **[追加]** ボタンをクリックします。 **[成果物の追加]** パネルが表示されます。

    ![リリース パイプライン - [成果物の追加] パネル](media/cicd/vsts-release-add-artifact.png)

1. **[ソースの種類]** セクションから **[ビルド]** タイルを選択します。 この種類を選択すれば、リリース パイプラインをビルド定義にリンクすることができます。
1. **[プロジェクト]** ドロップダウンから *[MyFirstProject]* を選択します。
1. **[ソース (ビルド定義)]** ドロップダウンから、ビルド定義名として *MyFirstProject-ASP.NET Core-CI* を選択します。
1. **[既定のバージョン]** ドロップダウンから *[最新]* を選択します。 このオプションでは、ビルド定義の最新の実行によって生成された成果物がビルドされます。
1. **[ソースの別名]** テキストボックス内のテキストを *Drop* に置き換えます。
1. **[追加]** ボタンをクリックします。 **[成果物]** セクションが更新され、変更内容が表示されます。
1. [稲妻] アイコンをクリックして、継続的デプロイを有効にします。

    ![リリース パイプラインの成果物 - [稲妻] アイコン](media/cicd/vsts-artifacts-lightning-bolt.png)

    このオプションを有効にすると、新しいビルドが使用可能になるたびにデプロイが行われます。
1. **[継続的デプロイ トリガー]** パネルが右側に表示されます。 トグル ボタンをクリックして、この機能を有効にします。 **[Pull request のトリガー]** を有効にする必要はありません。
1. **[ビルド ブランチ フィルター]** セクションの **[追加]** ドロップダウンをクリックします。 **[ビルド定義の既定のブランチ]** オプションを選択します。 このフィルターを使用すると、GitHub リポジトリの既定のブランチ (*main*) からのビルドの場合にのみ、リリースがトリガーされます。
1. **[保存]** ボタンをクリックします。 結果として生成された **[保存]** モーダル ダイアログの **[OK]** ボタンをクリックします。
1. **[環境 1]** ボックスをクリックします。 **[環境]** パネルが右側に表示されます。 **[環境名]** テキストボックスのテキスト "*環境 1*" を "*運用*" に変更します。

   ![リリース パイプライン - [環境名] テキストボックス](media/cicd/vsts-environment-name-textbox.png)

1. **[運用]** ボックスで、 **[1 フェーズ、2 タスク]** リンクをクリックします。

    ![リリース パイプライン - 運用環境リンク.png](media/cicd/vsts-production-link.png)

    その環境の **[タスク]** タブが表示されます。
1. **[Deploy Azure App Service to Slot]\(Azure App Service をスロットにデプロイ\)** タスクをクリックします。 その設定が右側のパネルに表示されます。
1. App Service に関連付けられている Azure サブスクリプションを **[Azure サブスクリプション]** ドロップダウンから選択します。 選択したら、 **[承認]** ボタンをクリックします。
1. **[アプリの種類]** ドロップダウンから *[Web アプリ]* を選択します。
1. **[App Service の名前]** ドロップダウンから *mywebapp/<unique_number/>* を選択します。
1. *[リソース グループ]* ドロップダウンから **[AzureTutorial]** を選択します。
1. **[スロット]** ドロップダウンから *[ステージング]* を選択します。
1. **[保存]** ボタンをクリックします。
1. 既定のリリース パイプライン名にマウス ポインターを合わせます。 それを編集するには、鉛筆アイコンをクリックします。 名前として *MyFirstProject-ASP.NET Core-CD* を使用します。

    ![リリース パイプラインの名前](media/cicd/vsts-release-definition-name.png)

1. **[保存]** ボタンをクリックします。

## <a name="commit-changes-to-github-and-automatically-deploy-to-azure"></a>GitHub への変更をコミットし、Azure に自動的にデプロイする

1. Visual Studio で *SimpleFeedReader.sln* を開きます。
1. ソリューション エクスプローラーで *Pages\Index.cshtml* を開きます。 `<h2>Simple Feed Reader - V3</h2>` を `<h2>Simple Feed Reader - V4</h2>` に変更します。
1. **Ctrl** + **Shift** + **B** キーを押して、アプリをビルドします。
1. GitHub リポジトリにファイルをコミットします。 Visual Studio の *[チーム エクスプローラー]* タブの **[変更]** ページを使用するか、ローカル コンピューターのコマンド シェルを使用して以下を実行します。

    ```console
    git commit -a -m "upgraded to V4"
    ```

1. 既定のブランチ (*main*) での変更を、ご利用の GitHub リポジトリの *origin* リモートにプッシュします。 次のコマンドで、プレースホルダー `{BRANCH}` を既定のブランチ (`main` を使用) に置き換えます。

    ```console
    git push origin {BRANCH}
    ```

    このコミットが、GitHub リポジトリの既定のブランチ (*main*) に表示されます。

    ![既定のブランチ (main) での GitHub コミット](media/cicd/github-commit.png)

    ビルド定義の **[トリガー]** タブで継続的インテグレーションが有効になっているため、ビルドがトリガーされます。

    ![継続的インテグレーションを有効にする](media/cicd/enable-ci.png)

1. Azure DevOps Services で、 **[Azure Pipelines]**  >  **[ビルド]** ページの **[キューに登録済み]** タブに移動します。 キューに登録されたビルドに、ビルドをトリガーしたブランチとコミットが表示されます。

    ![キューに登録されたビルド](media/cicd/build-queued.png)

1. ビルドが成功すると、Azure へのデプロイが行われます。 ブラウザーでアプリに移動します。 見出しに "V4" というテキストが表示されるのがわかります。

    ![更新されたアプリ](media/cicd/updated-app-v4.png)

## <a name="examine-the-azure-pipelines-pipeline"></a>Azure Pipelines パイプラインを調べる

### <a name="build-definition"></a>ビルド定義

ビルド定義は *MyFirstProject-ASP.NET Core-CI* という名前で作成しました。 完了時には、発行する資産を含む *.zip* ファイルがビルドによって生成されます。 それらの資産はリリース パイプラインによって Azure にデプロイされます。

ビルド定義の **[タスク]** タブには、使用されている個々のステップが一覧表示されます。 ビルド タスクは 5 つあります。

![ビルド定義タスク](media/cicd/build-definition-tasks.png)

1. **復元** &mdash; `dotnet restore` コマンドを実行して、アプリの NuGet パッケージが復元されます。 使用される既定のパッケージ フィードは nuget.org です。
1. **ビルド** &mdash; `dotnet build --configuration release` コマンドを実行して、アプリのコードがコンパイルされます。 この `--configuration` オプションは、コードの最適化されたバージョン (運用環境へのデプロイに適している) を生成するために使用されます。 たとえば、デバッグ構成が必要な場合は、ビルド定義の **[変数]** タブで *BuildConfiguration* 変数を変更します。
1. **テスト** &mdash; `dotnet test --configuration release --logger trx --results-directory <local_path_on_build_agent>` コマンドを実行して、アプリの単体テストが実行されます。 単体テストは、`**/*Tests/*.csproj` glob パターンに一致する C# プロジェクト内で実行されます。 `--results-directory` オプションで指定された場所にある *.trx* ファイルにテスト結果が保存されます。 いずれかのテストが失敗した場合、ビルドは失敗し、デプロイは行われません。

    > [!NOTE]
    > 単体テストが機能していることを確認するには、テストの 1 つを意図的に中断するように *SimpleFeedReader.Tests\Services\NewsServiceTests.cs* を変更します。 たとえば、`Returns_News_Stories_Given_Valid_Uri` メソッド内で `Assert.True(result.Count > 0);` を `Assert.False(result.Count > 0);` に変更します。 変更をコミットして GitHub にプッシュします。 ビルドがトリガーされますが失敗します。 ビルド パイプラインの状態が **[失敗]** に変わります。 変更を元に戻し、コミットして、もう一度プッシュします。 ビルドは成功します。

1. **発行** &mdash; `dotnet publish --configuration release --output <local_path_on_build_agent>` コマンドを実行して、デプロイする成果物を含む *.zip* ファイルが生成されます。 `--output` オプションによって *.zip* ファイルの発行場所が指定されます。 その場所は、`$(build.artifactstagingdirectory)` という名前の[定義済みの変数](/azure/devops/pipelines/build/variables)を渡すことによって指定されます。 その変数は、ビルド エージェント上のローカル パス (*c:\agent\_work\1\a* など) に展開されます。
1. **成果物の発行** &mdash; **発行** タスクによって生成された *.zip* ファイルを発行します。 このタスクでは、 *.zip* ファイルの場所をパラメーターとして受け取ります。それは定義済みの変数 `$(build.artifactstagingdirectory)` です。 *.zip* ファイルは、*drop* という名前のフォルダーとして発行されます。

ビルド定義の **[概要]** リンクをクリックして、ビルドの定義に関する履歴を表示します。

![ビルド定義の履歴を示すスクリーンショット](media/cicd/build-definition-summary.png)

結果として表示されるページで、一意のビルド番号に対応するリンクをクリックします。

![ビルド定義の概要ページを示すスクリーンショット](media/cicd/build-definition-completed.png)

この特定のビルドの概要が表示されます。 **[成果物]** タブをクリックすると、ビルドによって生成された *drop* フォルダーが一覧に表示されていることがわかります。

![ビルド定義の成果物を示すスクリーンショット - drop フォルダー](media/cicd/build-definition-artifacts.png)

**[ダウンロード]** および **[探索]** リンクを使用して、発行された成果物を検査します。

### <a name="release-pipeline"></a>リリース パイプライン

リリース パイプラインは、*MyFirstProject-ASP.NET Core-CD* という名前で作成しました。

![リリース パイプラインの概要を示すスクリーンショット](media/cicd/release-definition-overview.png)

リリース パイプラインの 2 つの主要なコンポーネントは、 **[成果物]** と **[環境]** です。 **[成果物]** セクションのボックスをクリックすると、次のパネルが表示されます。

![リリース パイプラインの成果物を示すスクリーンショット](media/cicd/release-definition-artifacts.png)

**[ソース (ビルド定義)]** の値は、このリリース パイプラインがリンクされているビルド定義を表します。 ビルド定義が正常に実行された場合に生成される *.zip* ファイルは、Azure にデプロイするための "*運用*" 環境に提供されます。 *[運用]* 環境ボックス内の *[1 フェーズ、2 タスク]* リンクをクリックすると、リリース パイプラインのタスクが表示されます。

![リリース パイプラインのタスクを示すスクリーンショット](media/cicd/release-definition-tasks.png)

リリース パイプラインは、次の 2 つのタスクで構成されています: *[Deploy Azure App Service to Slot]\(Azure App Service をスロットにデプロイ\)* と *[Azure App Service の管理 - スロットのスワップ]* 。 最初のタスクをクリックすると、次のタスク構成が表示されます。

![リリース パイプラインのデプロイ タスクを示すスクリーンショット](media/cicd/release-definition-task1.png)

デプロイ タスクでは、Azure サブスクリプション、サービスの種類、Web アプリ名、リソース グループ、デプロイ スロットが定義されます。 **[パッケージまたはフォルダー]** テキストボックスには、*mywebapp\<unique_number\>* という Web アプリの "*ステージング*" スロットに抽出されてデプロイされる *.zip* ファイル パスが保持されています。

スロット スワップ タスクをクリックすると、次のタスク構成が表示されます。

![リリース パイプラインのスロット スワップ タスクを示すスクリーン ショット](media/cicd/release-definition-task2.png)

サブスクリプション、リソース グループ、サービスの種類、Web アプリ名、デプロイ スロットの詳細が提供されます。 **[本稼働とのスワップ]** チェック ボックスがオンになっています。 したがって、"*ステージング*" スロットにデプロイされたビットは運用環境にスワップされます。

## <a name="additional-reading"></a>その他の参考資料

* [Azure Pipelines による最初のパイプラインの作成](/azure/devops/pipelines/get-started-yaml)
* [ビルドと .NET Core プロジェクト](/azure/devops/pipelines/languages/dotnet-core)
* [Azure Pipelines を使用した Web アプリのデプロイ](/azure/devops/pipelines/targets/webapp)
