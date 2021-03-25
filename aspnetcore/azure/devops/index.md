---
title: ASP.NET Core および Azure を使用した DevOps
author: CamSoper
description: Azure でホストされる ASP.NET Core アプリの DevOps パイプラインの構築に関するエンドツーエンドのガイダンスを提供するガイド。
ms.author: casoper
ms.date: 08/07/2018
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
uid: azure/devops/index
ms.openlocfilehash: c49ecde1ab93f37a80ff604c7be00fa309d55ea7
ms.sourcegitcommit: b81327f1a62e9857d9e51fb34775f752261a88ae
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/25/2021
ms.locfileid: "105050959"
---
# <a name="devops-with-aspnet-core-and-azure"></a>ASP.NET Core および Azure を使用した DevOps

[![カバーの画像](./media/cover-large.png)](https://aka.ms/devopsbook)

[Cam Soper](https://twitter.com/camsoper) および [Scott Addie](https://twitter.com/scottaddie) 著

このガイドは、[ダウンロード可能 PDF 電子ブック](https://github.com/dotnet-architecture/eBooks/blob/master/current/devops-aspnet-core/DevOps-with-ASP.NET-Core-and-Azure.pdf)として入手できます。

## <a name="welcome"></a>ようこそ 

.NET 向けの Azure 開発ライフサイクル ガイドへようこそ。 このガイドでは .NET のツールとプロセスを使用する Azure に関する開発ライフサイクルの構築に関する基本概念を紹介します。 このガイドを完了すると、進化した DevOps ツールチェーンの利点が得られます。

## <a name="who-this-guide-is-for"></a>このガイドの対象ユーザー

熟練した ASP.NET Core 開発者 (200-300 レベル) である必要があります。 Azure に関する知識は必要ありません。それについてはこのガイドで説明します。 このガイドは、開発よりも操作に注力する DevOps エンジニアにも役立つ可能性があります。

このガイドは、Windows 開発者を対象としています。 ただし、.NET Core によって Linux と macOS が完全にサポートされています。 このガイドを Linux/macOS で使用する場合は、吹き出しで Linux/macOS との違いを確認してください。

## <a name="what-this-guide-doesnt-cover"></a>このガイドに含まれないもの

このガイドは、.NET 開発者向けのエンドツーエンドの継続的デプロイに焦点を当てています。 Azure のすべてを網羅するガイドではなく、Azure サービス向けの .NET API について幅広く取り上げるものでもありません。 特に焦点を当てるのは、継続的インテグレーション、デプロイ、監視、デバッグについてです。 このガイドの終盤には、次のステップとしてお勧めする内容を紹介します。 その内容には ASP.NET Core 開発者に役立つ Azure プラットフォーム サービスも含まれています。

## <a name="whats-in-this-guide"></a>このガイドの内容

### <a name="tools-and-downloads"></a>[ツールとダウンロード](xref:azure/devops/tools-and-downloads)

このガイドで使用するツールを取得できる場所について説明します。

### <a name="deploy-to-app-service"></a>[App Service にデプロイする](xref:azure/devops/deploy-to-app-service)

Azure App Service に ASP.NET Core アプリをデプロイするためのさまざまな方法について説明します。

### <a name="continuous-integration-and-deployment"></a>[継続的インテグレーションと継続的デプロイ](xref:azure/devops/cicd)

GitHub、Azure DevOps Services、Azure で自分の ASP.NET Core アプリにエンドツーエンドの継続的インテグレーションと継続的デプロイ ソリューションを構築します。

### <a name="monitor-and-debug"></a>[監視とデバッグ](xref:azure/devops/monitor)

Azure のツールを使用して、アプリケーションの監視、トラブルシューティング、調整を行います。

### <a name="next-steps"></a>[次の手順](xref:azure/devops/next-steps)

Azure を学習する ASP.NET Core 開発者向けのその他のラーニング パス。

## <a name="additional-introductory-reading"></a>その他の入門資料

クラウド コンピューティングを初めて使用する場合は、基礎について次の記事を参照してください。

* [クラウド コンピューティングとは](https://azure.microsoft.com/overview/what-is-cloud-computing/)
* [クラウド コンピューティングの例](https://azure.microsoft.com/overview/examples-of-cloud-computing/)
* [IaaS とは](https://azure.microsoft.com/overview/what-is-iaas/)
* [PaaS とは](https://azure.microsoft.com/overview/what-is-paas/)
