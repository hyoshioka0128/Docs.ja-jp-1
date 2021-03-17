---
title: アプリを App Service にデプロイする - ASP.NET Core および Azure を使用した DevOps
author: CamSoper
description: ASP.NET Core および Azure を使用した DevOps の最初のステップとして、ASP.NET Core アプリを Azure App Service にデプロイします。
ms.author: casoper
ms.custom: devx-track-csharp, mvc, seodec18, devx-track-azurecli
ms.date: 10/24/2018
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
uid: azure/devops/deploy-to-app-service
ms.openlocfilehash: 6d82611eb2176126652d0d6c8db3779cde70dab5
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/10/2021
ms.locfileid: "102586242"
---
# <a name="deploy-an-app-to-app-service"></a>App Service にアプリをデプロイする

[Azure App Service](/azure/app-service/) は、Azure の Web ホスティング プラットフォームです。 Azure App Service への Web アプリのデプロイは、手動または自動プロセスで行うことができます。 ガイドのこのセクションでは、手動で、コマンド ラインを使用してスクリプトで、または Visual Studio を使用して手動でトリガーできる、デプロイ方法について説明します。

このセクションでは、次のタスクを実行します。

* サンプル アプリをダウンロードしてビルドします。
* Azure Cloud Shell を使用して Azure App Service Web アプリを作成します。
* Git を使用してサンプル アプリを Azure にデプロイします。
* Visual Studio を使用して変更をアプリにデプロイします。
* ステージング スロットを Web アプリに追加します。
* 更新プログラムをステージング スロットにデプロイします。
* ステージング スロットと運用スロットをスワップします。

## <a name="download-and-test-the-app"></a>アプリをダウンロードしてテストする

