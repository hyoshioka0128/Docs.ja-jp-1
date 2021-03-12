---
title: でのユーザーとグループの管理 SignalR
author: bradygaster
description: ASP.NET Core ユーザーとグループの管理の概要につい SignalR て説明します。
monikerRange: '>= aspnetcore-2.1'
ms.author: bradyg
ms.custom: mvc
ms.date: 05/17/2020
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
uid: signalr/groups
ms.openlocfilehash: 8f9e91f34386bb5b4cf79bee429a39ae27c03c5e
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/10/2021
ms.locfileid: "102589219"
---
# <a name="manage-users-and-groups-in-signalr"></a>でのユーザーとグループの管理 SignalR

[Brennan Conroy](https://github.com/BrennanConroy)

SignalR 特定のユーザーに関連付けられているすべての接続、および名前付きの接続グループにメッセージを送信できるようにします。

[サンプル コードを表示またはダウンロードする](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/signalr/groups/sample/) ([ダウンロード方法](xref:index#how-to-download-a-sample))

## <a name="users-in-signalr"></a>のユーザー SignalR

の1人のユーザー SignalR がアプリに対して複数の接続を持つことができます。 たとえば、ユーザーは自分のデスクトップや電話に接続することができます。 各デバイスには個別の接続があり SignalR ますが、これらはすべて同じユーザーに関連付けられています。 ユーザーにメッセージが送信されると、そのユーザーに関連付けられているすべての接続がメッセージを受信します。 接続のユーザー識別子には、ハブのプロパティからアクセスでき `Context.UserIdentifier` ます。

既定では、は、 SignalR `ClaimTypes.NameIdentifier` `ClaimsPrincipal` 接続に関連付けられているのをユーザー識別子として使用します。 この動作をカスタマイズするには、「 [要求を使用して id 処理をカスタマイズ](xref:signalr/authn-and-authz#use-claims-to-customize-identity-handling)する」を参照してください。

`User`次の例に示すように、ハブメソッドでユーザー識別子を関数に渡すことによって、特定のユーザーにメッセージを送信します。

> [!NOTE]
> ユーザー識別子では、大文字と小文字が区別されます。

[!code-csharp[Configure service](groups/sample/Hubs/ChatHub.cs?range=29-32)]

## <a name="groups-in-signalr"></a>のグループ SignalR

グループとは、名前に関連付けられている接続のコレクションです。 メッセージは、グループ内のすべての接続に送信できます。 グループはアプリケーションによって管理されるため、接続または複数の接続に送信する方法としては、グループをお勧めします。 接続は、複数のグループのメンバーになることができます。 グループは、各部屋をグループとして表すことができるチャットアプリケーションのようなものに最適です。 接続は、メソッドとメソッドを使用してグループに追加されるか、グループから削除され `AddToGroupAsync` `RemoveFromGroupAsync` ます。

[!code-csharp[Hub methods](groups/sample/Hubs/ChatHub.cs?range=15-27)]

接続を再接続しても、グループのメンバーシップは保持されません。 再確立された場合、接続はグループに再度参加する必要があります。 アプリケーションが複数のサーバーにスケーリングされている場合、この情報は利用できないため、グループのメンバーをカウントすることはできません。

グループの使用中にリソースへのアクセスを保護するには、ASP.NET Core で [認証と承認](xref:signalr/authn-and-authz) の機能を使用します。 そのグループに対して資格情報が有効な場合にのみ、ユーザーがグループに追加されると、そのグループに送信されたメッセージは、承認されたユーザーのみに送られます。 ただし、グループはセキュリティ機能ではありません。 認証要求には、有効期限や失効など、グループにはない機能があります。 グループにアクセスするためのユーザーのアクセス許可が取り消された場合、アプリはユーザーをグループから明示的に削除する必要があります。

> [!NOTE]
> グループ名は大文字と小文字が区別されます。

## <a name="related-resources"></a>関連リソース

* [開始するには](xref:tutorials/signalr)
* [ハブ](xref:signalr/hubs)
* [Azure に発行する](xref:signalr/publish-to-azure-web-app)
