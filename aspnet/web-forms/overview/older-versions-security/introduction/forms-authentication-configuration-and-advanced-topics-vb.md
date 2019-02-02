---
uid: web-forms/overview/older-versions-security/introduction/forms-authentication-configuration-and-advanced-topics-vb
title: フォーム認証構成と高度なトピック (VB) |Microsoft Docs
author: rick-anderson
description: このチュートリアルはさまざまなフォーム認証の設定を確認し、フォーム要素を使用して変更する方法を参照してください。 これが、詳細なが必要.
ms.author: riande
ms.date: 01/14/2008
ms.assetid: 829d2f56-5c48-445b-b826-3418a450c788
msc.legacyurl: /web-forms/overview/older-versions-security/introduction/forms-authentication-configuration-and-advanced-topics-vb
msc.type: authoredcontent
ms.openlocfilehash: eb533cf763c2f3132ea0a5420b4d4cbea16c61cd
ms.sourcegitcommit: 45ac74e400f9f2b7dbded66297730f6f14a4eb25
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/16/2018
ms.locfileid: "41828960"
---
<a name="forms-authentication-configuration-and-advanced-topics-vb"></a>フォーム認証構成と高度なトピック (VB)
====================
によって[Scott Mitchell](https://twitter.com/ScottOnWriting)

[コードのダウンロード](http://download.microsoft.com/download/2/F/7/2F705A34-F9DE-4112-BBDE-60098089645E/ASPNET_Security_Tutorial_03_VB.zip)または[PDF のダウンロード](http://download.microsoft.com/download/2/F/7/2F705A34-F9DE-4112-BBDE-60098089645E/aspnet_tutorial03_AuthAdvanced_vb.pdf)

> このチュートリアルはさまざまなフォーム認証の設定を確認し、フォーム要素を使用して変更する方法を参照してください。 (SignIn.aspx Login.aspx ではなく) などのカスタム URL と cookieless フォーム認証チケットでログイン ページを使用して、フォーム認証チケットのタイムアウト値のカスタマイズについて詳しく説明: これはします。


## <a name="introduction"></a>はじめに

[前のチュートリアル](an-overview-of-forms-authentication-vb.md)Web.config での表示 ページでさまざまなログを作成する構成設定を指定することから、ASP.NET アプリケーションでフォーム認証を実装するために必要な手順を説明しました認証および匿名ユーザーのコンテンツです。 Mode 属性を設定して、フォーム認証を使用する web サイトを構成したことを思い出してください、<authentication>フォームへの要素。 <authentication>要素には、必要に応じて、&lt;フォーム&gt;子要素で使用されるさまざまなフォーム認証設定を指定できます。

このチュートリアルでは、さまざまなフォーム認証設定を確認するを通じてそれらを変更する方法を参照してください、&lt;フォーム&gt;要素。 (SignIn.aspx Login.aspx ではなく) などのカスタム URL と cookieless フォーム認証チケットでログイン ページを使用して、フォーム認証チケットのタイムアウト値のカスタマイズについて詳しく説明: これはします。 また、フォーム認証チケットの構成をより詳しく調査し、ASP.NET がチケットのデータが検査および改ざんからセキュリティで保護されたことを確認するには、予防策を参照してください。 最後に、フォーム認証チケットに余分なユーザー データを格納する方法とカスタム プリンシパル オブジェクトを使用してこのデータをモデル化する方法を紹介します。

## <a name="step-1-examining-the-ltformsgt-configuration-settings"></a>手順 1: チェック、&lt;フォーム&gt;構成設定

ASP.NET フォーム認証システムは、アプリケーションごとにカスタマイズ可能な構成設定のいくつかあります。 などの設定が含まれます。 フォーム認証の有効期間がチケット。チケット; にどのような保護が適用されます。どのような条件クッキーなしの認証チケットが使用されます。ログイン ページへのパスその他の情報。 既定値を変更するには追加、 [&lt;フォーム&gt;要素](https://msdn.microsoft.com/library/1d3t3c61.aspx)の子として、 [<authentication>要素](https://msdn.microsoft.com/library/532aee0e.aspx)、これらのプロパティを指定します。値を XML 属性として、カスタマイズするようになります。

[!code-xml[Main](forms-authentication-configuration-and-advanced-topics-vb/samples/sample1.xml)]

表 1 でカスタマイズ可能なプロパティをまとめたものです、&lt;フォーム&gt;要素。 Web.config は、XML ファイルであるため、左の列の属性名は大文字小文字を区別します。


| <strong>属性</strong> |                                                                                                                                                                                                                                     <strong>説明</strong>                                                                                                                                                                                                                                      |
|----------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|         cookie なし         |                                                                                                                この属性は、URL に埋め込まれていると、cookie にどのような条件下で、認証チケットが格納されているを指定します。 使用可能な値: 既定値です。UseUri;自動検出します。値 (既定)。 手順 2 では、さらに詳しくは、この設定を検証します。                                                                                                                |
|         defaultUrl         |                                                                                                                                                         ユーザーは、クエリ文字列で指定された RedirectUrl 値がない場合、ログイン ページからサインインした後にリダイレクトされる URL を示します。 既定値は default.aspx です。                                                                                                                                                         |
|           ドメイン           | Cookie ベースの認証チケットを使用する場合、この設定は、cookie のドメインの値を指定します。 既定値は、発行 (www.yourdomain.com) などがドメインを使用するブラウザーにより空の文字列。 クッキーが、この場合、<strong>いない</strong>admin.yourdomain.com などのサブドメインに要求を行う場合に送信されます。 すべてのサブドメインに渡される cookie が必要な場合は、yourdomain.com に設定すると、ドメイン属性をカスタマイズする必要があります。 |
|  enableCrossAppRedirects   |                                                                                                                                                                   同じサーバー上の他の web アプリケーションの Url にリダイレクトされたときに、認証されたユーザーが記憶されるかどうかを示すブール値。 既定値は false です。                                                                                                                                                                   |
|          loginUrl          |                                                                                                                                                                                                                      ログイン ページの URL。 既定値は、login.aspx です。                                                                                                                                                                                                                      |
|            name            |                                                                                                                                                                                                   Cookie ベースの認証チケット cookie の名前を使用します。 場合、 既定では。ASPXAUTH です。                                                                                                                                                                                                   |
|            path            |                                                                             Cookie ベースの認証チケットを使用する場合、この設定は、クッキーのパス属性を指定します。 Path 属性は、開発者は特定のディレクトリ階層のクッキーのスコープを制限できます。 既定値は、/ドメインへの要求に認証チケットのクッキーを送信するブラウザーに通知します。                                                                              |
|         保護         |                                                                                                                                            フォーム認証チケットの保護にどのような手法を使用することを示します。 使用可能な値: すべて (既定)。暗号化。None;および検証します。 これらの設定は、手順 3 で詳しく説明します。                                                                                                                                            |
|         requireSSL         |                                                                                                                                                                                認証クッキーを送信する SSL 接続が必要かどうかを示すブール値。 既定値は false です。                                                                                                                                                                                |
|     slidingExpiration      |                                                                                                 ユーザーが 1 つのセッション中にサイトを訪問する認証の cookie のタイムアウトが毎回をリセットするかどうかを示すブール値。 既定値は true です。 認証チケットのタイムアウト ポリシーが、指定することで詳細に説明されているチケットのタイムアウト値のセクション。                                                                                                 |
|          タイムアウト           |                                                                                                                               認証チケットのクッキーの有効期限が切れるまでの分、時間を指定します。 既定値は、30 です。 認証チケットのタイムアウト ポリシーが、指定することで詳細に説明されているチケットのタイムアウト値のセクション。                                                                                                                               |

**表 1.**: の A の概要、&lt;フォーム&gt;要素の属性

ASP.NET 2.0 では、既定値を超えるフォーム認証の値は FormsAuthenticationConfiguration クラス、.NET Framework でハード コードされます。 Web.config ファイルで、アプリケーションによるごとに変更を適用する必要があります。 一方、ASP.NET 1.x では、フォーム認証の既定値、machine.config ファイルに格納されていた (ありため、machine.config を編集して変更できます)。 ASP.NET のトピックで説明することは、1.x では、さまざまなフォーム認証システムの設定がある ASP.NET 2.0 では、異なる既定値の asp.net よりも拡張 1.x します。 ASP.NET 1.x 環境からアプリケーションを移行する場合は、これらの違いを意識する必要があります。 参照してください[、&lt;フォーム&gt;要素のテクニカル ドキュメント](https://msdn.microsoft.com/library/1d3t3c61.aspx)の相違点の一覧についてはします。

> [!NOTE]
> タイムアウト、ドメイン、および、パスなど、いくつかのフォーム認証設定は、結果として得られるフォーム認証チケット cookie の詳細を指定します。 Cookie、そのしくみ、およびそのさまざまなプロパティの詳細については、読み取る[この Cookie チュートリアル](http://www.quirksmode.org/js/cookies.html)します。


### <a name="specifying-the-tickets-timeout-value"></a>チケットのタイムアウト値を指定します。

フォーム認証チケットは、id を表すトークンです。 このトークンは、cookie ベースの認証チケットを cookie の形式で保持されているし、要求ごとに web サーバーに送信します。 トークンを所有しているが基本的には、宣言、私は*username*に既にログインし、ページの訪問者の間でユーザーの id を記憶できるように使用されます。

フォーム認証チケットだけでなく、ユーザーの id が含まれていますが、整合性と、トークンのセキュリティを確保する情報も含まれます。 結局のところ、不正なユーザー underhanded 何らかのトークンの正当なエントリを変更したり、偽造のトークンを作成することがほしくありません。

チケットに含まれる情報のような 1 つのビットは、*有効期限*、これは、チケットが無効になった日時。 FormsAuthenticationModule は、認証チケットを検査するたびに、チケットの有効期限がまだ達していないことを保証します。 場合は、チケットは無視され、匿名としてユーザーを識別します。 このセーフガードは、リプレイ攻撃から保護するのに役立ちます。 ない場合は、ハッカーが自分のコンピューターに物理的にアクセスし、cookie を root 化によって、おそらく自分のユーザーの有効な認証チケットのハンズオンを取得することでこの盗難にあった認証チケットを使用してサーバーに要求を送信する可能性があります、有効期限とエントリを取得します。 有効期限は、このシナリオを防ぐため、中には、このような攻撃が成功するウィンドウを制限します。

> [!NOTE]
> 手順 3 の詳細その他の方法、フォーム認証システムで認証チケットを保護するために使用します。


認証チケットを作成するときに、フォーム認証システムは、タイムアウト設定を参照してその有効期限を決定します。 表 1 では、タイムアウトを 30 分に既定値の設定で説明したようには、フォーム認証チケットの作成時に、その有効期限が、日付と時刻の 30 分後に設定されていることを意味します。

有効期限が切れる絶対時刻が将来のフォーム認証チケットの有効期限が切れるを定義します。 通常は開発者が、ユーザーは、サイトを見直してたびにリセットするスライド式の有効期限を実装します。 この動作は、slidingExpiration 設定によって決定されます。 FormsAuthenticationModule は、ユーザーを認証するたびに true (既定値) に設定を更新、チケットの有効期限。 要求ごとに、有効期限を false に設定が更新されない場合は、チケットを過去の場合、チケットが最初の分のタイムアウト数を正確に有効期限が切れる原因となり作成されます。

> [!NOTE]
> 認証チケットに格納されている有効期限は、絶対日付と 2008 年 8 月 2 日午前 11時 34分のように、時刻の値です。 さらに、日付と時刻は、web サーバーのローカル時間を基準とは。 この設計の決定には、いくつか興味深い副作用を夏時間 (DST)、(web サーバーが夏時間のタイムが発生したロケールでホストされていると仮定)、1 時間先移動米国の州のクロックであることができます。 DST の開始時刻の近くの 30 分間の有効期限と ASP.NET web サイトに行われる処理を検討してください (2時 00分 am です)。 訪問者にサインオンし、サイト、2008 年 3 月 11 日の午前 1時 55分を想像してください。 これにより、2時 25分 AM (30 分後) に、2008 年 3 月 11 日に期限切れになるフォーム認証チケットが生成されます。 ただし、2時 00分 AM をロールに、クロック ジャンプ 3時 00分 am DST によりします。 ユーザーには、(3時 01分 AM) にサインインした後に 6 分間に新しいページが読み込まれる、FormsAuthenticationModule は、チケットが有効期限が切れたし、ユーザーをログイン ページにリダイレクトすることを示します。 これと他の認証チケットのタイムアウト奇妙な動作、ほかの回避策の詳細については、Stefan Schackow のコピーを取得します*Professional ASP.NET 2.0 のセキュリティ、メンバーシップ、およびロール管理*(ISBN:。978-0-7645-9698-8)。


図 1 は、slidingExpiration が false に設定して、タイムアウトが 30 に設定するときにワークフローを示します。 ログイン時に生成された認証チケットには、有効期限の日付が含まれていて、後続の要求では、この値は更新されていません。 FormsAuthenticationModule では、チケットの期限が切れたことが検出されると、それを破棄し、匿名として、要求を処理します。


[![フォーム認証チケットの有効期限と slidingExpiration のグラフィカル表現は false です。](forms-authentication-configuration-and-advanced-topics-vb/_static/image2.png)](forms-authentication-configuration-and-advanced-topics-vb/_static/image1.png)

**図 01**: A がグラフィカルにフォーム認証チケットの有効期限と slidingExpiration が false ([フルサイズの画像を表示する をクリックします](forms-authentication-configuration-and-advanced-topics-vb/_static/image3.png))。


SlidingExpiration に設定すると、図 2 は、ワークフローを示しています true とタイムアウトが 30 に設定します。 (有効期限のないチケット) を使用して認証された要求が受信したときに、その有効期限は、タイムアウト時間 (分)、将来に更新されます。


[![フォーム認証チケットのグラフィカル表現 slidingExpiration が true の場合](forms-authentication-configuration-and-advanced-topics-vb/_static/image5.png)](forms-authentication-configuration-and-advanced-topics-vb/_static/image4.png)

**図 02**: A がグラフィカルにフォーム認証チケットの slidingExpiration が true の場合 ([フルサイズの画像を表示する をクリックします](forms-authentication-configuration-and-advanced-topics-vb/_static/image6.png))。


Cookie ベースの認証チケット (既定値) を使用する場合このディスカッション cookie は、指定された独自の切れことがあるできますので、もう少し複雑になります。 Cookie の有効期限 (または欠如) クッキーを破棄する必要があるときに、ブラウザーに指示します。 Cookie に有効期限がない場合、ブラウザーがシャット ダウン時に破棄されます。 ただし、有効期限が存在する場合は、日まで、cookie がユーザーのコンピューターに格納されていると、有効期限で指定された時間が経過します。 ブラウザーで cookie が破棄されると、不要になった web サーバーに送信されます。 そのため、cookie の破棄は、ユーザーのサイトからのログインに似ています。

> [!NOTE]
> もちろん、ユーザーは、自分のコンピューターに格納されているすべての cookie を事前に削除できます。 Internet Explorer 7 はツール]、[オプション] に移動し、[閲覧の履歴セクションで [削除] ボタンをクリックします。 そこから、cookie の削除 ボタンをクリックします。


フォーム認証システムに渡される値によって、セッション ベースまたは有効期限ベースの cookie を作成し、 *persistCookie*パラメーター。 再現率、FormsAuthentication クラスの GetAuthCookie、SetAuthCookie、RedirectFromLoginPage のメソッドが 2 つの入力パラメーターに取る: *username*と*persistCookie*します。 前のチュートリアルで作成したログイン ページには、アカウントを記憶 チェック ボックス、永続的なクッキーが作成されたかどうかを決定が含まれています。 永続的な cookie は、有効期限ベースです。非永続の cookie は、セッション ベースです。

既に説明したタイムアウトと slidingExpiration の概念には、両方のセッションおよび有効期限ベースの cookie に同じが適用されます。 実行で 1 つだけの小さな違いがある: を true に設定された slidingTimeout cookie の有効期限ベースを使用する場合、cookie の有効期限のみ更新されます以上の指定した時間の半分が経過しました。

チケットのスライド式有効期限を使用して、1 時間 (60 分) 後にタイムアウトを実行するため、web サイトの認証チケットのタイムアウト ポリシーを更新してみましょう。 この変更の影響を与える、Web.config ファイルの更新の追加、&lt;フォーム&gt;要素を<authentication>を次のマークアップ要素。

[!code-xml[Main](forms-authentication-configuration-and-advanced-topics-vb/samples/sample2.xml)]

### <a name="using-an-login-page-url-other-than-loginaspx"></a>Login.aspx 以外のログイン ページの URL を使用します。

FormsAuthenticationModule は、ログイン ページに自動的に承認されていないユーザーをリダイレクト、されるので、ログイン ページの url が必要です。 この URL は loginUrl 属性で指定された、&lt;フォーム&gt;要素、既定値は login.aspx です。 既存の web サイト経由で移植する場合は、別の URL、ブックマークを設定および検索エンジンによってインデックスが作成された既にいずれかでログイン ページを既にがあります。 Login.aspx に、既存のログイン ページの名前を変更して、リンクやユーザーのブックマークの重大なではなく代わりに、ログイン ページを指す loginUrl 属性を変更できます。

たとえば、ログイン ページが SignIn.aspx をという名前がユーザー ディレクトリに存在していた場合は、~/Users/SignIn.aspx loginUrl 構成設定を指し示すでした次のようにします。

[!code-xml[Main](forms-authentication-configuration-and-advanced-topics-vb/samples/sample3.xml)]

カスタム値を指定する必要はありません、現在のアプリケーションは既にには、Login.aspx という名前のログイン ページがあるので、&lt;フォーム&gt;要素。

## <a name="step-2-using-cookieless-forms-authentication-tickets"></a>手順 2: Cookieless フォーム認証チケットを使用します。

既定では、フォーム認証システムには、その認証チケットをクッキー コレクションに格納したり、サイトにアクセスしてユーザー エージェントに基づく URL に埋め込むかどうかが決定します。 メイン ストリームのすべてのデスクトップ ブラウザーなど、Internet Explorer、Firefox、Opera、Safari、およびサポート cookie がないすべてのモバイル デバイスの操作を行います。

フォーム認証システムによって使用される cookie のポリシーが cookie なしの設定に依存、&lt;フォーム&gt;要素、4 つの値の 1 つ割り当てることができます。

- 既定値には、cookie ベースの認証チケットを常に使用することを指定します。
- UseUri - は、cookie ベースの認証チケットを使用しないことを示します。
- 自動検出 - デバイスのプロファイルは、cookie ベースの認証チケット cookie をサポートしていない場合は使用されません。デバイス プロファイルでは、cookie をサポートする場合、プローブ メカニズムは、cookie が有効になっているかどうかを確認に使用されます。
- UseDeviceProfile - 既定では、デバイス プロファイルは、cookie をサポートしている場合にのみ、クッキー ベースの認証チケットを使用します。 プローブのメカニズムは使用されません。

自動検出と値の設定に依存、*デバイス プロファイル*で cookie を使用するか cookie なしの認証チケットを使用するかどうかを確認します。 ASP.NET では、さまざまなデバイスとその機能、cookie をサポートしているかどうか、JavaScript をサポートしているのバージョンなどのデータベースを保持します。 毎回デバイスに送信する web サーバーから web ページを要求する*ユーザー エージェント*デバイスの種類を識別する HTTP ヘッダー。 ASP.NET には、そのデータベースで指定された対応するプロファイルで指定されたユーザー エージェント文字列に自動的と一致します。

> [!NOTE]
> デバイスの機能には、このデータベースに準拠している複数の XML ファイルに格納されている、[ブラウザー定義ファイルのスキーマ](https://msdn.microsoft.com/library/ms228122.aspx)します。 既定のデバイス プロファイル ファイルは、%windir%\microsoft.net\framework\v2.0.50727\config\browsers に配置されます。 アプリケーションのアプリにカスタム ファイルを追加することもできます。\_ブラウザー フォルダー。 詳細については、次を参照してください。 [How To: ブラウザーの種類を検出する ASP.NET Web Pages で](https://msdn.microsoft.com/library/3yekbd5b.aspx)します。


既定の設定は、値であるため cookieless フォーム認証チケットはプロファイルが cookie がサポートされていないことを報告するデバイスで、サイトがアクセスしたときに使用されます。

### <a name="encoding-the-authentication-ticket-in-the-url"></a>URL の認証チケットのエンコード

Cookie は、訪問先デバイスには、それらがサポートしている場合、既定のフォーム認証設定が cookie を使用する理由は、その web サイトへの要求ごとに、ブラウザーからの情報を含めるの自然なメディアです。 Cookie がサポートされていない場合は、クライアントからサーバーに認証チケットを渡すための別の方法が採用されている必要があります。 Cookie なしの環境で使用される一般的な回避策では、URL で cookie のデータをエンコードします。

URL 内でこのような情報を埋め込むことがどのように表示する最善の方法は、サイトでクッキーなしの認証チケットを使用するされます。 これは、UseUri に cookie なしの構成設定を設定して実行できます。

[!code-xml[Main](forms-authentication-configuration-and-advanced-topics-vb/samples/sample4.xml)]

この変更を行った場合、ブラウザーを使用してサイトを参照してください。 匿名ユーザーとしてすると、Url は以前とまったく同じになります。 たとえば、Default.aspx ページにアクセスすると、ブラウザーのアドレス バーは、次の URL を示しています。

`http://localhost:2448/ASPNET\_Security\_Tutorial\_03\_CS/default.aspx`

ただし、ログイン時にフォーム認証チケットが URL に埋め込まれます。 たとえば、ログイン ページにアクセスし、Sam としてログインした後、Default.aspx ページに戻るいますが、URL のこの時間は。

`http://localhost:2448/ASPNET\_Security\_Tutorial\_03\_CS/(F(jaIOIDTJxIr12xYS-VVgkqKCVAuIoW30Bu0diWi6flQC-FyMaLXJfow\_Vd9GZkB2Cv-rfezq0gKadKX0YPZCkA2))/default.aspx`

URL 内で、フォーム認証チケットが埋め込まれています。 文字列 (F (jaIOIDTJxIr12xYS-VVgkqKCVAuIoW30Bu0diWi6flQC-FyMaLXJfow\_Vd9GZkB2Cv rfezq0gKadKX0YPZCkA2)、16 進エンコードされた認証チケット情報を表し、cookie に通常格納されている同じデータです。

クッキーなしの認証チケットさせるためには、システムは、認証チケットのデータを含める ページのすべての Url をエンコードする必要があります、それ以外の場合、認証チケットは、ユーザーがリンクをクリックすると失われます。 さいわいにも、この埋め込みロジックは、自動的に実行されます。 この機能を示すためには、Default.aspx ページを開き、テキストと NavigateUrl プロパティをテスト用のリンクと SomePage.aspx に設定すると、それぞれ、ハイパーリンク コントロールを追加します。 ありませんページ SomePage.aspx という名前のプロジェクトでは関係ありません。

Default.aspx に変更を保存し、ブラウザーを使用しを参照してください。 フォーム認証チケットが URL に埋め込まれているように、サイトにログオンします。 次に、Default.aspx からには、テスト用のリンクのリンクをクリックします。 どうなっているのでしょうか。 SomePage.aspx という名前のページが存在しない場合、404 エラーが発生しましたが、いない重要なはここでは。 お使いのブラウザーのアドレス バーに専念します。 URL にフォーム認証チケットに含まれていることに注意してください。

`http://localhost:2448/ASPNET\_Security\_Tutorial\_03\_CS/(F(jaIOIDTJxIr12xYS-VVgkqKCVAuIoW30Bu0diWi6flQC-FyMaLXJfow\_Vd9GZkB2Cv-rfezq0gKadKX0YPZCkA2))/SomePage.aspx`

リンク URL SomePage.aspx が認証チケットに含まれている URL に変換が自動的にコードがまったくを記述する必要があるしませんでした。 Http:// で始まらない任意のハイパーリンクの URL には、フォーム認証チケットが自動的に埋め込まれるまたは/。 Response.Redirect への呼び出しで、ハイパーリンク コントロール、または HTML アンカー要素内にハイパーリンクが表示されるかは関係がありません (つまり、 &lt;、href =「...」&gt;.&lt;/a&gt;)。 URL のようなものでない限り、 http://www.someserver.com/SomePage.aspx または/SomePage.aspx、フォーム認証チケットは、私たちにとって埋め込まれます。

> [!NOTE]
> Cookieless フォーム認証チケットは、cookie ベースの認証チケットとして同じタイムアウト ポリシーに準拠します。 ただし、クッキーなしの認証チケットは、認証チケットが URL に直接埋め込まれているために、リプレイ攻撃を受けやすいです。 Web サイトを訪問、ログイン、および同僚に電子メールで URL を貼り付けますユーザーを想像してください。 仕事仲間は、有効期限に達する前にそのリンクをクリックすると、それらとして記録されます電子メールを送信したユーザー。


## <a name="step-3-securing-the-authentication-ticket"></a>手順 3: 認証チケットをセキュリティで保護します。

フォーム認証チケットは、ネットワーク経由で送信される cookie のいずれかまたは URL 内で直接埋め込まれました。 Id の情報に加えて認証チケット含めることができますもユーザー データ (手順 4. で表示されます)。 したがって、これが、チケットのデータが暗号化されている重要なにチケットが変更されていない覗き見とする (さらに) から、フォーム認証システムを保証できます。

チケットのデータのプライバシーを確保するには、フォーム認証システムは、チケット データを暗号化できます。 チケット データを暗号化する障害は、プレーン テキストでネットワーク経由で機密性の高い情報を送信します。

チケットの信頼性を保証するために、フォーム認証システムがする必要があります*検証*チケット。 検証が特定のデータが変更されていないと、によって実現されます。 確保の act、 *[メッセージ認証コード (MAC)](http://en.wikipedia.org/wiki/Message_authentication_code)* します。 簡単に言うと、MAC は、(この場合は、チケット) を検証する必要があるデータを識別する情報の一部です。 MAC で表されるデータが変更された場合、MAC と、データは調和していません。 さらに、データを変更して、変更されたデータに対応するため、自分の MAC の生成の両方に、ハッカーの計算が困難です。

作成 (または変更) とチケット、フォーム認証システムは、MAC を作成し、チケットのデータにアタッチします。 後続の要求が到着すると、フォーム認証システムは、チケット データの信頼性を検証する、MAC、およびチケットのデータを比較します。 図 3 では、このワークフローがグラフィカルに示しています。


[![MAC により、チケットの信頼性が確保されます。](forms-authentication-configuration-and-advanced-topics-vb/_static/image8.png)](forms-authentication-configuration-and-advanced-topics-vb/_static/image7.png)

**図 03**MAC により、チケットの信頼性が確保されます ([フルサイズの画像を表示する をクリックします。](forms-authentication-configuration-and-advanced-topics-vb/_static/image9.png))。


認証チケットにどのようなセキュリティ対策が適用されるは、保護設定によって異なります、&lt;フォーム&gt;要素。 保護の設定は、次の 3 つの値のいずれかに割り当てることができます。

- すべてのチケットが暗号化されており、デジタル署名されて (既定)。
- 暗号化 - のみの暗号化の適用 - MAC は生成されません。
- 暗号化なし - チケットもデジタル署名。
- 検証 - MAC が生成されますが、チケット データがプレーン テキストでネットワーク経由で送信されます。

Microsoft では、すべての設定を使用して強くお勧めします。

### <a name="setting-the-validation-and-decryption-keys"></a>検証と復号化キーの設定

暗号化とハッシュの暗号化、認証チケットを検証し、フォーム認証システムで使用されるアルゴリズムはを通じてカスタマイズ可能な[ &lt;machineKey&gt;要素](https://msdn.microsoft.com/library/w8h3skw9.aspx)Web.config にします。表 2 のアウトライン、 &lt;machineKey&gt;要素の属性とその考えられる値。

| **属性** | **説明** |
| --- | --- |
| 復号化 | 暗号化に使用されるアルゴリズムを示します。 この属性は次の 4 つの値のいずれかであることができます: - 自動 - 既定では、キー decryptionKey 属性の長さに基づくアルゴリズムを決定します。 -AES - を使用して、 [Advanced Encryption Standard (AES)](http://en.wikipedia.org/wiki/Advanced_Encryption_Standard)アルゴリズム。 DES - を使用して、[データ暗号化標準 (DES)](http://en.wikipedia.org/wiki/Data_Encryption_Standard)このアルゴリズムは脆弱な計算と見なされます、使用できません。 3 - des-を使用して、 [Triple DES](http://en.wikipedia.org/wiki/Triple_DES)アルゴリズムで、3 回、DES アルゴリズムを適用することで動作します。 |
| decryptionKey | 暗号化アルゴリズムで使用されるシークレット キー。 この値に、(復号化の値に基づく) 適切な長さ、自動生成、またはいずれかの値を使用すると、追加の 16 進数文字列 IsolateApps します。 IsolateApps を追加するには、ASP.NET アプリケーションごとに一意の値を使用するように指示します。 既定では AutoGenerate, IsolateApps です。 |
| 検証 | 検証に使用されるアルゴリズムを示します。 この属性は次の 4 つの値のいずれかであることができます: - AES - は、Advanced Encryption Standard (AES) アルゴリズムを使用します。 MD5 の使用、[メッセージ ダイジェスト 5 (MD5)](http://en.wikipedia.org/wiki/MD5)アルゴリズム。 -SHA1 - を使用して、 [SHA1](http://en.wikipedia.org/wiki/Sha1)アルゴリズム (既定)。 -3 des、Triple DES アルゴリズムを使用します。 |
| validationKey | 検証アルゴリズムで使用されるシークレット キー。 この値に、(検証の値に基づく) 適切な長さ、自動生成、またはいずれかの値を使用すると、追加の 16 進数文字列 IsolateApps します。 IsolateApps を追加するには、ASP.NET アプリケーションごとに一意の値を使用するように指示します。 既定では AutoGenerate, IsolateApps です。 |

**表 2.**: &lt;machineKey&gt;要素の属性

暗号化と検証のオプション、およびプロの徹底的に検討して、さまざまなアルゴリズムの欠点は、このチュートリアルの範囲外です。 どのようなキーの長さを使用するガイダンスについては、使用するにはどのような暗号化と検証アルゴリズムを含む、これらの問題で、詳細な検索のこれらのキーの生成を参照する最適な方法と*Professional ASP.NET 2.0 のセキュリティ、メンバーシップ、およびロール管理*.

既定では、暗号化と検証に使用されるキーは、アプリケーションごとに自動的に生成し、これらのキーはローカル セキュリティ機関 (LSA) で格納されます。 つまり、既定の設定では、web サーバー-web でのサーバーとアプリケーション間ごとに一意のキーが保証されます。 そのため、この既定の動作は、次の 2 つのシナリオでは機能しません。

- **Web ファーム**-、 [web ファーム](http://en.wikipedia.org/wiki/Web_farm)シナリオでは、1 つの web アプリケーションが拡張性と冗長性のために複数の web サーバーでホストされています。 各受信要求は、ユーザーのセッションの有効期間は、別のサーバーをさまざまな要求を処理するために使用可能性があることを意味、ファーム内のサーバーにディスパッチされます。 その結果、フォーム認証チケットを作成できるように、各サーバーは、同じ暗号化と検証キーを使用する必要があります、暗号化、および 検証済みの 1 つのサーバーの暗号化を解除して、ファーム内の別のサーバーで検証されることができます。
- **アプリケーション チケットの共有をクロス**-1 つの web サーバーは、複数の ASP.NET アプリケーションをホストできます。 これらのさまざまなアプリケーションを 1 つのフォーム認証チケットを共有する必要がある場合は、、暗号化と検証キーが一致する必要があります。

Web ファームの設定や、同じサーバー上のアプリケーション間での認証チケットの共有で作業するときは、構成する必要があります、 &lt;machineKey&gt;影響を受けるアプリケーション内の要素をそのキー decryptionKey とキー validationkey キー値に一致。

サンプル アプリケーションに適用されますが、上記のシナリオのどちらも、明示的なキー decryptionKey キー validationkey キー値を指定を使用するアルゴリズムを定義もできます。 追加、 &lt;machineKey&gt; Web.config ファイルに設定します。

[!code-xml[Main](forms-authentication-configuration-and-advanced-topics-vb/samples/sample5.xml)]

詳細については、チェック アウト[How To: ASP.NET 2.0 の構成の MachineKey](https://msdn.microsoft.com/library/ms998288.aspx)します。

> [!NOTE]
> キー decryptionKey と validationKey の値から取得されました[Steve Gibson](http://www.grc.com/stevegibson.htm)の[パスワードの完全な web ページ](https://www.grc.com/passwords.htm)、64 のランダムな 16 進数の文字を各ページのアクセス時に生成されます。 これらのキーが、実稼働アプリケーションに入り込んでが発生する可能性を軽減するために、完璧なパスワード ページからランダムに生成されたもので、上記のキーを置換することが推奨されます。


## <a name="step-4-storing-additional-user-data-in-the-ticket"></a>手順 4: チケットに追加のユーザー データを格納します。

多くの web アプリケーションでは、に関する情報を表示またはページの表示に基づく、現在ログオンしているユーザー。 たとえば、ユーザーの名前と日付のすべてのページの上隅で、最後にログインした web ページを表示する場合があります。 フォーム認証チケットには、現在ログオンしているユーザーのユーザー名が格納されますが、ページが、ユーザー ストア - 通常はデータベース - 認証チケットに格納されていない情報を参照に移動する必要がありますその他の情報が必要なときにします。

ほんの少しのコードでは、フォーム認証チケットを追加のユーザー情報を保存したことができます。 このようなデータを表すことができます、 [FormsAuthenticationTicket クラス](https://msdn.microsoft.com/library/system.web.security.formsauthenticationticket.aspx)の[UserData プロパティ](https://msdn.microsoft.com/library/system.web.security.formsauthenticationticket.userdata.aspx)します。 これは、少量が一般的に必要なユーザーに関する情報を配置するのに便利な場所です。 UserData プロパティを暗号化され、検証が含まれる一部として、認証チケット cookie のと同様、他のチケットのフィールドで指定された値は、フォーム認証システムの構成に基づいています。 既定では、UserData は空の文字列です。

認証チケットのユーザー データを格納するためには、ユーザーに固有の情報を取得し、チケットをログイン ページでのコードを記述する必要があります。 UserData は、文字列型のプロパティであるためにを文字列としてが格納されている、データを正しくシリアル化する必要があります。 たとえば、ユーザー ストアには、各ユーザーの生年月日と、雇用者の名前が含まれていること、および認証チケットでこれらの 2 つのプロパティ値を保存する必要があるとします。 文字列にこれらの値をシリアル化後に、雇用主名前パイプ (|)、生年月日の文字列のユーザーの日付を連結することによってでした。 Northwind Traders に適している、1974 年 8 月 15日でユーザーを割り当てます UserData プロパティ文字列: 1974-08-15 |Northwind Traders します。

チケットに格納されているデータにアクセスする必要があります、たびにこれを現在の要求の所属をグラブして UserData プロパティを逆シリアル化して実行できます。 生年月日と雇用者名の例の日付の場合は、区切り記号 (|) に基づく 2 つの部分文字列に UserData 文字列を分割します。


[![認証チケットに追加のユーザー情報を格納できます。](forms-authentication-configuration-and-advanced-topics-vb/_static/image11.png)](forms-authentication-configuration-and-advanced-topics-vb/_static/image10.png)

**図 04**: その他のユーザー情報に格納できる認証チケット ([フルサイズの画像を表示する をクリックします](forms-authentication-configuration-and-advanced-topics-vb/_static/image12.png))。


### <a name="writing-information-to-userdata"></a>UserData 情報を書き込む

残念ながら、フォーム認証チケットをユーザーに固有の情報を追加するは想像のいずれかの単純ではありません。 FormsAuthenticationTicket クラスの UserData プロパティは読み取り専用と、FormsAuthenticationTicket クラスのコンス トラクターでのみ指定できます。 また、チケットの他の値を提供する必要がありますをコンス トラクターで UserData プロパティを指定する場合: ユーザー名、発行日、有効期限、およびなど。 前のチュートリアルでは、ログイン ページを作成したときにこれがすべてのクラスによって処理される、FormsAuthentication します。 UserData を所属に追加する際は既に FormsAuthentication クラスによって提供される機能の多くをレプリケートするコードを記述する必要があります。

認証チケットにユーザーに関する追加情報を記録する Login.aspx ページを更新することで、UserData を操作するために必要なコードを見てみましょう。 ふり、ユーザー ストアにはでのユーザーが勤務する会社と、そのタイトルに関する情報が含まれていると認証チケットには、この情報をキャプチャします。 次のように、次のコードは、Login.aspx ページのログイン ボタンの Click イベント ハンドラーを更新します。

[!code-vb[Main](forms-authentication-configuration-and-advanced-topics-vb/samples/sample6.vb)]

一度にこのコードを 1 行ずつステップみましょう。 4 つの文字列配列を定義することで、メソッドを開始します。 ユーザー、パスワード、companyName、および titleAtCompany します。 これらの配列を保持ユーザー名、パスワード、会社名、およびユーザー アカウントのタイトルをシステムでは、これは、3 つ: Scott、Jisun、および Sam です。 実際のアプリケーションでこれらの値はハードコードされていません ページのソース コード内のユーザー ストアから照会は。

前のチュートリアルで指定された資格情報が有効な場合を単純に呼び出す、次の手順を実行する FormsAuthentication.RedirectFromLoginPage (UserName.Text、RememberMe.Checked)。

1. フォーム認証チケットの作成
2. 適切なストアにチケットを作成しました。 Cookie ベースの認証チケットのブラウザーのクッキーのコレクションが使用されます。クッキーなしの認証のチケットの場合は、チケット データは、URL にシリアル化します。
3. 適切なページにユーザーをリダイレクトします。

次の手順は、上記のコードでレプリケートされます。 最初に、UserData プロパティは最終的に格納する文字列は、会社名とパイプ文字 (|) では、2 つの値を区切る、タイトルを組み合わせることによって構成です。

Dim userDataString As String = String.Concat(companyName(i)、"|"、titleAtCompany(i))

次に、認証チケットを作成するには、メソッドが呼び出される FormsAuthentication.GetAuthCookie 暗号化構成の設定に従って検証しますおよび HttpCookie オブジェクトに配置されます。

Dim authCookie として HttpCookie FormsAuthentication.GetAuthCookie (UserName.Text、RememberMe.Checked) を =

クッキー内に埋め込まれて FormAuthenticationTicket を使用するにを FormAuthentication クラスを呼び出す必要[メソッドを復号化](https://msdn.microsoft.com/library/system.web.security.formsauthentication.decrypt.aspx)で cookie の値渡し。

チケットとして FormsAuthenticationTicket を dim FormsAuthentication.Decrypt(authCookie.Value) を =

作成し、*新しい*既存 FormsAuthenticationTicket の値に基づいて、FormsAuthenticationTicket インスタンス。 ただし、この新しいチケットには、ユーザーに固有の情報 (userDataString) が含まれています。

Dim newTicket FormsAuthenticationTicket として新しい FormsAuthenticationTicket(ticket. を =チケットのバージョンです。チケットの名前。IssueDate、チケットします。有効期限、チケット。IsPersistent、userDataString)

暗号化 (し検証) を呼び出して新しい FormsAuthenticationTicket インスタンス、[暗号化メソッド](https://msdn.microsoft.com/library/system.web.security.formsauthentication.encrypt.aspx)authCookie にこの暗号化された (および検証済み) のデータを戻すとします。

authCookie.Value FormsAuthentication.Encrypt(newTicket) を =

最後に、authCookie が Response.Cookies コレクションに追加し、GetRedirectUrl メソッドはユーザーに送信する適切なページを確認します。

[!code-vb[Main](forms-authentication-configuration-and-advanced-topics-vb/samples/sample7.vb)]

UserData プロパティは読み取り専用と FormsAuthentication クラスがその GetAuthCookie や SetAuthCookie、RedirectFromLoginPage メソッドで UserData 情報を指定するためのメソッドを提供していないため、このコードのすべてが必要です。

> [!NOTE]
> について確認しましたコードでは、cookie ベースの認証チケットのユーザー固有の情報を格納します。 URL にフォーム認証チケットをシリアル化するクラスは、.NET Framework の内部。 かいつまんで言う、cookieless フォーム認証チケットのユーザー データを格納することはできません。


### <a name="accessing-the-userdata-information"></a>UserData 情報にアクセスします。

この時点で各ユーザーの会社名とタイトルは、ログインしたときに、フォーム認証チケットの UserData プロパティに格納されます。 この情報は、ユーザー ストアへのトリップを必要とせず、任意のページで、認証チケットからアクセスできます。 UserData プロパティからこの情報を取得する方法を示すためには、更新 Default.aspx、ようこそメッセージは、だけでなく、ユーザーの名前がも各自が勤務する会社とそのタイトルが含まれるようにします。

現時点では、Default.aspx、AuthenticatedMessagePanel パネルの WelcomeBackMessage という名前のラベル コントロールに含まれています。 このパネルは、認証されたユーザーにのみ表示されます。 Default.aspx のページのコードを更新\_Load イベント ハンドラーを次のように見えるようにします。

[!code-vb[Main](forms-authentication-configuration-and-advanced-topics-vb/samples/sample8.vb)]

Request.IsAuthenticated が true の場合場合 WelcomeBackMessage の Text プロパティが最初に、ようこそ*username*します。 次に、//新しい一般プリンシパル プロパティは、基になる FormsAuthenticationTicket アクセスできるように、FormsIdentity オブジェクトにキャストされます。 所属したらは、会社名とタイトルに UserData プロパティ逆シリアル化します。 これは、パイプ文字の文字列を分割することで実現されます。 会社名とタイトルが WelcomeBackMessage ラベルに表示されます。

図 5 は、実行中、この画面のスクリーン ショットを示します。 Scott としてログインするには、Scott の会社とタイトルを含むバックへようこそ のメッセージが表示されます。


[![現在ログインして ユーザーの会社とタイトルが表示されます。](forms-authentication-configuration-and-advanced-topics-vb/_static/image14.png)](forms-authentication-configuration-and-advanced-topics-vb/_static/image13.png)

**図 05**: The 現在にログオンしたユーザーの会社とタイトルが表示されます ([フルサイズの画像を表示する をクリックします](forms-authentication-configuration-and-advanced-topics-vb/_static/image15.png))。


> [!NOTE]
> 認証チケットの UserData プロパティは、ユーザー ストアのキャッシュとして機能します。 任意のキャッシュのように基になるデータが変更されたときに更新が必要です。 たとえば、ユーザーが自分のプロファイルを更新できる web ページがある場合、ユーザーが行った変更を反映するように UserData プロパティにキャッシュされているフィールドを更新する必要があります。


## <a name="step-5-using-a-custom-principal"></a>手順 5: カスタム プリンシパルを使用します。

受信要求のたびに、FormsAuthenticationModule は、ユーザーを認証しようとします。 有効期限のない認証チケットが存在する場合、FormsAuthenticationModule は HttpContext.User プロパティを新しい GenericPrincipal オブジェクトに割り当てます。 この GenericPrincipal オブジェクトには、フォーム認証チケットへの参照を含む、FormsIdentity 型の Id があります。 GenericPrincipal クラスには、IPrincipal を実装するクラスで必要なベア最低限の機能が含まれています - Identity プロパティと、IsInRole メソッドだけがあります。

プリンシパル オブジェクトが 2 つの役割: ユーザーが所属するロールを示すと、id 情報を提供します。 IPrincipal インターフェイスの IsInRole を介して行われます (*roleName*) メソッドと Identity プロパティでは、それぞれします。 GenericPrincipal クラスでは、そのコンス トラクターを使用して指定するロール名の文字列配列を使用できます。その IsInRole (*roleName*) メソッドは単なるかどうかをチェック、渡されたで*roleName*文字列配列内に存在します。 FormsAuthenticationModule は、GenericPrincipal を作成するときに GenericPrincipal のコンス トラクターに空の文字列配列に渡します。 その結果、IsInRole への呼び出しは常に False を返します。

GenericPrincipal クラスは、ロールが使用されていないほとんどのフォーム ベースの認証シナリオのニーズを満たします。 既定の役割の処理が十分な場合にカスタム IIdentity オブジェクトをユーザーに関連付ける必要がある場合は、認証ワークフロー中に、カスタム IPrincipal オブジェクトを作成し、HttpContext.User プロパティに割り当てるか。

> [!NOTE]
> わかるとおり今後のチュートリアルでは、時に ASP します。NET の役割のフレームワークが有効になっている型のカスタム プリンシパル オブジェクトを作成します[RolePrincipal](https://msdn.microsoft.com/library/system.web.security.roleprincipal.aspx)とフォーム認証が作成した GenericPrincipal オブジェクトが上書きされます。 これは、プリンシパルの IsInRole メソッドの役割のフレームワークの API を使用したインターフェイスをカスタマイズするためにします。


私たちがいないに関して自分たちの役割を持つまだため唯一の理由がこの時点で、カスタム プリンシパルを作成するためになりますが、プリンシパルにカスタム IIdentity オブジェクトに関連付けることでしょう。 手順 4 でユーザーの会社名とそのタイトルで特定の認証チケットの UserData プロパティに追加のユーザー情報を格納しました。 ただし、UserData 情報は、認証チケットを使用してアクセスおよびとしてシリアル化された文字列、つまり、チケットに格納されているユーザー情報を表示するときにいつでも必要がある UserData プロパティを解析し、のみのみです。

IIdentity を実装し、CompanyName とタイトルのプロパティを含むクラスを作成して、開発者エクスペリエンスを向上させます。 そうすることは、開発者が、現在ログオンしているユーザーの会社名にアクセスできるし、UserData プロパティを解析する方法を理解するために必要なタイトルなし CompanyName とタイトルのプロパティを直接使用します。

### <a name="creating-the-custom-identity-and-principal-classes"></a>カスタム Id およびプリンシパル クラスを作成します。

このチュートリアルでは、アプリでみましょうカスタム プリンシパルと id オブジェクトを作成\_コード フォルダー。 アプリを追加して開始\_コード フォルダーをプロジェクトに、ソリューション エクスプ ローラーでプロジェクト名を右クリックして - ASP.NET フォルダーの追加オプションを選択、およびアプリの選択\_コード。 アプリ\_コード フォルダーは、web サイト固有のクラス ファイルを保持する特殊な ASP.NET フォルダーです。

> [!NOTE]
> アプリ\_コード フォルダーは、web サイト プロジェクト モデルを使用してプロジェクトを管理する場合にのみ使用する必要があります。 使用する場合、 [Web アプリケーション プロジェクト モデル](https://msdn.microsoft.com/asp.net/Aa336618.aspx)、標準的なフォルダーを作成し、クラスを追加します。 たとえば、クラスをという名前の新しいフォルダーを追加し、コードがありますに配置するでした。


次に、2 つの新しいクラス ファイルをアプリに追加\_CustomPrincipal.vb という名前のコードのフォルダー、1 つの名前付き CustomIdentity.vb および 1 つ。


[![CustomIdentity と CustomPrincipal クラスをプロジェクトに追加します。](forms-authentication-configuration-and-advanced-topics-vb/_static/image17.png)](forms-authentication-configuration-and-advanced-topics-vb/_static/image16.png)

**図 06**: CustomIdentity と CustomPrincipal クラスをプロジェクトに追加 ([フルサイズの画像を表示する をクリックします](forms-authentication-configuration-and-advanced-topics-vb/_static/image18.png))。


CustomIdentity クラスは、AuthenticationType、IsAuthenticated、および名前のプロパティを定義する IIdentity インターフェイスを実装する責任を負います。 これらの必要なプロパティだけでなく私たちは、基になるフォーム認証チケットだけでなく、ユーザーの会社名とタイトルのプロパティを公開します。 CustomIdentity クラスには、次のコードを入力します。

[!code-vb[Main](forms-authentication-configuration-and-advanced-topics-vb/samples/sample9.vb)]

クラスに所属メンバー変数が含まれることに注意してください (\_チケット) コンス トラクターを通じてこのチケット情報を渡す必要があるとします。 このチケット データは id の名前を返すときに使用します。CompanyName とタイトルのプロパティの値を返す、UserData プロパティが解析されます。

次に、CustomPrincipal クラスを作成します。 CustomPrincipal クラスのコンス トラクターが; CustomIdentity オブジェクトのみを受け入れるため、これは問題ありませんロールでこの時点で、その IsInRole メソッドは、常に False を返します。

[!code-vb[Main](forms-authentication-configuration-and-advanced-topics-vb/samples/sample10.vb)]

### <a name="assigning-a-customprincipal-object-to-the-incoming-requests-security-context"></a>受信要求のセキュリティ コンテキストに CustomPrincipal オブジェクトを割り当てる

カスタム id を使用するカスタム プリンシパル クラスと同様に、CompanyName とタイトルのプロパティを含める既定 IIdentity の仕様を拡張するクラスがあるようになりました。 ASP.NET パイプラインにステップ インする準備がお受信要求のセキュリティ コンテキストに、カスタム プリンシパル オブジェクトを割り当てます。

ASP.NET パイプラインは、受信要求を受け取り、ステップの数を処理します。 各手順で、特定イベントが発生する、開発者は ASP.NET 処理パイプラインをタップし、ライフ サイクルの特定の時点で、要求を変更できるようにします。 FormsAuthenticationModule は、発生させる ASP.NET などの待機、 [AuthenticateRequest イベント](https://msdn.microsoft.com/library/system.web.httpapplication.authenticaterequest.aspx)、この時点で、認証チケットを受信要求を検査します。 認証チケットが見つかった場合は、GenericPrincipal オブジェクトが作成され、HttpContext.User プロパティに割り当てられます。

AuthenticateRequest イベント後に ASP.NET パイプラインを発生させます、 [PostAuthenticateRequest イベント](https://msdn.microsoft.com/library/system.web.httpapplication.postauthenticaterequest.aspx)のインスタンスと FormsAuthenticationModule によって作成された GenericPrincipal オブジェクトを置換できますが、などの CustomPrincipal オブジェクト。 図 7 は、このワークフローを示しています。


[![GenericPrincipal は、CustomPrincipal PostAuthenticationRequest イベントに置き換え](forms-authentication-configuration-and-advanced-topics-vb/_static/image20.png)](forms-authentication-configuration-and-advanced-topics-vb/_static/image19.png)

**図 07**:、GenericPrincipal は、CustomPrincipal PostAuthenticationRequest イベントに置き換え ([フルサイズの画像を表示する をクリックします](forms-authentication-configuration-and-advanced-topics-vb/_static/image21.png))。


コードを実行するには、ASP.NET パイプライン イベントに応答するためには、Global.asax で適切なイベント ハンドラーを作成しますか、独自の HTTP モジュールを作成します。 このチュートリアルには、Global.asax でイベント ハンドラーを作成しましょう。 Global.asax を web サイトに追加することで開始します。 ソリューション エクスプ ローラーでプロジェクト名を右クリックし、Global.asax という、グローバル アプリケーション クラスの種類のアイテムを追加します。


[![Global.asax ファイル、web サイトを追加します。](forms-authentication-configuration-and-advanced-topics-vb/_static/image23.png)](forms-authentication-configuration-and-advanced-topics-vb/_static/image22.png)

**図 08**: Global.asax ファイルを web サイトに追加 ([フルサイズの画像を表示する をクリックします](forms-authentication-configuration-and-advanced-topics-vb/_static/image24.png))。


Global.asax の既定のテンプレートには、さまざまな開始日の終了を含む、ASP.NET パイプライン イベントのイベント ハンドラーが含まれています。 と[エラー イベント](https://msdn.microsoft.com/library/system.web.httpapplication.error.aspx)、他のユーザーの間で。 自由にこれらのイベント ハンドラーを削除するようにこのアプリケーションの必要ことはありません。 ここでは、イベントは PostAuthenticateRequest です。 そのマークアップは、次のようになりますので、Global.asax ファイルを更新します。

[!code-aspx[Main](forms-authentication-configuration-and-advanced-topics-vb/samples/sample11.aspx)]

アプリケーション\_OnPostAuthenticateRequest メソッドの実行のたびに、ASP.NET ランタイムは、受信した各ページ要求では、1 回、PostAuthenticateRequest イベントを発生させます。 イベント ハンドラーは、ユーザーが認証され、フォーム認証によって認証されたかどうかにどうかを確認して開始します。 そうである場合は、新しい CustomIdentity オブジェクトが作成され、現在の要求の認証チケットをコンス トラクターに渡されます。 次に、CustomPrincipal オブジェクトが作成され、作成したばかりの CustomIdentity オブジェクトをそのコンス トラクターに渡されました。 最後に、現在の要求のセキュリティ コンテキストは、新しく作成された CustomPrincipal オブジェクトに割り当てられます。

-要求のセキュリティ コンテキストと、などの CustomPrincipal オブジェクトの関連付け - 最後の手順が 2 つのプロパティをプリンシパルを割り当てることに注意してください: HttpContext.User と Thread.CurrentPrincipal します。 ASP.NET でのセキュリティ コンテキストの処理方法により、これら 2 つの割り当ては必要があります。 .NET Framework は、各実行中のスレッドをセキュリティ コンテキストを関連付けますこの情報は IPrincipal オブジェクトとして、[スレッド オブジェクト](https://msdn.microsoft.com/library/system.threading.thread.aspx)の[CurrentPrincipal プロパティ](https://msdn.microsoft.com/library/system.threading.thread.currentcontext.aspx)します。 ASP.NET には、独自のセキュリティ コンテキスト情報 (HttpContext.User) は、混乱を招くです。

セキュリティ コンテキストを決定するときに Thread.CurrentPrincipal プロパティを調べて特定のシナリオで他のシナリオでは、HttpContext.User が使用されます。 たとえば、.NET の開発者は宣言によってどのようなユーザーの状態を許可するセキュリティ機能があるまたはロールのクラスのインスタンスを作成または特定のメソッドを呼び出すことができます (を参照してください[ビジネス レイヤーを使用してデータを承認規則の追加PrincipalPermissionAttributes](https://weblogs.asp.net/scottgu/archive/2006/10/04/Tip_2F00_Trick_3A00_-Adding-Authorization-Rules-to-Business-and-Data-Layers-using-PrincipalPermissionAttributes.aspx))。 実際には、下には、これらの宣言型の手法は、Thread.CurrentPrincipal プロパティを使用してセキュリティ コンテキストを決定します。

他のシナリオでは、HttpContext.User プロパティが使用されます。 たとえば、前のチュートリアルでは、現在ログオンしているユーザーのユーザー名を表示するこのプロパティを使いました。 明らかに、次は Thread.CurrentPrincipal と HttpContext.User プロパティでセキュリティ コンテキスト情報が一致します。

ASP.NET ランタイムは、私たちにとって、これらのプロパティ値を自動的に反映されます。 ただし、この同期は、AuthenticateRequest イベント後に発生しますが、*する前に*PostAuthenticateRequest イベント。 その結果、PostAuthenticateRequest イベントでカスタム プリンシパルを追加するときにあるものを Thread.CurrentPrincipal ではそう Thread.CurrentPrincipal を手動で割り当てる必要があり、HttpContext.User が同期されます。参照してください[Context.User vs します。Thread.CurrentPrincipal](http://leastprivilege.com/2005/11/23/context-user-vs-thread-currentprincipal/)の詳細については、この問題にします。

### <a name="accessing-the-companyname-and-title-properties"></a>CompanyName とタイトルのプロパティにアクセスします。

要求が到着して、ASP.NET エンジンをアプリケーションにディスパッチされますたびに\_Global.asax で OnPostAuthenticateRequest イベント ハンドラーが起動されます。 要求が、FormsAuthenticationModule で正常に認証されている場合、イベント ハンドラーは、フォーム認証チケットに基づく CustomIdentity オブジェクトに新しい CustomPrincipal オブジェクトを作成します。 このロジックで、現在ログオンしているユーザーの会社名とタイトルに関する情報へのアクセスは、非常に簡単です。

ページに戻る\_フォーム認証チケットを取得し、ユーザーの会社名とタイトルを表示するには、UserData プロパティを解析するコードが作成した手順 4. で、Default.aspx の Load イベント ハンドラー。 使用を今すぐ CustomPrincipal および CustomIdentity オブジェクトでは、チケットの UserData プロパティから値を解析する必要はありません。 代わりに、単に CustomIdentity オブジェクトへの参照を取得し、CompanyName とタイトルのプロパティを使用します。

[!code-vb[Main](forms-authentication-configuration-and-advanced-topics-vb/samples/sample12.vb)]

## <a name="summary"></a>まとめ

このチュートリアルでは、Web.config を使用して、フォーム認証システムの設定をカスタマイズする方法について確認しました。認証チケットの有効期限の処理方法と、暗号化と検証の保護を使用してチケットを検査および変更から保護する方法について説明しました。 最後に、認証チケットの UserData プロパティを使用してチケット自体に追加のユーザー情報とカスタム プリンシパルと id オブジェクトを使用して、複数の開発者向けの方法でこの情報を公開する方法を格納するについて説明します。

このチュートリアルでは、ASP.NET フォーム認証、調査を終了します。 次のチュートリアルでは、メンバーシップ フレームワークに旅を開始します。

満足のプログラミングです。

### <a name="further-reading"></a>関連項目

このチュートリアルで説明したトピックの詳細については、次の情報を参照してください。

- [フォーム認証の詳細に分析します。](http://aspnet.4guysfromrolla.com/articles/072005-1.aspx)
- [ASP.NET 2.0 でのフォーム認証について説明します。](https://msdn.microsoft.com/library/aa480476.aspx)
- [方法: ASP.NET 2.0 でフォーム認証を保護します。](https://msdn.microsoft.com/library/ms998310.aspx)
- [Professional ASP.NET 2.0 のセキュリティ、メンバーシップ、およびロール管理](http://www.wrox.com/WileyCDA/WroxTitle/productCd-0764596985.html)(ISBN: 978-0-7645-9698-8)
- [ログイン コントロールをセキュリティで保護します。](https://msdn.microsoft.com/library/ms178346.aspx)
- [<authentication>要素](https://msdn.microsoft.com/library/532aee0e.aspx)
- [&lt;フォーム&gt;要素<authentication>](https://msdn.microsoft.com/library/1d3t3c61.aspx)
- [&lt;MachineKey&gt;要素](https://msdn.microsoft.com/library/w8h3skw9.aspx)
- [フォーム認証チケット Cookie を理解します。](https://support.microsoft.com/kb/910443)

### <a name="video-training-on-topics-contained-in-this-tutorial"></a>このチュートリアルに含まれるトピックのビデオ トレーニング

- [フォーム認証プロパティを変更する方法](../../../videos/authentication/how-to-change-the-forms-authentication-properties.md)
- [ASP.NET アプリケーションで Cookie なしの認証をセットアップおよび使用する方法](../../../videos/authentication/how-to-setup-and-use-cookie-less-authentication-in-an-aspnet-application.md)
- [ASP フォーム ログイン再配置](../../../videos/authentication/asp-forms-login-relocation.md)
- [フォーム ログイン カスタム キー構成](../../../videos/authentication/forms-login-custom-key-configuration.md)
- [認証方法にカスタム データを追加する](../../../videos/authentication/add-custom-data-to-the-authentication-method.md)
- [カスタム プリンシパル オブジェクトを使用する](../../../videos/authentication/use-custom-principal-objects.md)

### <a name="about-the-author"></a>執筆者紹介

Scott Mitchell、複数の受け取ります書籍の著者と、4GuysFromRolla.com の創設者では、1998 年から、Microsoft Web テクノロジに取り組んできました。 Scott は、フリーのコンサルタント、トレーナー、およびライターとして動作します。 最新の著書は *[Sams 教える自分で ASP.NET 2.0 24 時間以内に](https://www.amazon.com/exec/obidos/ASIN/0672327384/4guysfromrollaco)* します。 Scott に到達できる[ mitchell@4guysfromrolla.com ](mailto:mitchell@4guysfromrolla.com)または彼のブログ[ http://ScottOnWriting.NET](http://scottonwriting.net/)します。

### <a name="special-thanks-to"></a>特別なに感謝します。

このチュートリアル シリーズは、多くの便利なレビュー担当者によってレビューされました。 このチュートリアルでは、潜在顧客レビュー担当者は、Alicja Maziarz でした。 今後、MSDN の記事を確認したいですか。 場合は、筆者に[ mitchell@4GuysFromRolla.com](mailto:mitchell@4guysfromrolla.com)します。

> [!div class="step-by-step"]
> [前へ](an-overview-of-forms-authentication-vb.md)
