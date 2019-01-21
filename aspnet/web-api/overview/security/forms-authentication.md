---
uid: web-api/overview/security/forms-authentication
title: ASP.NET Web API でフォーム認証 |Microsoft Docs
author: MikeWasson
description: ASP.NET Web API でフォーム認証の使用について説明します。
ms.author: riande
ms.date: 12/12/2012
ms.assetid: 9f06c1f2-ffaa-4831-94a0-2e4a3befdf07
msc.legacyurl: /web-api/overview/security/forms-authentication
msc.type: authoredcontent
ms.openlocfilehash: 35d62a83382553085ed8a728dcdcdae0e93090b8
ms.sourcegitcommit: 45ac74e400f9f2b7dbded66297730f6f14a4eb25
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/16/2018
ms.locfileid: "41832419"
---
<a name="forms-authentication-in-aspnet-web-api"></a>ASP.NET Web API でフォーム認証
====================
作成者[Mike Wasson](https://github.com/MikeWasson)

フォーム認証では、HTML フォームを使用して、ユーザーの資格情報をサーバーに送信します。 インターネット標準ではありません。 ユーザーが HTML フォームを操作できるように、フォーム認証は web、web アプリケーションから呼び出される API の適切なのみ。

| 長所 | 短所 |
| --- | --- |
| 実装が容易: ASP.NET に組み込まれています。 -ASP.NET メンバーシップ プロバイダーは、簡単にユーザー アカウントの管理を使用します。 | -非標準の HTTP 認証機構です。標準の Authorization ヘッダーではなく、HTTP クッキーを使用します。 -クライアントのブラウザーが必要です。 -資格情報は、プレーン テキストとして送信されます。 クロスサイト リクエスト フォージェリ (CSRF); に対して脆弱CSRF 対策メジャーが必要です。 -Nonbrowser クライアントから使用するは困難です。 ログインには、ブラウザーが必要です。 のユーザー資格情報は、要求で送信されます。 -一部のユーザーは、cookie を無効にします。 |

について簡単に、ASP.NET フォーム認証は、次のように動作します。

1. クライアントは、認証が必要なリソースを要求します。
2. ユーザーが認証されていない場合、サーバーは HTTP 302 (検出) を返し、ログイン ページにリダイレクトします。
3. ユーザーは資格情報を入力し、フォームを送信します。
4. サーバーは、元の URI にリダイレクトするもう 1 つの HTTP 302 を返します。 この応答には、認証 cookie が含まれています。
5. クライアントは、もう一度リソースを要求します。 要求には、サーバー要求を許可するために、認証 cookie が含まれます。

![](forms-authentication/_static/image1.png)

詳細については、次を参照してください。 [、フォーム認証の概要。](../../../web-forms/overview/older-versions-security/introduction/an-overview-of-forms-authentication-cs.md)

## <a name="using-forms-authentication-with-web-api"></a>フォーム認証を使用して Web API

フォーム認証を使用するアプリケーションを作成するには、MVC 4 プロジェクト ウィザードで、「インターネット アプリケーション」テンプレートを選択します。 このテンプレートは、アカウント管理のための MVC コント ローラーを作成します。 ASP.NET の Fall 2012 更新プログラムで使用できる「シングル ページ アプリケーション」テンプレートを使用することもできます。

使用してアクセスを制限する、web API コント ローラーで、`[Authorize]`属性」の説明に従って[[Authorize] 属性を使用して](authentication-and-authorization-in-aspnet-web-api.md#auth3)します。

フォーム認証では、セッションの cookie を使用して、要求を認証します。 ブラウザーは、web サイトに自動的にすべての関連クッキーを送信します。 この機能により、フォーム認証のクロスサイト リクエスト フォージェリ (CSRF) 攻撃を参照する可能性のある脆弱性のある[防止クロスサイト リクエスト フォージェリ (CSRF) 攻撃](preventing-cross-site-request-forgery-csrf-attacks.md)します。

フォーム認証では、ユーザーの資格情報は暗号化されません。 そのため、フォーム認証と SSL を使用しない場合は、安全ではありません。 参照してください[Web API の SSL の使用](working-with-ssl-in-web-api.md)します。
