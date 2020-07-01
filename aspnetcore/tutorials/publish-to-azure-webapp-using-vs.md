---
title: author: description: ms.author: ms.custom: ms.date: no-loc:
- 'Blazor'
- 'Identity'
- 'Let's Encrypt'
- 'Razor'
- 'SignalR' uid: 

---
# <a name="publish-an-aspnet-core-app-to-azure-with-visual-studio"></a>Visual Studio を使用して Azure に ASP.NET Core アプリを発行する

作成者: [Rick Anderson](https://twitter.com/RickAndMSFT)
::: moniker range=">= aspnetcore-3.0"

[!INCLUDE [Azure App Service Preview Notice](../includes/azure-apps-preview-notice.md)]

::: moniker-end


macOS で作業している場合は、「[Visual Studio for Mac を使用して Azure App Service に Web アプリを発行する](https://docs.microsoft.com/visualstudio/mac/publish-app-svc?view=vsmac-2019)」を参照してください。

App Service の配置に関する問題を解決するには、「<xref:test/troubleshoot-azure-iis>」を参照してください。

## <a name="set-up"></a>設定

* Azure アカウントをお持ちでない場合は、[Azure 無料アカウント](https://azure.microsoft.com/free/dotnet/)を作成します。 

## <a name="create-a-web-app"></a>Web アプリの作成

Visual Studio のスタート ページで、 **[ファイル]、[新規作成]、[プロジェクト]** の順に選択します。

![[ファイル] メニュー](publish-to-azure-webapp-using-vs/_static/file_new_project.png)

**[新しいプロジェクト]** ダイアログで次のように設定します。

* **[ASP.NET Core Web アプリケーション]** を選択します。
* **[次へ]** を選択します。

![[新しいプロジェクト] ダイアログ](publish-to-azure-webapp-using-vs/_static/new_prj.png)

**[新しい ASP.NET Core Web アプリケーション]** ダイアログで次の手順を実行します。

* **[Web アプリケーション]** を選択します。
* [認証] で **[変更]** を選択します。

![[新しい ASP.NET Core Web アプリケーション] ダイアログ](publish-to-azure-webapp-using-vs/_static/new_prj_2.png)

**[認証の変更]** ダイアログが表示されます。 

* **[個人のユーザー アカウント]** を選択します。
* **[OK]** を選択して **[新しい ASP.NET Core Web アプリケーション]** に戻り、 **[作成]** を選択します。

![新しい ASP.NET Core Web 認証ダイアログ](publish-to-azure-webapp-using-vs/_static/new_prj_auth.png) 

Visual Studio によってソリューションが作成されます。

## <a name="run-the-app"></a>アプリを実行する

* Ctrl キーを押しながら F5 キーを押してプロジェクトを実行します。
* **[プライバシー]** リンクをテストします。

![Microsoft Edge で開いているローカルホストの Web アプリケーション](publish-to-azure-webapp-using-vs/_static/show.png)

### <a name="register-a-user"></a>ユーザーを登録する

* **[登録]** を選択して、新しいユーザーを登録します。 架空の電子メール アドレスを使用できます。 送信すると、ページに次のエラーが表示されます。

    *A database operation failed while processing the request.\(要求の処理中にデータベースの操作に失敗しました。\)Applying existing migrations for Application DB context may resolve this issue. \(アプリケーション DB コンテキストの既存の移行を適用すると問題が解決する場合があります。\)*
* **[Apply Migrations]\(移行を適用する\)** を選択し、移行が完了したら、ページを更新します。

![要求の処理中にデータベースの操作に失敗しました。 Applying existing migrations for Application DB context may resolve this issue. /(アプリケーション DB コンテキスト用の既存の以降を適用すると問題が解決する場合があります。/)](publish-to-azure-webapp-using-vs/_static/mig.png)

アプリに、新しいユーザーの登録に使用した電子メールと **[ログアウト]** リンクが表示されます。

![Microsoft Edge で開いている Web アプリケーション。 [Register]\(登録\) リンクは Hello user1@example.com! というテキストで置き換えられます。](publish-to-azure-webapp-using-vs/_static/hello.png)

## <a name="deploy-the-app-to-azure"></a>Azure にアプリを配置する

ソリューション エクスプローラーでプロジェクトを右クリックし、 **[発行]** を選択します。

![[発行] リンクが選択された状態でコンテキスト メニューが開きます](publish-to-azure-webapp-using-vs/_static/pub.png)

**[発行]** ダイアログで、次の操作を行います。

* **[Azure]** を選択します。
* **[次へ]** を選択します。

![[発行] ダイアログ](publish-to-azure-webapp-using-vs/_static/maas1.png)

**[発行]** ダイアログで、次の操作を行います。

* **[Azure App Service (Linux)]** を選択します。
* **[次へ]** を選択します。

![[発行] ダイアログ: Azure サービスを選択](publish-to-azure-webapp-using-vs/_static/maas2.png)

**[発行]** ダイアログで、 **[新しい Azure App Service の作成...]** を選びます。

![[発行] ダイアログ: Azure サービス インスタンスを選択](publish-to-azure-webapp-using-vs/_static/maas3.png)

**[App Service の作成]** ダイアログが表示されます。

* **[アプリ名]** 、 **[リソース グループ]** 、 **[App Service プラン]** の各入力フィールドに値が設定されます。 これらの名前を保持することも、変更することもできます。
* **[作成]** を選択します。

![[App Service の作成] ダイアログ](publish-to-azure-webapp-using-vs/_static/newrg1.png)

作成が完了すると、ダイアログが自動的に閉じられ、 **[発行]** ダイアログに再度フォーカスが移ります。

* 作成したばかりの新しいインスタンスが自動的に選択されます。
* **[完了]** を選択します。

![[発行] ダイアログ: App Service インスタンスを選択](publish-to-azure-webapp-using-vs/_static/select_as.png)

次に、**発行プロファイルの概要**ページが表示されます。 Visual Studio によって、このアプリケーションには SQL Server データベースが必要であることが検出されているため、構成するように求められています。 **[構成]** をクリックします。

![発行プロファイルの概要ページ: SQL Server 依存関係の構成](publish-to-azure-webapp-using-vs/_static/sql.png)

**[依存関係の構成]** ダイアログが表示されます。

* **[Azure SQL Database]** を選択します。
* **[次へ]** を選択します。

![SQL Server の依存関係の構成ダイアログ](publish-to-azure-webapp-using-vs/_static/sql1.png)

**[Azure SQL Database の構成]** ダイアログで **[SQL Database の作成]** を選択します。

![[Azure SQL Database の構成] ダイアログ](publish-to-azure-webapp-using-vs/_static/sql2.png)

**Azure SQL Database の作成**が表示されます。

* **[データベース名]** 、 **[リソース グループ]** 、 **[データベース サーバー]** 、 **[App Service プラン]** の各入力フィールドに値が設定されます。 これらの値を保持することも、変更することもできます。
* 選択した **[データベース サーバー]** の **[データベース管理者のユーザー名]** と **[データベース管理者のパスワード]** を入力します (注: 使用するアカウントには、新しい Azure SQL データベースを作成するために必要なアクセス許可が必要です)。
* **[作成]** を選択します。

![新しい [Azure SQL Database] ダイアログ](publish-to-azure-webapp-using-vs/_static/sql_create.png)

作成が完了すると、ダイアログが自動的に閉じられ、 **[Azure SQL Database の構成]** ダイアログに再度フォーカスが移ります。

* 作成したばかりの新しいインスタンスが自動的に選択されます。
* **[次へ]** を選択します。

![[Azure SQL Database の構成] ダイアログ](publish-to-azure-webapp-using-vs/_static/sql_select.png)

**[Azure SQL Database の構成]** ダイアログの次の手順で、以下を実行します。

* **[データベース接続のユーザー名]** と **[データベース管理者のパスワード]** フィールドを入力します。 これらは、アプリケーションが実行時にデータベースに接続するために使用する詳細です。 ベスト プラクティスとして、前の手順で使用した管理者ユーザー名とパスワードと同じ詳細を使用しないようにすることをお勧めします。
* **[完了]** を選択します。

![[Azure SQL Database の構成] ダイアログ、接続文字列の詳細](publish-to-azure-webapp-using-vs/_static/sql_connection.png)

**発行プロファイルの概要**ページで **[設定]** を選択します。

![発行プロファイルの概要ページ: 設定の編集](publish-to-azure-webapp-using-vs/_static/pp_configured.png)

**[発行]** ダイアログの **[設定]** ページで次の手順を実行します。

* **[データベース]** を展開し、 **[この接続文字列を実行時に使用する]** をオンにします。
* **[Entity Framework の移行]** を展開し、 **[発行時にこの移行を適用する]** をオンにします。

* **[保存]** を選択します。 Visual Studio が **[発行]** ダイアログに戻ります。 

![[発行] ダイアログ:[設定] パネル](publish-to-azure-webapp-using-vs/_static/pp_settings.png)

**[発行]** をクリックします。 Visual Studio が Azure にアプリを発行します。 デプロイが完了すると、ブラウザーでアプリが開きます。

![[発行] ダイアログ:[設定] パネル](publish-to-azure-webapp-using-vs/_static/pp_publish.png)

### <a name="update-the-app"></a>アプリを更新する

* *Pages/Index.cshtml* Razor ページを編集し、その内容を変更します。 たとえば、"Hello ASP.NET Core!" と表示されるように段落を修正できます。

    [!code-html[Index](publish-to-azure-webapp-using-vs/sample/index.cshtml?highlight=10&range=1-12)]

* **発行プロファイルの概要**ページから **[発行]** を選択します。

![発行プロファイルの概要ページ](publish-to-azure-webapp-using-vs/_static/pp_publish.png)

* アプリが発行されたら、Azure に変更内容が反映されていることを確認します。

![タスクの完了を確認します。](publish-to-azure-webapp-using-vs/_static/final.png)

### <a name="clean-up"></a>クリーンアップ

アプリのテストが完了したら、[Azure Portal](https://portal.azure.com/) に移動し、アプリを削除します。

* **[リソース グループ]** を選択し、作成したリソース グループを選択します。

![Azure Portal: サイドバー メニューの [リソース グループ]](publish-to-azure-webapp-using-vs/_static/portalrg.png)

* **[リソース グループ]** ページで、 **[削除]** を選択します。

![Azure Portal:[リソース グループ] ページ](publish-to-azure-webapp-using-vs/_static/rgd.png)

* リソース グループ名を入力し、 **[削除]** を選択します。 このチュートリアルで作成されたアプリとその他すべてのリソースが Azure から削除されます。

### <a name="next-steps"></a>次の手順

* <xref:host-and-deploy/azure-apps/azure-continuous-deployment>

## <a name="additional-resources"></a>その他の技術情報

* Visual Studio Code については、「[発行プロファイル](xref:host-and-deploy/visual-studio-publish-profiles#publish-profiles)」をご覧ください。
* [Azure App Service](/azure/app-service/app-service-web-overview)
* [Azure リソース グループ](/azure/azure-resource-manager/resource-group-overview#resource-groups)
* [Azure SQL Database](/azure/sql-database/)
* <xref:host-and-deploy/visual-studio-publish-profiles>
* <xref:test/troubleshoot-azure-iis>
