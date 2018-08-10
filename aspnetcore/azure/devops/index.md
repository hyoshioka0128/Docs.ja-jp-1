---
title: ASP.NET Core および Azure を使用した DevOps
author: CamSoper
description: Azure でホストされる ASP.NET Core アプリの DevOps パイプラインの構築に関するエンドツーエンドのガイダンスを提供するガイド。
ms.author: casoper
ms.date: 08/07/2018
uid: azure/devops/index
ms.openlocfilehash: 09ca835e908e81c6f38f9430fb40638ba6dc3350
ms.sourcegitcommit: 29dfe436f54a27fbb4f6494bc639d16c75001fab
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/09/2018
ms.locfileid: "39722539"
---
# <a name="devops-with-aspnet-core-and-azure"></a>ASP.NET Core および Azure を使用した DevOps

.NET 向けの Azure 開発ライフサイクル ガイドへようこそ。 このガイドでは .NET のツールとプロセスを使用する Azure に関する開発ライフサイクルの構築に関する基本概念を紹介します。 このガイドを完了すると、進化した DevOps ツールチェーンの利点が得られます。

## <a name="who-this-guide-is-for"></a>このガイドの対象ユーザー

熟練した ASP.NET 開発者 (200-300 レベル) である必要があります。 Azure に関する知識は必要ありません。それについてはこのガイドで説明します。 このガイドは、開発よりも操作に注力する DevOps エンジニアにも役立つ可能性があります。

このガイドは、Windows 開発者を対象としています。 ただし、.NET Core によって Linux と macOS が完全にサポートされています。 このガイドを Linux/macOS で使用する場合は、吹き出しで Linux/macOS との違いを確認してください。

## <a name="what-this-guide-doesnt-cover"></a>このガイドに含まれないもの

このガイドは、.NET 開発者向けのエンドツーエンドの継続的デプロイに焦点を当てています。 Azure のすべてを網羅するガイドではなく、Azure サービス向けの .NET API について幅広く取り上げるものでもありません。 特に焦点を当てるのは、継続的インテグレーション、デプロイ、監視、デバッグについてです。 このガイドの終盤には、次のステップとしてお勧めする内容を紹介します。 その内容には ASP.NET 開発者に役立つ Azure プラットフォーム サービスも含まれています。

## <a name="whats-in-this-guide"></a>このガイドの内容

### <a name="tools-and-downloadsxrefazuredevopstools-and-downloads"></a>[ツールとダウンロード](xref:azure/devops/tools-and-downloads)

このガイドで使用するツールを取得できる場所について説明します。

### <a name="deploy-to-app-servicexrefazuredevopsdeploy-to-app-service"></a>[App Service にデプロイする](xref:azure/devops/deploy-to-app-service)

Azure App Service に ASP.NET Core アプリをデプロイするためのさまざまな方法について説明します。

### <a name="continuous-integration-and-deploymentxrefazuredevopscicd"></a>[継続的インテグレーションと継続的デプロイ](xref:azure/devops/cicd)

GitHub、VSTS、Azure で自分の ASP.NET Core アプリにエンドツーエンドの継続的インテグレーションと継続的デプロイ ソリューションを構築します。

### <a name="monitor-and-debugxrefazuredevopsmonitor"></a>[監視とデバッグ](xref:azure/devops/monitor)

Azure のツールを使用して、アプリケーションの監視、トラブルシューティング、調整を行います。

### <a name="next-stepsxrefazuredevopsnext-steps"></a>[次のステップ](xref:azure/devops/next-steps)

Azure を学習する ASP.NET Core 開発者向けのその他のラーニング パス。

## <a name="acknowledgments"></a>謝辞

このガイドに有用な提案を行っていただいた .NET コミュニティのすべてのユーザーに感謝いたします。 この資料の最終レビューに参加していただいた、以下のコミュニティ メンバーの皆様には、特に感謝しております。

* [Sam Wronski](https://www.youtube.com/c/worldofzerodevelopment)
* [Jeffrey Palermo](https://twitter.com/jeffreypalermo)

## <a name="conclusion"></a>まとめ

このガイドは、ASP.NET Core および Azure App Service に構築された継続的インテグレーションの開発ライフサイクルの構築を準備するものです。

## <a name="additional-reading"></a>その他の参考資料

* [クラウド コンピューティングとは](https://azure.microsoft.com/overview/what-is-cloud-computing/)
* [クラウド コンピューティングの例](https://azure.microsoft.com/overview/examples-of-cloud-computing/)
* [IaaS とは](https://azure.microsoft.com/overview/what-is-iaas/)
* [PaaS とは](https://azure.microsoft.com/overview/what-is-paas/)