このガイドで使用するアプリは、ビルド済みの ASP.NET Core アプリである [Simple Feed Reader](https://github.com/Azure-Samples/simple-feed-reader/) です。 これは、`Microsoft.SyndicationFeed.ReaderWriter` API を使用して RSS または Atom フィードを取得し、ニュース項目を一覧で表示する Razor Pages アプリです。

コードを自由に確認してかまいませんが、このアプリに関して特別なことは何もないことを理解しておくことが重要です。 例示だけを目的とするシンプルな ASP.NET Core アプリです。

次のようにコマンド シェルからコードをダウンロードし、プロジェクトをビルドして、実行します。

> *注:Linux/macOS ユーザーは、パスを適切に変更する必要があります。たとえば、バック スラッシュ (`\`) ではなくスラッシュ (`/`) を使用します。*

1. ローカル コンピューター上のフォルダーにコードを複製します。

    ```console
    git clone https://github.com/Azure-Samples/simple-feed-reader/
    ```

2. 作業フォルダーを、作成された *simple-feed-reader* フォルダーに変更します。

    ```console
    cd .\simple-feed-reader\SimpleFeedReader
    ```

3. パッケージを復元し、ソリューションをビルドします。

    ```dotnetcli
    dotnet build
    ```

4. アプリを実行します。

    ```dotnetcli
    dotnet run
    ```

    ![dotnet run コマンドが成功する](./media/deploying-to-app-service/dotnet-run.png)

5. ブラウザーを開き、 `http://localhost:5000` に移動します。 アプリでは、配信フィード URL を入力するか貼り付けて、ニュース項目の一覧を表示することができます。

     ![RSS フィードのコンテンツが表示されているアプリ](./media/deploying-to-app-service/app-in-browser.png)

6. アプリが正常に動作していることを確認したら、コマンド シェルで **Ctrl** + **C** キーを押してシャットダウンします。

## <a name="create-the-azure-app-service-web-app"></a>Azure App Service Web アプリを作成する

アプリをデプロイするには、App Service [Web アプリ](/azure/app-service/app-service-web-overview)を作成する必要があります。 Web アプリを作成した後、Git を使用してローカル コンピューターからそれをデプロイします。

1. [Azure Cloud Shell](https://shell.azure.com/bash) にサインインします。 メモ:初めてサインインすると、構成ファイル用のストレージ アカウントの作成を求めるメッセージが Cloud Shell に表示されます。 既定値をそのまま使用するか、一意の名前を指定します。

2. 以下の手順では Cloud Shell を使用します。

    a. Web アプリの名前を格納する変数を宣言します。 既定の URL で使用するには、名前が一意である必要があります。 `$RANDOM` Bash 関数を使用して名前を作成すると、一意性が保証され、`webappname99999` の形式になります。

    ```console
    webappname=mywebapp$RANDOM
    ```

    b. リソース グループを作成します。 リソース グループを使用すると、管理対象の Azure リソースをグループとして集約させることができます。

    ```azurecli
    az group create --location centralus --name AzureTutorial
    ```

    `az` コマンドでは [Azure CLI](/cli/azure/) が呼び出されます。 CLI はローカル環境でも実行できますが、Cloud Shell で使用すると、時間と構成が節約されます。

    c. S1 レベルで App Service プランを作成します。 App Service プランは、同じ価格レベルを共有する Web アプリをグループ化したものです。 S1 レベルは無料ではありませんが、ステージング スロット機能に必要です。

    ```azurecli
    az appservice plan create --name $webappname --resource-group AzureTutorial --sku S1
    ```

    d. App Service プランを使用して、同じリソース グループ内に Web アプリ リソースを作成します。

    ```azurecli
    az webapp create --name $webappname --resource-group AzureTutorial --plan $webappname
    ```

    e. デプロイの資格情報を設定します。 これらのデプロイ資格情報は、サブスクリプション内のすべての Web アプリに適用されます。 ユーザー名では特殊文字を使用しないでください。

    ```azurecli
    az webapp deployment user set --user-name REPLACE_WITH_USER_NAME --password REPLACE_WITH_PASSWORD
    ```

    f. ローカル環境の Git からデプロイを受け入れるように Web アプリを構成し、"*Git デプロイ URL*" を表示します。 **後で参照するので、この URL を記録しておきます**。

    ```azurecli
    echo Git deployment URL: $(az webapp deployment source config-local-git --name $webappname --resource-group AzureTutorial --query url --output tsv)
    ```

    g. "*Web アプリの URL*" を表示します。 この URL を参照すると、空の Web アプリが表示されます。 **後で参照するので、この URL を記録しておきます**。

    ```console
    echo Web app URL: http://$webappname.azurewebsites.net
    ```

3. ローカル コンピューターでコマンド シェルを使用して、Web アプリのプロジェクト フォルダー (`.\simple-feed-reader\SimpleFeedReader` など) に移動します。 次のコマンドを実行し、デプロイ URL にプッシュするように Git を設定します。

    a. ローカル リポジトリにリモート URL を追加します。

    ```console
    git remote add azure-prod GIT_DEPLOYMENT_URL
    ```

    b. ローカルの既定のブランチ (*main*) を *azure-prod* リモートの既定のブランチ (*main*) にプッシュします。

    ```console
    git push azure-prod main
    ```

    前に作成したデプロイ資格情報の入力を求められます。 コマンド シェルで出力を確認します。 Azure によって、ASP.NET Core アプリがリモートでビルドされます。

4. ブラウザーで "*Web アプリ URL*" に移動し、アプリがビルドおよびデプロイされたことを確認します。 `git commit` を使用して、追加の変更をローカル Git リポジトリにコミットできます。 前の `git push` コマンドを使用すると、これらの変更が Azure にプッシュされます。

## <a name="deployment-with-visual-studio"></a>Visual Studio でのデプロイ

> *注:このセクションは Windows のみに適用されます。Linux と macOS のユーザーは、下のステップ 2 で説明されている変更を行う必要があります。ファイルを保存し、`git commit` を使用してローカル リポジトリに変更をコミットします。最後に、最初のセクションと同様に、`git push` を使用して変更をプッシュします。*

アプリはコマンド シェルから既にデプロイされています。 Visual Studio の統合ツールを使用して、アプリに更新プログラムをデプロイしてみましょう。 Visual Studio の使い慣れた UI では、バックグラウンドで、コマンド ライン ツールと同じことが実行されています。

1. Visual Studio で *SimpleFeedReader.sln* を開きます。
2. ソリューション エクスプローラーで *Pages\Index.cshtml* を開きます。 `<h2>Simple Feed Reader</h2>` を `<h2>Simple Feed Reader - V2</h2>` に変更します。
3. **Ctrl** + **Shift** + **B** キーを押して、アプリをビルドします。
4. ソリューション エクスプローラーでプロジェクトを右クリックし、 **[発行]** をクリックします。

    ![右クリックの [発行] を示すスクリーンショット](./media/deploying-to-app-service/publish.png)
5. Visual Studio で新しい App Service リソースを作成できますが、この更新プログラムは既存のデプロイに対して発行されます。 **[発行先を選択]** ダイアログで、左側の一覧から **[App Service]** を選択し、 **[既存のものを選択]** を選択します。 **[発行]** をクリックします。
6. **[App Service]** ダイアログで、Azure サブスクリプションの作成に使用した Microsoft アカウントまたは組織アカウントが右上に表示されていることを確認します。 そうでない場合は、ドロップダウンをクリックして追加します。
7. 正しい Azure **サブスクリプション** が選択されていることを確認します。 **[表示]** で、 **[リソース グループ]** を選択します。 **AzureTutorial** リソース グループを展開し、既存の Web アプリを選択します。 **[OK]** をクリックします。

    ![App Service への発行ダイアログを示すスクリーンショット](./media/deploying-to-app-service/publish-dialog.png)

Visual Studio によってアプリがビルドされ、Azure にデプロイされます。 Web アプリの URL を参照します。 `<h2>` 要素の変更が有効であることを確認します。

![タイトルが変更されたアプリ](./media/deploying-to-app-service/app-v2.png)

## <a name="deployment-slots"></a>デプロイ スロット

デプロイ スロットは、運用環境で実行されているアプリに影響を与えることなく、変更のステージングがサポートされます。 アプリのステージング バージョンが品質保証チームによって検証されると、運用スロットとステージング スロットを入れ替えることができます。 ステージング中のアプリは、この方法で運用環境にレベル上げされます。 次の手順では、ステージング スロットを作成し、変更をそこにデプロイして、検証後にステージング スロットと運用環境を入れ替えます。

1. まだサインインしていない場合は、[Azure Cloud Shell](https://shell.azure.com/bash) にサインインします。
2. ステージング スロットを作成します。

    a. *staging* という名前でデプロイ スロットを作成します。

    ```azurecli
    az webapp deployment slot create --name $webappname --resource-group AzureTutorial --slot staging
    ```

    b. ローカル環境の Git からのデプロイを使用するようにステージング スロットを構成し、**ステージング** デプロイ URL を取得します。 **後で参照するので、この URL を記録しておきます**。

    ```azurecli
    echo Git deployment URL for staging: $(az webapp deployment source config-local-git --name $webappname --resource-group AzureTutorial --slot staging --query url --output tsv)
    ```

    c. ステージング スロットの URL を表示します。 URL を参照して、空のステージン グスロットを表示します。 **後で参照するので、この URL を記録しておきます**。

    ```console
    echo Staging web app URL: http://$webappname-staging.azurewebsites.net
    ```

3. テキスト エディターまたは Visual Studio で、`<h2>` 要素が `<h2>Simple Feed Reader - V3</h2>` を読み取り、ファイルを保存するように、*Pages/Index.cshtml* をもう一度変更します。

4. Visual Studio の *[チーム エクスプローラー]* タブの **[変更]** ページを使用するか、ローカル コンピューターのコマンド シェルを使用して次のように入力し、ファイルをローカル Git リポジトリにコミットします。

    ```console
    git commit -a -m "upgraded to V3"
    ```

5. ローカル コンピューターのコマンド シェルを使用して、ステージング デプロイ URL を Git リモートとして追加し、コミットされた変更をプッシュします。

    a. ステージング用のリモート URL をローカル Git リポジトリに追加します。

    ```console
    git remote add azure-staging <Git_staging_deployment_URL>
    ```

    b. ローカルの既定のブランチ (*main*) を *azure-staging* リモートの既定のブランチ (*main*) にプッシュします。

    ```console
    git push azure-staging main
    ```

    Azure によってアプリがビルドされてデプロイされるまで待ちます。

6. V3 がステージング スロットにデプロイされたことを確認するため、2 つのブラウザー ウィンドウを開きます。 1 つのウィンドウで、元の Web アプリの URL に移動します。 もう 1 つのウィンドウで、ステージング Web アプリの URL に移動します。 運用 URL では、アプリの V2 が提供されています。 ステージング URL では、アプリの V3 が提供されています。

    ![ブラウザー ウィンドウの比較のスクリーンショット](./media/deploying-to-app-service/ready-to-swap.png)

7. Cloud Shell で、検証済みのウォームアップされたステージング スロットを運用環境にスワップします。

    ```azurecli
    az webapp deployment slot swap --name $webappname --resource-group AzureTutorial --slot staging
    ```

8. 2 つのブラウザー ウィンドウを更新して、スワップが発生したことを確認します。

    ![スワップ後のブラウザー ウィンドウの比較](./media/deploying-to-app-service/swapped.png)

## <a name="summary"></a>まとめ

このセクションでは、次のタスクを完了しました。

* サンプル アプリをダウンロードしてビルドしました。
* Azure Cloud Shell を使用して Azure App Service Web アプリを作成しました。
* Git を使用してサンプル アプリを Azure にデプロイしました。
* Visual Studio を使用して変更をアプリにデプロイしました。
* ステージング スロットを Web アプリに追加しました。
* 更新プログラムをステージング スロットにデプロイしました。
* ステージング スロットと運用スロットをスワップしました。

次のセクションでは、Azure Pipelines を使用して DevOps パイプラインを構築する方法について説明します。

## <a name="additional-reading"></a>その他の参考資料

* [Web Apps の概要](/azure/app-service/app-service-web-overview)
* [Azure App Service で .NET Core および SQL Database のアプリを作成する](/azure/app-service/app-service-web-tutorial-dotnetcore-sqldb)
* [Azure App Service のデプロイ資格情報の構成](/azure/app-service/app-service-deployment-credentials)
* [Azure App Service でステージング環境を設定する](/azure/app-service/web-sites-staged-publishing)
