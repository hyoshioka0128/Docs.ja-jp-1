---
title: Visual Studio を使用して Azure API Management に ASP.NET Core Web API を発行する
author: codemillmatt
description: Visual Studio を使用して Azure API Management に ASP.NET Core Web API を発行する方法を説明します。
ms.author: masoucou
ms.custom: devx-track-csharp, mvc
ms.date: 11/22/2020
uid: tutorials/publish-to-azure-api-management-using-vs
ms.openlocfilehash: ddff54bbd146c98cf83a865910401df26e7ac4ec
ms.sourcegitcommit: 7e394a8527c9818caebb940f692ae4fcf2f1b277
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/31/2021
ms.locfileid: "99217584"
---
# <a name="publish-an-aspnet-core-web-api-to-azure-api-management-with-visual-studio"></a>Visual Studio を使用して Azure API Management に ASP.NET Core Web API を発行する

作成者: [Matt Soucoup](https://twitter.com/codemillmatt)

## <a name="set-up"></a>設定

- Azure アカウントをお持ちでない場合は、[Azure 無料アカウント](https://azure.microsoft.com/free/dotnet/)を作成します。
- まだ行っていない場合は、[新しい Azure API Management インスタンスを作成します](/azure/api-management/get-started-create-service-instance)。
- [Web API アプリ プロジェクトを作成します](xref:tutorials/first-web-api#create-a-web-project)。

## <a name="configure-the-app"></a>Configure the app

Swagger 定義を ASP.NET Core Web API に追加すると、Azure API Management でアプリの API 定義を読み取ることができます。 次の手順のようにします。

### <a name="add-swagger"></a>Swagger を追加する

1. [Swashbuckle.AspNetCore](https://www.nuget.org/packages/Swashbuckle.AspNetCore) NuGet パッケージを ASP.NET Core Web API プロジェクトに追加します。

    ![NuGet ダイアログの構成のスクリーンショット](publish-to-azure-api-management-using-vs/_static/configure_nuget.png)

1. 次の行を `Startup.ConfigureServices` メソッドに追加します。
    
    ```csharp
    services.AddSwaggerGen();
    ```
    
1. 次の行を `Startup.Configure` メソッドに追加します。

    ```csharp
    app.UseSwagger();
    ```

### <a name="change-the-api-routing"></a>API ルーティングを変更する

次に、`WeatherForecastController`の `Get` アクションにアクセスするために必要な URL 構造を変更します。 次の手順のようにします。

1. *WeatherForecastController.cs* ファイルを開きます。
1. クラス レベルの `[Route("[controller]")]` 属性を削除します。 クラスの定義は次のようになります。

    ```csharp
    [ApiController]
    public class WeatherForecastController : ControllerBase
    ```

1. `Get()` アクションに `[Route("/")]` 属性を追加します。 関数の定義は次のようになります。

    ```csharp
    [HttpGet]
    [Route("/")]
    public IEnumerable<WeatherForecast> Get()
    ```

## <a name="publish-the-web-api-to-azure-app-service"></a>Azure App Service に Web API を発行する

ASP.NET Core Web API を Azure API Management に発行するには、次の手順のようにします。

1. API アプリを Azure App Service に発行します。
1. Azure API Management サービス インスタンスに空の API を追加します。
1. ASP.NET Core Web API アプリを Azure API Management サービス インスタンスに発行します。

### <a name="publish-the-api-app-to-azure-app-service"></a>API アプリを Azure App Service に発行する

ASP.NET Core Web API を Azure API Management に発行するには、次の手順のようにします。

1. **ソリューション エクスプローラー** で、プロジェクトを右クリックして、 **[発行]** を選択します。

    ![[発行] リンクが選択された状態でコンテキスト メニューが開きます](publish-to-azure-api-management-using-vs/_static/publish_menu.png)

1. **[発行]** ダイアログで **[Azure]** を選択し、 **[次へ]** ボタンを選択します。

    ![[発行] ダイアログ](publish-to-azure-api-management-using-vs/_static/publish_dialog.png)

1. **[Azure App Service (Windows)]** を選択し、 **[次へ]** ボタンを選択します。

    ![[発行] ダイアログ: App Service を選択する](publish-to-azure-api-management-using-vs/_static/publish_dialog_app_svc.png)

1. **[新しい Azure App Service の作成]** を選択します。

    ![[発行] ダイアログ: Azure サービス インスタンスを選択](publish-to-azure-api-management-using-vs/_static/publish_dialog_create_new_app_svc.png)

    **[App Service の作成]** ダイアログが表示されます。 **[アプリ名]** 、 **[リソース グループ]** 、 **[App Service プラン]** の各入力フィールドに値が設定されます。 これらの名前を保持することも、変更することもできます。
1. **[作成]** ボタンを選択します。

    ![[App Service の作成] ダイアログ](publish-to-azure-api-management-using-vs/_static/publish_dialog_app_svc_attributes.png)

作成が完了すると、ダイアログが自動的に閉じられ、 **[発行]** ダイアログに再度フォーカスが移ります。 作成されたインスタンスが自動的に選択されます。

![[発行] ダイアログ: App Service インスタンスを選択](publish-to-azure-api-management-using-vs/_static/publish_dialog_app_svc_created.png)

この時点で、Azure API Management サービスに API を追加する必要があります。 次のタスクを行っている間は、Visual Studio を開いたままにしておきます。

### <a name="add-an-api-to-azure-api-management"></a>Azure API Management に API を追加する

1. 前に作成した API Management サービス インスタンスを Azure portal で開きます。 **[API]** ブレードを選択します。

  ![API Management サービス インスタンスから選択した [API] ブレード](publish-to-azure-api-management-using-vs/_static/portal_api_overview.png)

1. **Echo API** の横にある 3 つのドットを選択し、ポップアップ メニューから **[削除]** を選択して削除します。

  ![API Management サービス インスタンスからの echo API の削除](publish-to-azure-api-management-using-vs/_static/portal_delete_echo.png)

1. **[Add a new API]\(新しい API の追加\)** パネルで、 **[空の API]** タイルを選択します。

  ![[空の API] タイルが強調表示されている画面](publish-to-azure-api-management-using-vs/_static/portal_api_create_blank.png)

1. 表示される **[空の API の作成]** ダイアログで、次の値を入力します。    

    - **表示名**: *WeatherForecasts*
    - **名前**: *weatherforecasts*
    - **API URL サフィックス**: *v1*
    - **[Web サービス URL]** フィールドは空のままにします。

1. **[作成]** ボタンを選択します。

    ![完成した [空の API の作成] ダイアログのスクリーンショット](publish-to-azure-api-management-using-vs/_static/portal_api_blank_complete.png)

空の API が作成されます。

![API Management ポータルでの空の API のスクリーンショット](publish-to-azure-api-management-using-vs/_static/portal_api_blank_created_empty.png)

### <a name="publish-the-aspnet-core-web-api-to-azure-api-management"></a>Azure API Management に ASP.NET Core Web API を発行する

1. Visual Studio に戻ります。 前にそのままにしておいた **[発行]** ダイアログ ボックスがまだ開いているはずです。
1. 新しく発行した Azure App Service を選択して強調表示にします。
1. **[次へ]** ボタンを選択します。

    ![App Service が選択されている [発行] ダイアログのスクリーンショット](publish-to-azure-api-management-using-vs/_static/publish_dialog_app_svc_created.png)

1. ダイアログに、前に作成した Azure API Management サービスが表示されるようになります。 それと *[APIs]* フォルダーを展開して、作成された空の API を表示します。
1. 空の API の名前を選択して、 **[完了]** ボタンを選択します。

    ![[発行] ダイアログで選択された、新しく作成された Azure API Management の空の API のスクリーンショット](publish-to-azure-api-management-using-vs/_static/publish_dialog_api_selected.png)

1. ダイアログが閉じ、発行に関する情報が含まれる概要画面が表示されます。 **[発行]** ボタンを選びます。

    ![発行プロファイルが表示されている Visual Studio のスクリーンショット](publish-to-azure-api-management-using-vs/_static/vs_publish_profile.png)

    Web API は、Azure App Service と Azure API Management の両方に発行されます。 新しいブラウザー ウィンドウが表示され、Azure App Service で実行されている API が示されます。 そのウィンドウは閉じてもかまいません。

1. Azure portal で Azure API Management インスタンスに戻ります。
1. ブラウザー ウィンドウを更新します。
1. 前の手順で作成した空の API を選択します。 内容が設定されているので、調べることができます。

    ![Azure API Management で設定された API のスクリーンショット](publish-to-azure-api-management-using-vs/_static/deployed_to_azure_api_mgmt.png)

### <a name="configure-the-published-api-name"></a>発行された API の名前を構成する

API の名前が、自分で付けたものとは異なることに注意してください。 発行された API は *WeatherAPI* という名前になっています。しかし、作成したときには *WeatherForecasts* という名前を設定しました。 名前を修正するには、次の手順のようにします。

![異なる名前が強調して示されている、設定された API のスクリーンショット](publish-to-azure-api-management-using-vs/_static/deployed_to_azure_api_mgmt_wrong_name.png)

1. `Startup.ConfigureServices` メソッドから次の行を削除します。
    
    ```csharp
    services.AddSwaggerGen();
    ```

1. `Startup.ConfigureServices` メソッドに次のコードを追加します。
    
    ```csharp
    services.AddSwaggerGen(config =>
    {
        config.SwaggerDoc("v1", new Microsoft.OpenApi.Models.OpenApiInfo
        {
            Title = "WeatherForecasts",
            Version = "v1"
        });
    });
    ```

1. 新しく作成された発行プロファイルを開きます。 **ソリューション エクスプローラー** の *Properties/PublishProfiles* フォルダー内にあります。

    ![発行プロファイル ファイルの場所が強調して示されているスクリーンショット](publish-to-azure-api-management-using-vs/_static/vs_publish_profile_highlighted.png)

1. `<OpenAPIDocumentName>` 要素の値を `v1` から `WeatherForecasts` に変更します。

    ![発行プロファイルに必要な変更のスクリーンショット](publish-to-azure-api-management-using-vs/_static/vs_publish_profile_changes.png)

1. ASP.NET Core Web API を再発行し、Azure portal で Azure API Management インスタンスを開きます。
1. ブラウザーでページを更新します。 API の名前が正しくなったことがわかります。

    ![完成した API のポータルでのスクリーンショット](publish-to-azure-api-management-using-vs/_static/portal_finish.png)

### <a name="verify-the-web-api-is-working"></a>Web API が動作していることを確認する

次の手順のようにして、Azure API Management にデプロイされた ASP.NET Core Web API を Azure portal からテストできます。

1. **[テスト]** タブを開きます。
1. **[/]** または **[GET]** 操作を選択します。
1. **[Send]** を選択します。

    ![テスト前のポータルのスクリーンショット](publish-to-azure-api-management-using-vs/_static/portal_pre_test.png)

成功した場合の応答は次のようになります。

![API Management からの正常な応答のスクリーンショット](publish-to-azure-api-management-using-vs/_static/portal_successful_response.png)

## <a name="clean-up"></a>クリーンアップ

アプリのテストが完了したら、[Azure portal](https://portal.azure.com/) に移動して、アプリを削除します。

1. **[リソース グループ]** を選択し、作成したリソース グループを選択します。

    ![Azure Portal: サイドバー メニューの [リソース グループ]](publish-to-azure-api-management-using-vs/_static/portalrg.png)

1. **[リソース グループ]** ページで、 **[削除]** を選択します。

    ![Azure Portal:[リソース グループ] ページ](publish-to-azure-api-management-using-vs/_static/rgd.png)

1. リソース グループ名を入力し、 **[削除]** を選択します。 このチュートリアルで作成されたアプリとその他すべてのリソースが Azure から削除されます。

## <a name="next-steps"></a>次の手順

<xref:host-and-deploy/azure-apps/azure-continuous-deployment>

## <a name="additional-resources"></a>その他の技術情報

- [Azure API Management](/azure/api-management/api-management-key-concepts)
- [Azure App Service](/azure/app-service/app-service-web-overview)
