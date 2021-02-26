---
title: ASP.NET Core Blazor 用のトリマーを構成する
author: guardrex
description: Blazor アプリをビルドする際に中間言語 (IL) リンカー (トリマー) を制御する方法について説明します。
monikerRange: '>= aspnetcore-5.0'
ms.author: riande
ms.custom: mvc
ms.date: 02/08/2021
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
uid: blazor/host-and-deploy/configure-trimmer
ms.openlocfilehash: 41887638f13a08d375075e8377da19d1d0098c4b
ms.sourcegitcommit: ef8d8c79993a6608bf597ad036edcf30b231843f
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/09/2021
ms.locfileid: "99975217"
---
# <a name="configure-the-trimmer-for-aspnet-core-blazor"></a>ASP.NET Core Blazor 用のトリマーを構成する

Blazor WebAssembly では、発行された出力のサイズを縮小するために、[中間言語 (IL)](/dotnet/standard/managed-code#intermediate-language--execution) トリミングが実行されます。 既定では、トリミングはアプリの発行時に行われます。

トリミングは好ましくない効果を与える場合があります。 リフレクションを使用するアプリの場合、実行時にリフレクションに必要な型がトリマーでは判断できないことがしばしばあります。 リフレクションを使用するアプリをトリミングするには、アプリのコードと、アプリが依存するパッケージまたはフレームワークの両方で、リフレクションに必要な型をトリマーに通知する必要があります。 実行時にアプリの動的な動作に反応することもトリマーはできません。 展開後、トリミングされたアプリが確実に正しく機能するよう、発行された出力を開発中に頻繁にテストしてください。

トリマーを構成するには、.NET の基礎ドキュメントの[トリミング オプション](/dotnet/core/deploying/trimming-options)に関する記事を参照してください。次の項目に関するガイダンスが含まれています。

* プロジェクト ファイルの `<PublishTrimmed>` プロパティを使用し、アプリ全体のトリミングを無効にします。
* 使用されていない IL をトリマーによって破棄する積極度を制御します。
* 特定のアセンブリのトリミングをトリマーに禁止します。
* トリミング用の "ルート" アセンブリ。
* プロジェクト ファイルで `<SuppressTrimAnalysisWarnings>` プロパティを `false` に設定することで、リフレクションされた型の警告を表示します。
* シンボル トリミングとデバッガー サポートを制御します。
* フレームワーク ライブラリ機能をトリミングするためのトリマー機能を設定します。

## <a name="additional-resources"></a>その他のリソース

* [自己完結型の展開と実行可能ファイルのトリミング](/dotnet/core/deploying/trim-self-contained)
* <xref:blazor/webassembly-performance-best-practices#intermediate-language-il-trimming>
