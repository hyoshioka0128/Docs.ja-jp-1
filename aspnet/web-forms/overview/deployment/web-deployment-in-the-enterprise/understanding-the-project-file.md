---
uid: web-forms/overview/deployment/web-deployment-in-the-enterprise/understanding-the-project-file
title: プロジェクト ファイルを理解する |Microsoft Docs
author: jrjlee
description: Microsoft Build Engine (MSBuild) プロジェクト ファイルは、ビルドおよび配置プロセスの中核にあります。 このトピックでは、MSBuild の概念の概要を開始しています.
ms.author: riande
ms.date: 05/04/2012
ms.assetid: 07978d9d-341c-4524-bcba-62976f390f77
msc.legacyurl: /web-forms/overview/deployment/web-deployment-in-the-enterprise/understanding-the-project-file
msc.type: authoredcontent
ms.openlocfilehash: 114dd21002ef41627f3a101c0197a85fd5208887
ms.sourcegitcommit: 45ac74e400f9f2b7dbded66297730f6f14a4eb25
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/16/2018
ms.locfileid: "41826888"
---
<a name="understanding-the-project-file"></a>プロジェクト ファイルを理解します。
====================
によって[Jason Lee](https://github.com/jrjlee)

[PDF のダウンロード](https://msdnshared.blob.core.windows.net/media/MSDNBlogsFS/prod.evol.blogs.msdn.com/CommunityServer.Blogs.Components.WeblogFiles/00/00/00/63/56/8130.DeployingWebAppsInEnterpriseScenarios.pdf)

> Microsoft Build Engine (MSBuild) プロジェクト ファイルは、ビルドおよび配置プロセスの中核にあります。 このトピックでは、MSBuild とプロジェクト ファイルの概念の概要から始まります。 これには、プロジェクトのファイルを使用して、プロジェクト ファイルを使用して、実際のアプリケーションをデプロイする方法の例を使用して動作時に遭遇するが、主要なコンポーネントについて説明します。
> 
> 学習内容。
> 
> - MSBuild がプロジェクトをビルドするのに MSBuild プロジェクト ファイルを使用方法。
> - 方法、MSBuild は、インターネット インフォメーション サービス (IIS) Web 配置ツール (Web 配置) などの展開テクノロジと統合します。
> - プロジェクト ファイルの主要なコンポーネントを理解する方法。
> - どのプロジェクト ファイルを使用して、複雑なアプリケーションをビルドおよびデプロイできます。


## <a name="msbuild-and-the-project-file"></a>MSBuild とプロジェクト ファイル

作成する Visual Studio でソリューションを構築すると、Visual Studio は、ソリューション内の各プロジェクトをビルドするのに MSBuild を使用します。 すべての Visual Studio プロジェクトには、プロジェクトの種類を反映するファイル拡張子を持つ、MSBuild プロジェクト ファイルが含まれています。&#x2014;C# プロジェクト (.csproj)、Visual basic.net プロジェクト (.vbproj)、またはデータベース プロジェクト (.dbproj)。 プロジェクトをビルドするために、MSBuild はプロジェクトに関連付けられているプロジェクト ファイルを処理する必要があります。 プロジェクト ファイルは、すべての情報と MSBuild がコンテンツを含めるには、プラットフォームの要件、バージョン管理情報、web サーバーまたはデータベース サーバーの設定など、プロジェクトをビルドするために必要な手順を含む XML ドキュメントでは、実行する必要があるタスク。

MSBuild プロジェクト ファイルがに基づいて、 [MSBuild XML スキーマ](https://msdn.microsoft.com/library/5dy88c2e.aspx)、その結果、ビルド プロセスは完全にオープンで透明とします。 さらに、MSBuild エンジンを使用するには、Visual Studio をインストールする必要はありません&#x2014;MSBuild.exe の実行可能ファイルは、.NET Framework の一部と、コマンド プロンプトから行うことができます。 開発者には、MSBuild XML スキーマを使用して、プロジェクトをビルドおよび配置する方法を洗練されたきめ細かな制御を強制する独自 MSBuild プロジェクト ファイルを作成できます。 これらのカスタム プロジェクト ファイルは、Visual Studio によって自動的に生成されるプロジェクト ファイルとまったく同じ方法で動作します。

> [!NOTE]
> Team Build サービスでは、Team Foundation Server (TFS) と MSBuild プロジェクト ファイルを使用することもできます。 たとえば、継続的インテグレーション (CI) のシナリオでのプロジェクト ファイルを使用して、新しいコードをチェックインするときに、テスト環境にデプロイを自動化することができます。 詳細については、次を参照してください。 [Web デプロイの自動化用の Team Foundation Server の構成](../configuring-team-foundation-server-for-web-deployment/configuring-team-foundation-server-for-web-deployment.md)します。


### <a name="project-file-naming-conventions"></a>プロジェクト ファイルの名前付け規則

プロジェクト ファイルを作成するときに、希望する任意のファイル拡張子を使用できます。 ただし、お客様のソリューションを他のユーザーが理解しやすくするには、これらの一般的な規則を使用する必要があります。

- プロジェクトをビルドするプロジェクト ファイルを作成すると、.proj 拡張機能を使用します。
- 他のプロジェクト ファイルをインポートする再利用可能なプロジェクト ファイルを作成するときに .targets 拡張子を使用します。 .Targets 拡張子を持つファイル通常しない何でも構築自体、単に、.proj ファイルにインポートできる手順が含まれます。

### <a name="integration-with-deployment-technologies"></a>展開テクノロジとの統合

ASP.NET web アプリケーションおよび ASP.NET MVC web アプリケーションのように、Visual Studio 2010 で web アプリケーション プロジェクトで作業したことがある場合は、これらのプロジェクトにパッケージ化と配置ターゲット環境に web アプリケーションの組み込みサポートが含まれていることがわかります。 **プロパティ**これらのプロジェクトに含まれているページ**パッケージ化/発行 Web**と**パッケージ化/発行 SQL**構成に使用できるタブ方法のコンポーネント、アプリケーションをパッケージ化され、デプロイします。 これを示しています、**パッケージ化/発行 Web**  タブ。

![](understanding-the-project-file/_static/image1.png)

これらの機能の背後にある基になるテクノロジは、Web 発行パイプライン (WPP) と呼ばれます。 基本的には、MSBuild、WPP と[Web Deploy](https://go.microsoft.com/?linkid=9805122) web アプリケーションの完全なビルド、パッケージ、およびデプロイ プロセスを提供します。

良い知らせは、web プロジェクト用のカスタム プロジェクト ファイルを作成するときに、WPP が提供する統合ポイントの利用を行うことができます。 プロジェクトをビルドし、web デプロイ パッケージを作成、およびリモート サーバー上で 1 つのプロジェクト ファイルと MSBuild への 1 回の呼び出しを通じてこれらのパッケージをインストールすることができるプロジェクト ファイルでは、デプロイの手順を含めることができます。 実行可能ファイル、ビルド プロセスの一部として呼び出すこともできます。 たとえば、スキーマ ファイルからデータベースを配置する VSDBCMD.exe コマンド ライン ツールを実行できます。 このトピックの過程で、エンタープライズ展開シナリオの要件を満たすためにこれらの機能を活用を実行する方法が表示されます。

> [!NOTE]
> Web アプリケーションの展開プロセスのしくみの詳細については、次を参照してください。 [ASP.NET Web アプリケーション プロジェクトの配置の概要](https://msdn.microsoft.com/library/dd394698.aspx)します。


## <a name="the-anatomy-of-a-project-file"></a>プロジェクト ファイルの構造

ビルド プロセスの詳細を見ると、前に、MSBuild プロジェクト ファイルの基本的な構造を理解して、わずかな時間をかけて価値です。 このセクションでは、確認、編集、またはプロジェクト ファイルを作成するときに発生する一般的な要素の概要を示します。 具体的には、学習内容。

- 使用する*プロパティ*変数のビルド プロセスを管理します。
- 使用する*項目*などのコード ファイルのビルド プロセスへの入力を識別するためにします。
- 使用する*ターゲット*と*タスク*msbuild、実行手順を提供するを使用して*プロパティ*と*項目*で定義されています。プロジェクト ファイルです。

これは、MSBuild プロジェクト ファイルの主な要素間の関係を示します。

![](understanding-the-project-file/_static/image2.png)

### <a name="the-project-element"></a>Project 要素

[プロジェクト](https://msdn.microsoft.com/library/bcxfsh87.aspx)要素はすべてのプロジェクト ファイルのルート要素です。 プロジェクト ファイルの XML スキーマを識別するだけでなく、**プロジェクト**要素は、ビルド プロセスのエントリ ポイントを指定する属性を含めることができます。 たとえば、 [Contact Manager サンプル ソリューション](the-contact-manager-solution.md)、 *Publish.proj*という名前のターゲットを呼び出すことによって、ビルドを開始するファイルを指定**FullPublish**します。


[!code-xml[Main](understanding-the-project-file/samples/sample1.xml)]


### <a name="properties-and-conditions"></a>プロパティと条件

通常、プロジェクト ファイルは、多数のさまざまな正常にビルドし、プロジェクトを配置するために情報を提供する必要があります。 これらの情報には、サーバー名、接続文字列、資格情報、ビルド構成、ソースと変換先ファイルのパス、およびカスタマイズをサポートするために追加するその他の情報を含めることができます。 内で、プロジェクト ファイルのプロパティを定義する必要があります、 [PropertyGroup](https://msdn.microsoft.com/library/t4w159bs.aspx)要素。 MSBuild プロパティは、キーと値のペアで構成されます。 内で、 **PropertyGroup**要素、要素名は、プロパティのキーを定義し、要素のコンテンツ プロパティの値を定義します。 たとえば、という名前のプロパティを定義できます**ServerName**と**ConnectionString**静的サーバー名と接続文字列を格納します。


[!code-xml[Main](understanding-the-project-file/samples/sample2.xml)]


形式を使用するプロパティの値を取得する<strong>$(</strong><em>PropertyName</em><strong>)</strong><em>.</em>たとえばの値を取得するため、 <strong>ServerName</strong>入力プロパティ。


[!code-powershell[Main](understanding-the-project-file/samples/sample3.ps1)]


> [!NOTE]
> 方法の例と使用すると、プロパティ値を使用して、このトピックの後半を確認します。


プロジェクト ファイルで静的なプロパティとして情報を埋め込むは常に、ビルド プロセスを管理するための理想的なアプローチです。 多数のシナリオでは、他のソースから情報を取得したり、コマンド プロンプトからの情報を提供するユーザーを支援します。 MSBuild は、コマンド ライン パラメーターとしてのプロパティの値を指定することができます。 たとえば、ユーザーが値を指定でした**ServerName**ときに実行するとき MSBuild.exe コマンドラインから。


[!code-console[Main](understanding-the-project-file/samples/sample4.cmd)]


> [!NOTE]
> MSBuild.exe で使用できるスイッチと引数の詳細については、次を参照してください。 [MSBuild コマンド ライン リファレンス](https://msdn.microsoft.com/library/ms164311.aspx)します。


環境変数と組み込みのプロジェクトのプロパティの値を取得するのにプロパティのと同じ構文を使用することができます。 一般的に使用されるプロパティの多くが、定義されているし、する使用プロジェクト ファイルで、関連するパラメーター名を含めることで。 たとえば、現在のプロジェクト プラットフォームを取得する&#x2014;など **x86** または **AnyCpu**&#x2014;含めることができます、 **$(Platform)** プロパティ参照でプロジェクト ファイルです。 詳細については、次を参照してください。[ビルドのコマンドとプロパティのマクロの](https://msdn.microsoft.com/library/c02as0cs.aspx)、 [MSBuild プロジェクトの共通プロパティ](https://msdn.microsoft.com/library/bb629394.aspx)、および[の予約済みプロパティ](https://msdn.microsoft.com/library/ms164309.aspx)します。

プロパティと組み合わせてよく*条件*します。 ほとんどの MSBuild 要素をサポートして、**条件**属性には、MSBuild が要素を評価する条件を指定することができます。 たとえば、このプロパティの定義があるとします。


[!code-xml[Main](understanding-the-project-file/samples/sample5.xml)]


MSBuild では、このプロパティの定義を処理するとき、まず確認するかどうか、 **$(OutputRoot)** プロパティの値が使用可能な。 プロパティの値が空白の場合&#x2014;つまり、ユーザーは、このプロパティの値を指定していない&#x2014;条件の評価が **true** にプロパティの値を設定および **.\Publish\Out** です。条件の評価が場合は、ユーザーには、このプロパティの値が提供された、 **false**静的プロパティの値は使用されません。

条件を指定するさまざまな方法の詳細については、次を参照してください。 [MSBuild の条件](https://msdn.microsoft.com/library/7szfhaft.aspx)します。

### <a name="items-and-item-groups"></a>項目と項目グループ

プロジェクト ファイルの重要な役割の 1 つは、ビルド プロセスへの入力を定義します。 通常、ファイルは、これらの入力&#x2014;ファイル、構成ファイル、コマンド ファイルは、処理またはとしてコピーする必要があるその他のファイルのコード、ビルド プロセスの一部です。 MSBuild プロジェクトのスキーマでこれらの入力がによって表される[項目](https://msdn.microsoft.com/library/ms164283.aspx)要素。 内でプロジェクト ファイルで項目を定義する必要があります、 [ItemGroup](https://msdn.microsoft.com/library/646dk05y.aspx)要素。 同じように**プロパティ**名前を要素、**項目**要素自由です。 ただし、指定する必要があります、 **Include**ファイルまたは項目を表すワイルドカードを識別する属性。


[!code-xml[Main](understanding-the-project-file/samples/sample6.xml)]


複数を指定することによって**項目**と同じ名前の要素を効果的に作成するリソースの名前付きのリスト。 この動作を確認する良い方法は、Visual Studio で作成するプロジェクト ファイルのいずれかを確認します。 たとえば、 *ContactManager.Mvc.csproj*サンプル ソリューション内のファイルには、それぞれに同じ名前のいくつかの項目グループの多くが含まれる**項目**要素。


[!code-xml[Main](understanding-the-project-file/samples/sample7.xml)]


この方法で、プロジェクト ファイルのと同じ方法で処理する必要があるファイルのリストを構築するために MSBuild がよう指示&#x2014;、**参照**成功したビルドの場所にする必要があるアセンブリが一覧に含まれています、 **コンパイル**リストにはコンパイルする必要があります、コード ファイルが含まれていますと**コンテンツ**一覧にそのままコピーする必要があるリソースが含まれています。 ビルド プロセスの参照し、これらの項目を使用して、このトピックの後半で紹介します。

項目の要素を含めることも[ItemMetadata](https://msdn.microsoft.com/library/ms164284.aspx)子要素。 これらはユーザー定義のキー/値ペアであり、基本的にその項目に固有のプロパティを表します。 たとえば、多くの**コンパイル**項目要素をプロジェクト ファイルには、 **DependentUpon**子要素。


[!code-xml[Main](understanding-the-project-file/samples/sample8.xml)]


> [!NOTE]
> ユーザーが作成した項目のメタデータだけでなく、すべての項目には作成時にさまざまな一般的なメタデータが割り当てられます。 詳細については、「[既知の項目メタデータ](https://msdn.microsoft.com/library/ms164313.aspx)」を参照してください。


作成することができます**ItemGroup**ルート レベル内の要素**プロジェクト**要素、または特定の**ターゲット**要素。 **ItemGroup**要素もサポート**条件**属性で、プロジェクトの構成またはプラットフォームのような条件に従って、ビルド プロセスへの入力を調整することができます。

### <a name="targets-and-tasks"></a>ターゲットとタスク

MSBuild のスキーマで、[タスク](https://msdn.microsoft.com/library/77f2hx1s.aspx)要素は、個々 のビルド命令 (またはタスク) を表します。 MSBuild には、さまざまな定義済みのタスクが含まれています。 例えば:

- **コピー**タスクは、新しい場所にファイルをコピーします。
- **Csc**タスクは、Visual C# コンパイラを起動します。
- **Vbc**タスクは、Visual Basic コンパイラを起動します。
- **Exec**タスクは、指定したプログラムを実行します。
- **メッセージ**タスクでは、ロガーにメッセージを書き込みます。

> [!NOTE]
> すぐに使用できるタスクの詳細については、次を参照してください。 [MSBuild タスク リファレンス](https://msdn.microsoft.com/library/7z253716.aspx)します。 カスタムのタスクを作成する方法など、タスクの詳細については、次を参照してください。 [MSBuild タスク](https://msdn.microsoft.com/library/ms171466.aspx)します。


内のタスクを含めることが常に必要がある[ターゲット](https://msdn.microsoft.com/library/t50z2hka.aspx)要素。 A**ターゲット**要素は、一連の順番に実行される 1 つまたは複数のタスクとプロジェクト ファイルは、複数のターゲットを含めることができます。 タスクまたは一連のタスクを実行する場合は、それらを含むターゲットを呼び出します。 たとえば、メッセージをログに記録する単純なプロジェクト ファイルがあるとします。


[!code-xml[Main](understanding-the-project-file/samples/sample9.xml)]


使用して、コマンドラインからターゲットを呼び出すことができます、 **/t**ターゲットを指定するスイッチ。


[!code-console[Main](understanding-the-project-file/samples/sample10.cmd)]


また、追加することができます、 **DefaultTargets**属性を**プロジェクト**を起動するターゲットを指定する要素。


[!code-xml[Main](understanding-the-project-file/samples/sample11.xml)]


この場合は、コマンドラインからターゲットを指定する必要はありません。 プロジェクト ファイルを指定するだけで、MSBuild が呼び出す、 **FullPublish**するターゲット。


[!code-console[Main](understanding-the-project-file/samples/sample12.cmd)]


ターゲットとタスクの両方を含めることができます**条件**属性。 そのため、特定の条件が満たされた場合に、ターゲットの全体または個々 のタスクを省略することができます。

一般に、便利なタスクとターゲットを作成するときに、プロパティと他の場所で、プロジェクト ファイルで定義した項目を参照する必要があります。

- プロパティ値を使用する入力<strong>$(</strong><em>PropertyName</em><strong>)</strong>ここで、 <em>PropertyName</em>の名前を指定します、 <strong>プロパティ</strong>要素またはパラメーターの名前。
- 項目を使用する入力<strong>@(</strong><em>ItemName</em><strong>)</strong>ここで、 <em>ItemName</em>の名前を指定します、<strong>項目</strong>要素。

> [!NOTE]
> 同じ名前の複数の項目を作成する場合は、一覧を構築していることを覚えておいてください。 これに対し、同じ名前の複数のプロパティを作成する場合は、最後のプロパティ値を指定するプロパティが上書きされます前と同じ名前&#x2014;プロパティがのみ 1 つの値を含めることができます。


たとえば、 *Publish.proj*サンプル ソリューションでファイルを見て、 **BuildProjects**ターゲット。


[!code-xml[Main](understanding-the-project-file/samples/sample13.xml)]


このサンプルでは、次の点を確認できます。

- 場合、 **BuildingInTeamBuild**パラメーターが指定されておりの値を持つ**true**、このターゲット内のタスクのいずれも実行されます。
- ターゲットには 1 つのインスタンスが含まれています、 [MSBuild](https://msdn.microsoft.com/library/z7f65y0d.aspx)タスク。 このタスクでは、他の MSBuild プロジェクトをビルドすることができます。
- **ProjectsToBuild**項目は、タスクに渡されます。 この項目はプロジェクトまたはソリューション ファイル、によって定義されたすべての一覧を表すことができます**ProjectsToBuild**項目の項目グループ内の要素。 ここで、 **ProjectsToBuild**項目が 1 つのソリューション ファイルを参照します。

    [!code-xml[Main](understanding-the-project-file/samples/sample14.xml)]
- 渡されるプロパティの値、 **MSBuild**タスクという名前のパラメーターに含める**OutputRoot**と**構成**します。 これらは、それ以外の場合に、パラメーターの値が指定された場合または静的プロパティの値に設定されます。

    [!code-xml[Main](understanding-the-project-file/samples/sample15.xml)]

表示することも、 **MSBuild**タスクという名前のターゲットを呼び出す**ビルド**します。 これは広く使用されている Visual Studio プロジェクト ファイルでは、カスタムのプロジェクト ファイルで使用できるようないくつかの組み込みのターゲットの 1 つ**ビルド**、**クリーン**、 **を再構築**、および**発行**します。 ターゲットとし、ビルド プロセスの制御やに関するタスクの使用方法の詳細を学習、 **MSBuild**タスクの具体的には、このトピックで後述します。

> [!NOTE]
> ターゲットの詳細については、次を参照してください。 [MSBuild ターゲット](https://msdn.microsoft.com/library/ms171462.aspx)します。


## <a name="splitting-project-files-to-support-multiple-environments"></a>複数の環境をサポートするためのプロジェクト ファイルの分割

ソリューションをテスト サーバー、ステージング プラットフォームは、運用環境など、複数の環境に配置したいとします。 これらの環境間で、構成が大幅に異なる場合があります&#x2014;だけでなくサーバー名、接続文字列、およびの観点からも資格情報、セキュリティの設定、およびその他の要因の多くの観点から可能性があります。 これを定期的に実行する必要がある場合、ターゲット環境を切り替えるたびにプロジェクト ファイル内の複数のプロパティを編集する非常に便利ではありません。 また理想的なソリューションをビルド プロセスに提供するプロパティの値の果てしないリストを必要とすることはありません。

さいわい別の方法があります。 MSBuild では、複数のプロジェクト ファイル、ビルド構成を分割することができます。 動作を確認するこの、サンプルのソリューションでは、2 つのカスタム プロジェクト ファイルがあることに注意してください。

- *Publish.proj*プロパティ、項目が含まれているターゲットはすべての環境に共通します。
- *Env Dev.proj*開発者の環境に固有のプロパティを含むです。

ようになりましたことに注意して、 *Publish.proj*ファイルが含まれています、[インポート](https://msdn.microsoft.com/library/92x05xfs.aspx)開始のすぐ下にある要素、**プロジェクト**タグ。


[!code-xml[Main](understanding-the-project-file/samples/sample16.xml)]


**インポート**要素を使用して、現在の MSBuild プロジェクト ファイルを別の MSBuild プロジェクト ファイルの内容をインポートします。 ここで、 **TargetEnvPropsFile**パラメーターをインポートするプロジェクト ファイルのファイル名を提供します。 MSBuild を実行すると、このパラメーターの値を行うことができます。


[!code-console[Main](understanding-the-project-file/samples/sample17.cmd)]


実質的に、これは、1 つのプロジェクト ファイルに 2 つのファイルの内容をマージします。 このアプローチを使用して、汎用のビルド構成を含む 1 つのプロジェクト ファイルと環境固有のプロパティを格納している複数の補助プロジェクト ファイルを作成できます。 その結果、異なるパラメーター値を持つコマンドを実行するだけできますソリューションを別の環境に配置します。

![](understanding-the-project-file/_static/image3.png)

この方法で、プロジェクト ファイルの分割に従うことをお勧めします。 複数のプロジェクト ファイルに汎用のビルド プロパティの重複を回避しながら 1 つのコマンドを実行して複数の環境にデプロイすることができます。

> [!NOTE]
> 独自のサーバー環境の環境に固有のプロジェクト ファイルをカスタマイズする方法のガイダンスについては、次を参照してください。[ターゲット環境の配置プロパティを構成する](../configuring-server-environments-for-web-deployment/configuring-deployment-properties-for-a-target-environment.md)します。


## <a name="conclusion"></a>まとめ

このトピックでは、MSBuild プロジェクト ファイルの概要を提供し、ビルド プロセスを制御する独自のカスタム プロジェクト ファイルを作成する方法について説明します。 ユニバーサルのビルド手順と環境固有のビルド プロパティ、ビルドし、複数の変換先にプロジェクトをデプロイするが簡単にプロジェクト ファイルに分割の概念も導入されました。

次のトピックでは、[ビルド プロセスを理解する](understanding-the-build-process.md)複雑さのレベルが現実的なソリューションのデプロイを検討するコントロールのビルドおよび配置するプロジェクト ファイルを使用する方法の詳細について洞察を提供します。

## <a name="further-reading"></a>関連項目

プロジェクト ファイルと、WPP をより詳細な概要については、次を参照してください。[内で、Microsoft Build Engine: Using MSBuild and Team Foundation ビルド](http://amzn.com/0735645248)Sayed Ibrahim Hashimi、William Bartholomew、ISBN: 978-0-7356-4524-0。

> [!div class="step-by-step"]
> [前へ](setting-up-the-contact-manager-solution.md)
> [次へ](understanding-the-build-process.md)
