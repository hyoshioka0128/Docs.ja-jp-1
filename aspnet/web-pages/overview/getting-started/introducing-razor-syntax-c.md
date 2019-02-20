---
uid: web-pages/overview/getting-started/introducing-razor-syntax-c
title: Razor 構文 (C#) を使用して ASP.NET Web プログラミングの概要 |Microsoft Docs
author: Rick-Anderson
description: この章では、するプログラミングの概要 ASP.NET Web ページ Razor 構文を使用します。 動的な web の pa を実行するためのマイクロソフトのテクノロジを ASP.NET には.
ms.author: riande
ms.date: 02/07/2014
ms.assetid: aa67d304-583b-4bf8-a231-195656cfb587
msc.legacyurl: /web-pages/overview/getting-started/introducing-razor-syntax-c
msc.type: authoredcontent
ms.openlocfilehash: b5eb98dfdf3fc013920f45080d4a20e1fa507725
ms.sourcegitcommit: 2d3e5422d530203efdaf2014d1d7df31f88d08d0
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/05/2018
ms.locfileid: "51021782"
---
<a name="introduction-to-aspnet-web-programming-using-the-razor-syntax-c"></a>Razor 構文 (C#) を使用して ASP.NET Web プログラミングの概要
====================
によって[Tom FitzMacken](https://github.com/tfitzmac)

> この記事では、プログラミングの概要 ASP.NET Web ページ Razor 構文を使用します。 ASP.NET は、web サーバー上で動的な web ページを実行するためのマイクロソフトのテクノロジです。 この記事を使用して C# のプログラミング言語での焦点を当てます。
> 
> **学習内容**:
> 
> - プログラミングの Razor 構文を使用して ASP.NET Web Pages のプログラミングの概要に関するヒント上位 8。
> - 基本的なプログラミング概念必要があります。
> - どのような ASP.NET サーバー コードと Razor 構文についてです。
>   
> 
> ## <a name="software-versions"></a>ソフトウェアのバージョン
> 
> 
> - ASP.NET Web Pages (Razor) 3
>   
> 
> このチュートリアルは、ASP.NET Web Pages 2 でも機能します。


## <a name="the-top-8-programming-tips"></a>最上位の 8 つのプログラミングのヒント

このセクションでは、Razor 構文を使用して ASP.NET サーバー コードの記述を開始する際に知っておく必要ないくつかのヒントを示します。

> [!NOTE]
> Razor 構文は C# プログラミング言語に基づいており、ASP.NET Web Pages を最もよく使用されている言語です。 ただし、Razor 構文は、Visual Basic 言語と Visual Basic で実行することもできます。 表示されるものをもサポートします。 詳細については、「付録」を参照してください。 [Visual Basic 言語と構文](https://go.microsoft.com/fwlink/?LinkId=202908)します。


記事の後半でこれらのプログラミング手法のほとんどについての詳細が表示されます。

### <a name="1-you-add-code-to-a-page-using-the--character"></a>1.コードを追加するページを使用して、@ 文字

`@文字は、インライン式、単一文のブロック、複文のブロックの始まりを表します。

[!code-html[Main](introducing-razor-syntax-c/samples/sample1.html)]

これは、これらのステートメントがどのようなページがブラウザーで実行するとします。

![Razor-Img1](introducing-razor-syntax-c/_static/image1.jpg)

> [!TIP] 
> 
> **HTML エンコード**
> 
> ページを使用して、コンテンツを表示すると、 `@` ASP.NET HTML エンコードの出力を前の例のように文字します。 これにより、HTML の予約文字が置き換えられます (など`<`と`>`と`&`) の HTML タグまたはエンティティとして解釈されるのではなく、web ページ内の文字として表示される文字を有効にするコードを持つ。 HTML エンコードをせずサーバー コードからの出力は正しく表示されない場合があり、セキュリティのリスクについてのページを公開する可能性があります。
> 
> 目的がマークアップとしてタグをレンダリングする HTML マークアップを出力するかどうか (たとえば`<p></p>`段落のまたは`<em></em>`テキストを強調するために) を参照してください[を組み合わせることのテキスト、マークアップ、およびコード ブロック内のコード](#BM_CombiningTextMarkupAndCode)この記事で後述します。
> 
> 詳細について、HTML でのエンコード[フォームを使用する](https://go.microsoft.com/fwlink/?LinkId=202892)。


### <a name="2-you-enclose-code-blocks-in-braces"></a>2.コード ブロックを中かっこで囲むこと

A*コード ブロック*1 つまたは複数のコード ステートメントを含み、中かっこで囲まれました。

[!code-html[Main](introducing-razor-syntax-c/samples/sample2.html)]

ブラウザーで表示される結果:

![Razor-Img2](introducing-razor-syntax-c/_static/image2.jpg)

### <a name="3-inside-a-block-you-end-each-code-statement-with-a-semicolon"></a>3.セミコロンで各コード ステートメントを終了する、ブロックの内部

コード ブロック内では、完全なコードの各ステートメントはセミコロンで終了しなければなりません。 インライン式は、セミコロンで終了しません。

[!code-html[Main](introducing-razor-syntax-c/samples/sample3.html)]

### <a name="4-you-use-variables-to-store-values"></a>4.値を格納するのに変数を使用します。

内の値を格納することができます、*変数*文字列、数字、および日付などを含め、します。新しい変数を使用して、作成する、`var`キーワード。 変数の値を挿入するにを使用してページに直接`@`します。

[!code-html[Main](introducing-razor-syntax-c/samples/sample4.html)]

ブラウザーで表示される結果:

![Razor-Img3](introducing-razor-syntax-c/_static/image3.jpg)

<a id="ID_StringLiterals"></a>
### <a name="5-you-enclose-literal-string-values-in-double-quotation-marks"></a>5.リテラル文字列値を二重引用符で括る

A*文字列*テキストとして扱われる文字のシーケンスです。 文字列を指定するには、二重引用符で囲みます。

[!code-cshtml[Main](introducing-razor-syntax-c/samples/sample5.cshtml)]

文字列表示するにはかどうかは、円記号が含まれています ( `\` ) または二重引用符 ( `"` )、使用、 *verbatim 文字列リテラル*が付いている`@`演算子。 (C# で、\ 文字は verbatim 文字列リテラルを使用する場合を除き、特別な意味を持つ)。

[!code-html[Main](introducing-razor-syntax-c/samples/sample6.html)]

二重引用符を埋め込むには、逐語的文字列リテラルを使用し、引用符を繰り返します。

[!code-html[Main](introducing-razor-syntax-c/samples/sample7.html)]

ページを使用してこれらの例の両方の結果を次に示します。

![Razor-Img4](introducing-razor-syntax-c/_static/image4.jpg)

> [!NOTE]
> 注意、 `@` verbatim 文字列リテラル (C#) をマークして、ASP.NET ページ内のコードをマークする、文字を使用します。


### <a name="6-code-is-case-sensitive"></a>6.コードは大文字小文字を区別

C# のキーワード (など`var`、 `true`、および`if`) し、変数名では大文字小文字を区別します。 次のコード行が 2 つの異なる変数を作成`lastName`と `LastName.`

[!code-cshtml[Main](introducing-razor-syntax-c/samples/sample8.cshtml)]

として変数を宣言する場合`var lastName = "Smith";`としてページには、その変数を参照しようとする場合と`@LastName`、ために、エラーが発生`LastName`認識されません。

> [!NOTE]
> Visual Basic ではキーワードと変数は*いない*大文字小文字を区別します。


### <a name="7-much-of-your-coding-involves-objects"></a>7.コーディングのほとんどには、オブジェクトが含まれます

*オブジェクト*をプログラミングできることを表します&#8212;ページ、テキスト ボックス、ファイル、画像、web 要求を電子メール メッセージ、(データベースの行) の顧客レコードなど。オブジェクトの特性を記述するプロパティがあると、読み取りまたは変更ができる&#8212;テキスト ボックスのオブジェクトが、 `Text` (特に) プロパティは、要求オブジェクトが、`Url`プロパティ、電子メール メッセージが、`From`プロパティcustomer オブジェクトであり、`FirstName`プロパティ。 オブジェクトには、メソッドも含まれて、&quot;動詞&quot;を実行できます。 例としては、ファイル オブジェクトの`Save`メソッドは、イメージ オブジェクトの`Rotate`メソッド、および電子メール オブジェクトの`Send`メソッド。

多くの場合、使用する、`Request`オブジェクトを説明するテキスト ボックス (フォームのフィールド) の値などの情報 ページで、ブラウザーの種類は、要求、ページ、ユーザー id などの URL を作成します。次の例のプロパティにアクセスする方法を示しています、`Request`オブジェクトおよび呼び出す方法、`MapPath`のメソッド、`Request`オブジェクトで、サーバーで、ページの絶対パスを確認できます。

[!code-html[Main](introducing-razor-syntax-c/samples/sample9.html)]

ブラウザーで表示される結果:

![Razor-Img5](introducing-razor-syntax-c/_static/image5.jpg)

### <a name="8-you-can-write-code-that-makes-decisions"></a>8.意思決定を行うコードを記述することができます。

動的な web ページの重要な機能は、条件に基づいて対処方法を決定できます。 これを行う最も一般的な方法は、`if`ステートメント (と省略可能な`else`ステートメント)。

[!code-cshtml[Main](introducing-razor-syntax-c/samples/sample10.cshtml)]

ステートメント`if(IsPost)`は記事の執筆を簡略化`if(IsPost == true)`します。 と共に`if`ステートメント、さまざまなコードのブロックを繰り返し、条件をテストする方法があるしはこの記事の後半で説明されています。

結果がブラウザーで表示されます (クリックすると**送信**)。

![Razor-Img6](introducing-razor-syntax-c/_static/image6.jpg)

> [!TIP] 
> 
> <a id="SB_HttpGetPost"></a>
> ### <a name="http-get-and-post-methods-and-the-ispost-property"></a>HTTP GET と POST メソッドと IsPost プロパティ
> 
> Web ページ (HTTP) で使用されるプロトコルは、サーバーに要求を行うために使用するメソッド (動詞) の非常に限られた数をサポートします。 2 つの最も一般的なは、ページの読み取りに使用される、GET と POST のページを送信するために使用されます。 一般に、初めてユーザーの要求 ページでは、ページの要求が GET を使用します。 ユーザーが、フォームに入力し、送信ボタンをクリックし場合、ブラウザーは POST 要求をサーバーには。
> 
> Web プログラミング、かどうか、ページを要求する、POST または GET としてページを処理する方法がわかるように、知っておくと便利がよくあります。 使用する ASP.NET Web ページで、`IsPost`プロパティを要求が GET または POST がかどうかを参照してください。 要求が、投稿の場合、`IsPost`プロパティが true を返すし、フォーム上のテキスト ボックスの値などの読み取りを行うことができます。 値に応じて異なる方法でページを処理する方法がわかります多くの例に示します`IsPost`します。


## <a name="a-simple-code-example"></a>単純なコード例

この手順では、基本的なプログラミング手法を説明するページを作成する方法を示します。 例では、次にそれらを追加し、結果が表示されます、2 つの数値を入力できるようにするページを作成します。

1. エディターで、新しいファイルを作成し、名前*AddNumbers.cshtml*します。
2. ページでは、既にページ内のあらゆるものを置き換えるには、次のコードとマークアップをコピーします。  

    [!code-cshtml[Main](introducing-razor-syntax-c/samples/sample11.cshtml)]

    ここでは、いくつかの点に注意してください。

    - `@`文字 ページで、コードの最初のブロックを開始して、その直後にある、`totalMessage`ページの下部付近に埋め込まれている変数。
    - ページの上部にあるブロックは中かっこで囲まれます。
    - 上部にあるブロックでは、すべての行は、セミコロンで終わります。
    - 変数`total`、 `num1`、 `num2`、および`totalMessage`複数の数値と文字列を格納します。
    - 割り当てられているリテラル文字列値、`totalMessage`変数は、二重引用符内。
    - コードはときに、大文字小文字を区別するため、`totalMessage`ページの下部にある変数を使用すると、その名前では、上部にある変数を正確に一致する必要があります。
    - 式`num1.AsInt() + num2.AsInt()`オブジェクトおよびメソッドを使用する方法を示しています。 `AsInt`各変数のメソッドで算術演算を実行できるように、数値 (整数) に、ユーザーが入力した文字列に変換します。
    - `<form>`タグが含まれています、`method="post"`属性。 指定のユーザーがクリックしたときに**追加**ページは、HTTP POST メソッドを使用して、サーバーに送信されます。 ページが送信されたときに、 `if(IsPost)` test が true で、条件式が評価される、コードの数値を加算の結果を表示するを実行します。
3. ページを保存し、ブラウザーで実行します。 (内でページが選択されていることを確認、**ファイル**ワークスペースを実行する前にします)。2 つの整数を入力し、クリックして、**追加**ボタンをクリックします。 

    ![Razor-Img7](introducing-razor-syntax-c/_static/image7.jpg)

## <a name="basic-programming-concepts"></a>基本的なプログラミング概念

この記事では、ASP.NET web プログラミングの概要を提供します。 徹底的な検査も、最も頻繁に使用するプログラミングのコンセプトを通じて、クイック ツアーだけではありません。 そうであっても、ほとんどの ASP.NET Web Pages を開始する必要がありますが含まれます。

しかし、最初にほとんどの技術的な背景。

### <a name="the-razor-syntax-server-code-and-aspnet"></a>Razor 構文、サーバー コード、および ASP.NET

Razor 構文は、web ページにサーバー ベースのコードを埋め込むの単純なプログラミング構文です。 Razor 構文を使用する web ページでは、コンテンツの 2 つの種類があります。 クライアントのコンテンツとサーバーのコード。 クライアントのコンテンツは、web ページを使用しているもの。 HTML マークアップ (要素) のスタイルを設定、CSS などの情報も JavaScript、およびプレーン テキストなどのいくつかのクライアント スクリプト。

Razor 構文を使用して、このクライアントがコンテンツをサーバー コードを追加できます。 ページにサーバー コードがある場合、サーバーは、ブラウザーにページを送信する前に最初に、そのコードを実行します。 サーバーで実行すると、コードは、はるかに複雑クライアント コンテンツを単独で、サーバー ベースのデータベースへのアクセスなどを使用して行うことができるタスクを実行できます。 最も重要なは、サーバー コードをクライアントがコンテンツ動的に作成&#8212;HTML マークアップまたはその他のコンテンツをその場で生成し、ページを含む可能性がある静的な HTML と共にブラウザーに送信します。 ブラウザーの観点から、サーバー コードによって生成されるクライアントのコンテンツは、その他のクライアントのコンテンツと同じです。 既に説明したように必要なサーバー コードは簡単です。

Razor 構文を含む ASP.NET web ページがある特殊なファイル拡張子 (*.cshtml*または *.vbhtml*)。 サーバーでは、これらの拡張機能を認識して、、Razor 構文でマークされ、ブラウザーにページを送信するコードを実行します。

### <a name="where-does-aspnet-fit-in"></a>ここが ASP.NET 適合しますか。

Razor 構文は、Microsoft .NET Framework に基づくさらに、ASP.NET と呼ばれる Microsoft からのテクノロジに基づいています。 .Net Framework は、任意の種類のコンピューターのアプリケーションのほとんどを開発するための Microsoft から、包括的なプログラミング フレームワークです。 ASP.NET は、web アプリケーションを作成するために設計されている .NET Framework の一部です。 開発者は、ASP.NET を使用して、世界中で最大かつ最高のトラフィックの web サイトの多くを作成するのにがします。 (ファイル名拡張子を表示、いつ *.aspx*サイトの URL の一部として、ASP.NET を使用して、サイトに書き込まれたことを確認します)。

Razor 構文では、専門家がいるかどうかは、初心者と生産性の向上を利用を理解しやすくなりますが簡略化された構文を使用して、ASP.NET のすべての電源を使用します。 この構文は簡単に使用できるが、ASP.NET および .NET Framework とファミリの関係は、web サイトが複雑になると、使用できる大規模なフレームワークの機能をあることを意味します。

![Razor-Img8](introducing-razor-syntax-c/_static/image8.jpg)

> [!TIP] 
> 
> **クラスとインスタンス**
> 
> ASP.NET サーバー コードでは、クラスの概念に組み込まれているさらに、オブジェクトを使用します。 クラスは、定義またはオブジェクトのテンプレートです。 たとえば、アプリケーションが含まれる、`Customer`の customer オブジェクトが必要なメソッドとプロパティを定義するクラス。
> 
> アプリケーションは、実際の顧客情報を使用する必要があるのインスタンスが作成されます (または*をインスタンス化*) customer オブジェクト。 個々 の顧客は、別のインスタンス、`Customer`クラス。 すべてのインスタンスを同じプロパティとメソッドをサポートしていますが、各 customer オブジェクトが一意であるために、各インスタンスのプロパティの値は通常は異なります。 1 人の顧客のオブジェクトで、`LastName`プロパティが"Smith"をする可能性がありますもう 1 つの customer オブジェクトに、`LastName`プロパティで"Jones。"可能性があります。
> 
> 同様に、すべて個別の web ページ サイトでは、`Page`オブジェクトのインスタンスでは、`Page`クラス。 ページ上のボタンは、`Button`オブジェクトのインスタンスでは、`Button`クラス、という具合です。 各インスタンスがその独自の特性がすべてに基づいているオブジェクトのクラスの定義で指定されています。


## <a name="basic-syntax"></a>基本構文

以前、ASP.NET Web Pages ページを作成する方法と、HTML マークアップをサーバー コードを追加する方法の基本的な例を説明しました。 Razor 構文を使用して ASP.NET サーバー コードの記述の基礎を学習ここ&#8212;プログラミング言語の規則では、します。

(特に C、C++、C#、Visual Basic、または JavaScript を使用した) 場合のプログラミング経験豊富な場合は、ここで読んだの多くが使い慣れたになります。 内のマークアップにサーバー コードを追加する方法でのみについて理解しておく必要がありますおそらく *.cshtml*ファイル。

<a id="BM_CombiningTextMarkupAndCode"></a>
### <a name="combining-text-markup-and-code-in-code-blocks"></a>テキスト、マークアップ、およびコード ブロック内のコードを組み合わせること

サーバー コード ブロックでは、多くの場合、出力テキストまたはマークアップ (またはその両方) のページにします。 サーバー コード ブロックにコードでないし、する代わりに表示するか、テキストが含まれている場合、ASP.NET をコードからそのテキストを区別するためにできる必要があります。 これにはいくつかの方法があります。

- ような HTML 要素にテキストを囲む`<p></p>`または`<em></em>`:   

    [!code-cshtml[Main](introducing-razor-syntax-c/samples/sample12.cshtml)]

    HTML 要素には、テキスト、HTML 要素の追加、およびサーバー コード式を含めることができます。 ASP.NET が HTML の開始タグを表示する場合 (たとえば、 `<p>`) としてその内容は、ブラウザーが移動するときに、サーバー コード式を解決するのには、要素を含め、すべてを表示します。
- 使用して、`@:`演算子または`<text>`要素。 `@:`プレーン テキストまたは HTML タグが一致していませんが含まれているコンテンツを 1 行の出力、`<text>`要素を出力する複数の行で囲みます。 これらのオプションは、出力の一部として HTML 要素を表示したくない場合に便利です。  

    [!code-cshtml[Main](introducing-razor-syntax-c/samples/sample13.cshtml)]

    複数のテキストまたは HTML タグが一致しない行を出力する場合は、各行の前に記述できます`@:`、または内の行を囲むことができます、`<text>`要素。 ように、`@:`演算子、`<text>`タグのテキスト コンテンツを識別するために、ASP.NET で使用され、ページ出力には常にレンダリングします。

    [!code-cshtml[Main](introducing-razor-syntax-c/samples/sample14.cshtml)]

    最初の例では、前の例の繰り返しですがの 1 つのペアを使用して`<text>`タグを表示するテキストを囲みます。 2 番目の例で、`<text>`と`</text>`タグは、一部の非包含エンティティのテキストと HTML タグの不一致があるすべての 3 つの行を囲む (`<br />`)、サーバー コードと一致した HTML タグ。 ここでも、使用して個別に各の行の前でしたも、`@:`演算子です。 いずれかの方法の動作。

    > [!NOTE]
    > このセクションで示すようにテキストを出力するときに&#8212;、HTML 要素を使用して、`@:`演算子、または`<text>`要素&#8212;ASP.NET は HTML エンコード出力します。 (ASP.NET がサーバーのコード式とが付いているサーバー コード ブロックの出力をエンコードする前述のように、 `@`、このセクションに記載されている特殊なケースでは可します)。

### <a name="whitespace"></a>Whitespace

余分なスペース (および、文字列リテラルの外部で) ステートメントでステートメントには影響しません。

[!code-cshtml[Main](introducing-razor-syntax-c/samples/sample15.cshtml)]

ステートメント内での改行は、ステートメントの影響を与えませんし、読みやすくするためのステートメントをラップすることができます。 次のステートメントでは、同じです。

[!code-cshtml[Main](introducing-razor-syntax-c/samples/sample16.cshtml)]

ただし、文字列リテラル内の行をラップすることはできません。 次の例が機能しません。

[!code-cshtml[Main](introducing-razor-syntax-c/samples/sample17.cshtml)]

上記のコードのような複数の行に折り返す長い文字列を結合するには、2 つのオプションがあります。 連結演算子を使用することができます (`+`)、この記事の後半でがあります。 使用することも、`@`逐語的文字列リテラルの作成、この記事の前半で説明するようにする文字。 Verbatim 文字列リテラルを分割することで、境界をまたがってできます。

[!code-cshtml[Main](introducing-razor-syntax-c/samples/sample18.cshtml)]

### <a name="code-and-markup-comments"></a>コード (およびマークアップ) コメント

コメントを使用して、自分自身または他のユーザーは、ノートをそのまま使用できます。 これらも無効にすること (*をコメント アウト*) コードまたはを実行する必要はありませんが、当面は、ページに保持するマークアップのセクション。

別の構文と、HTML マークアップの Razor コードのコメントがあります。 すべての Razor コードでは、Razor コメントは処理 (と同様、削除)、ページがブラウザーに送信される前に、サーバー上。 そのため、Razor コメントの構文では、そのファイルを編集するが、ページのソースであっても、ユーザーが表示されないときに表示できますコメントをコードに (または、さらには、マークアップ) を挿入することができます。

ASP.NET Razor のコメントのコメントを開始する`@*`で終わら`*@`します。 コメントは、1 つの行または複数の行のできます。

[!code-cshtml[Main](introducing-razor-syntax-c/samples/sample19.cshtml)]

コード ブロック内のコメントを次に示します。

[!code-cshtml[Main](introducing-razor-syntax-c/samples/sample20.cshtml)]

ここでは行のコードで、コードの同じブロック コメント アウトされて実行されないようにします。

[!code-cshtml[Main](introducing-razor-syntax-c/samples/sample21.cshtml)]

Razor コメントの構文を使用する代わりにコード ブロック内では c# などを使用するプログラミング言語のコメント構文を使用できます。

[!code-cshtml[Main](introducing-razor-syntax-c/samples/sample22.cshtml)]

C# の場合は、単一行コメントが付いて、`//`文字、および複数行コメントの始まり`/*`と末尾`*/`します。 (Razor のコメントと同様 c# コメントは、ブラウザーにレンダリングされません)。

マークアップの場合、ご存知のとおり、HTML コメントを作成できます。

[!code-xml[Main](introducing-razor-syntax-c/samples/sample23.xml)]

HTML コメントの始まり`<!--`文字、末尾`-->`します。 だけでなく、テキスト、HTML マークアップもに、ページを保持することがありますが、表示するためにしたくないを囲む HTML コメントを使用することができます。 この HTML コメントには、タグとが含まれているテキストの内容全体が非表示になります。

[!code-html[Main](introducing-razor-syntax-c/samples/sample24.html)]

Razor コメント、HTML コメントとは異なり*は*ページに表示されると、ユーザーがページのソースを表示して確認できます。

Razor では、c# の入れ子になったブロックに制限があります。 詳細については、次を参照してください[という名前の c# 変数と入れ子になったブロック壊れたコードの生成。](http://aspnetwebstack.codeplex.com/workitem/1914)

## <a name="variables"></a>変数

変数は、データの格納に使用する名前付きオブジェクトです。 名前を付ける変数、何かが、名前がアルファベットの文字で始める必要があり、空白文字または予約文字を含めることはできません。

### <a name="variables-and-data-types"></a>変数とデータ型

変数は、データの種類が変数に格納されていることを示しますが、特定のデータ型を持つことができます。 文字列値を格納する文字列変数があることができます (よう&quot;Hello world&quot;)、(3 または 79) などの整数値を格納する整数変数とさまざまな (4/12/2012 や 2009 年 3 月などの形式で日付の値を格納する date 型の変数). その他の多くのデータ型を使用することができます。

ただし、通常、変数の型を指定するがありません。 ほとんどの場合、ASP.NET は、変数内のデータの使用方法に基づいて型を判断することができます。 (場合によっては、型を指定する必要があります。 これは true、例が表示されます)。

使用して変数を宣言する、`var`キーワード (型を指定しない) 場合や、型の名前を使用しています。

[!code-cshtml[Main](introducing-razor-syntax-c/samples/sample25.cshtml)]

次の例では、web ページで変数の一般的な使用を示します。

[!code-cshtml[Main](introducing-razor-syntax-c/samples/sample26.cshtml)]

ページで、前の例を結合すると、ブラウザーで表示されます。 これが表示されます。

![Razor-Img9](introducing-razor-syntax-c/_static/image9.jpg)

### <a name="converting-and-testing-data-types"></a>変換して、データ型をテストします。

ASP.NET は、データ型を自動的に決定することができます、通常が場合があります、ことはできません。 そのため、明示的な変換を実行することで ASP.NET を手助けする必要があります。 型変換を持っていない場合でもデータの種類が使用することを確認するテストに便利です。

最も一般的なは、整数や日付など、別の型を文字列に変換することがあります。 次の例では、文字列を数値に変換する必要があります、一般的なケースを示します。

[!code-cshtml[Main](introducing-razor-syntax-c/samples/sample27.cshtml)]

原則として、ユーザー入力では、文字列として。 番号を入力するユーザーを要求した場合でも、および数字をユーザー入力が送信され、コードでの読み取り時に入力する場合でも、データは文字列の形式です。 そのため、文字列を数値に変換する必要があります。 例では、変換、しなくても、値に対して算術演算を実行しようとする場合、次のエラーの結果、ASP.NET は、2 つの文字列を追加できないため。

*'Int' に ' string' 型に暗黙的に変換できません。*

呼び出すの値を整数に変換する、`AsInt`メソッド。 変換が成功した場合は、数値を追加できます。

次の表では、変数の一般的ないくつかの変換とテスト方法を示します。

:::row:::
    :::column:::
    <strong>メソッド</strong>
    :::column-end:::
    :::column:::
    <strong>説明</strong>
    :::column-end:::
    :::column:::
    <strong>例</strong>
    :::column-end:::
:::row-end:::
* * *
:::row:::
    :::column:::
        `AsInt(), IsInt()`
    :::column-end:::
    :::column:::
    整数 (「593」) などの整数を表す文字列に変換します。
    :::column-end:::
    :::column:::
        [!code-csharp[Main](introducing-razor-syntax-c/samples/sample28.cs)]
    :::column-end:::
:::row-end:::
* * *
:::row:::
    :::column:::
        `AsBool(), IsBool()`
    :::column-end:::
    :::column:::
    などの文字列に変換します&quot;true&quot;または&quot;false&quot;ブール型にします。
    :::column-end:::
    :::column:::
        [!code-csharp[Main](introducing-razor-syntax-c/samples/sample29.cs)]
    :::column-end:::
:::row-end:::
* * *
:::row:::
    :::column:::
        `AsFloat(), IsFloat()`
    :::column-end:::
    :::column:::
    ような 10 進数の値を持つ文字列に変換します&quot;1.3&quot;または&quot;7.439&quot;を浮動小数点数。
    :::column-end:::
    :::column:::
        [!code-csharp[Main](introducing-razor-syntax-c/samples/sample30.cs)]
    :::column-end:::
:::row-end:::
* * *
:::row:::
    :::column:::
        `AsDecimal(), IsDecimal()`
    :::column-end:::
    :::column:::
    ような 10 進数の値を持つ文字列に変換します&quot;1.3&quot;または&quot;7.439&quot; 10 進数。 (ASP.NET では、10 進数、浮動小数点数よりも正確)。
    :::column-end:::
    :::column:::
        [!code-csharp[Main](introducing-razor-syntax-c/samples/sample31.cs)]
    :::column-end:::
:::row-end:::
* * *
:::row:::
    :::column:::
        `AsDateTime(), IsDateTime()`
    :::column-end:::
    :::column:::
    Asp.net の日付と時刻の値を表す文字列に変換します`DateTime`型。
    :::column-end:::
    :::column:::
        [!code-csharp[Main](introducing-razor-syntax-c/samples/sample32.cs)]
    :::column-end:::
:::row-end:::
* * *
:::row:::
    :::column:::
        `ToString()`
    :::column-end:::
    :::column:::
    その他の任意のデータ型を文字列に変換します。
    :::column-end:::
    :::column:::
        [!code-javascript[Main](introducing-razor-syntax-c/samples/sample33.js)]
    :::column-end:::
:::row-end:::

## <a name="operators"></a>演算子

演算子は、キーワードまたは式の中で実行するコマンドの種類を ASP.NET に指示する文字です。 C# 言語 (とそれに基づいている Razor 構文) は、多くの演算子をサポートしていますが、開始するいくつかを認識するだけで済みます。 次の表では、最も一般的な演算子をまとめたものです。


:::row:::
    :::column:::
    <strong>Operator</strong>
    :::column-end:::
    :::column:::
    <strong>説明</strong>
    :::column-end:::
    :::column:::
    <strong>例</strong>
    :::column-end:::
:::row-end:::
* * *
:::row:::
    :::column:::
        `+` `-` `*` `/`
    :::column-end:::
    :::column:::
    算術演算子が数値式で使用します。
    :::column-end:::
    :::column:::
        [!code-css[Main](introducing-razor-syntax-c/samples/sample34.css)]
    :::column-end:::
:::row-end:::
* * *
:::row:::
    :::column:::
        `=`
    :::column-end:::
    :::column:::
    代入。 左側にあるオブジェクトには、ステートメントの右側にある値を割り当てます。
    :::column-end:::
    :::column:::
        [!code-csharp[Main](introducing-razor-syntax-c/samples/sample35.cs)]
    :::column-end:::
:::row-end:::
* * *
:::row:::
    :::column:::
        `==`
    :::column-end:::
    :::column:::
    等値。 返します`true`値が等しい場合。 (上での違いに注意してください、`=`演算子と`==`演算子です)。
    :::column-end:::
    :::column:::
        [!code-csharp[Main](introducing-razor-syntax-c/samples/sample36.cs)]
    :::column-end:::
:::row-end:::
* * *
:::row:::
    :::column:::
        `!=`
    :::column-end:::
    :::column:::
    非等値。 返します`true`値が等しくない場合。
    :::column-end:::
    :::column:::
        [!code-csharp[Main](introducing-razor-syntax-c/samples/sample37.cs)]
    :::column-end:::
:::row-end:::
* * *
:::row:::
    :::column:::
        `< > <= >=`
    :::column-end:::
    :::column:::
    以下のより大きい-より、小さいよりも-または-等しく、大きい-よりも-または-等しい。
    :::column-end:::
    :::column:::
        [!code-csharp[Main](introducing-razor-syntax-c/samples/sample38.cs)]
    :::column-end:::
:::row-end:::
* * *
:::row:::
    :::column:::
        `+`
    :::column-end:::
    :::column:::
    連結、文字列を結合するために使用します。 ASP.NET では、この演算子と式のデータ型に基づく加算演算子の違いを認識します。
    :::column-end:::
    :::column:::
        [!code-csharp[Main](introducing-razor-syntax-c/samples/sample39.cs)]
    :::column-end:::
:::row-end:::
* * *
:::row:::
    :::column:::
        `+=` `-=`
    :::column-end:::
    :::column:::
    インクリメントとデクリメント演算子を追加し、変数から 1 をそれぞれ減算します。
    :::column-end:::
    :::column:::
        [!code-csharp[Main](introducing-razor-syntax-c/samples/sample40.cs)]
    :::column-end:::
:::row-end:::
* * *
:::row:::
    :::column:::
        `.`
    :::column-end:::
    :::column:::
    ドットです。 オブジェクトとそのプロパティおよびメソッドを区別するために使用されます。
    :::column-end:::
    :::column:::
        [!code-csharp[Main](introducing-razor-syntax-c/samples/sample41.cs)]
    :::column-end:::
:::row-end:::
* * *
:::row:::
    :::column:::
        `()`
    :::column-end:::
    :::column:::
    かっこです。 グループ式をメソッドにパラメーターを渡すために使用します。
    :::column-end:::
    :::column:::
        [!code-javascript[Main](introducing-razor-syntax-c/samples/sample42.js)]
    :::column-end:::
:::row-end:::
* * *
:::row:::
    :::column:::
        `[]`
    :::column-end:::
    :::column:::
    角かっこです。 配列またはコレクション内の値にアクセスするために使用されます。
    :::column-end:::
    :::column:::
        [!code-csharp[Main](introducing-razor-syntax-c/samples/sample43.cs)]
    :::column-end:::
:::row-end:::
* * *
:::row:::
    :::column:::
        `!`
    :::column-end:::
    :::column:::
    じゃない。 逆に、`true`値を`false`またはその逆です。 通常のテストを簡略化として使用される`false`(には、いない`true`)。
    :::column-end:::
    :::column:::
        [!code-csharp[Main](introducing-razor-syntax-c/samples/sample44.cs)]
    :::column-end:::
:::row-end:::
* * *
:::row:::
    :::column:::
        `&&` <code>&#124;&#124;</code>
    :::column-end:::
    :::column:::
    論理 AND またはと、条件をまとめてリンクに使用されます。
    :::column-end:::
    :::column:::
        [!code-csharp[Main](introducing-razor-syntax-c/samples/sample45.cs)]
    :::column-end:::
:::row-end:::

<a id="ID_WorkingWithFileAndFolderPaths"></a>
## <a name="working-with-file-and-folder-paths-in-code"></a>ファイルとコード内のフォルダー パスを使用します。

コードでは、ファイルとフォルダーのパスを持つ多くの場合、操作します。 開発用コンピューターに表示される次の web サイトの物理フォルダー構造の例に示します。

`C:\WebSites\MyWebSite default.cshtml datafile.txt \images Logo.jpg \styles Styles.css`

Url およびパスについていくつかの重要な詳細情報を次に示します。

- URL のいずれかでドメイン名を開始 (`http://www.example.com`) またはサーバー名 (`http://localhost`、 `http://mycomputer`)。
- URL は、ホスト コンピューター上の物理パスに対応します。 たとえば、`http://myserver`フォルダーに対応する*C:\websites\mywebsite*サーバー。
- 仮想パスは、完全なパスを指定せずに、コード内のパスを表すための短縮形です。 ドメインまたはサーバー名に続く URL の一部が含まれています。 仮想パスを使用すると、パスを更新することがなく別のドメインまたはサーバーに、コードを移動できます。

相違点の理解に役立つ例を次に示します。

| 完全な URL | `http://mycompanyserver/humanresources/CompanyPolicy.htm` |
| --- | --- |
| サーバー名 | *mycompanyserver* |
| [仮想パス] | */humanresources/CompanyPolicy.htm* |
| 物理パス | *C:\mywebsites\humanresources\CompanyPolicy.htm* |

仮想ルートは、/、ドライブは、c: のルートと同じように \ です。 (仮想フォルダーのパス常にスラッシュを使用します。)フォルダーの仮想パスとして、物理フォルダー。 同じ名前を指定する必要はありません。エイリアスがあります。 (実稼働サーバー上の仮想パスことはほとんどありませんと一致する特定の物理パス。)

コードのファイルとフォルダーを使用する場合は場合がありますの物理パスおよび場合によってはどのようなオブジェクトを使用しているに応じての仮想パスを参照する必要があります。 ASP.NET ツールを提供するこれらのコードでのファイルとフォルダーのパスの作業:`Server.MapPath`メソッド、および`~`演算子と`Href`メソッド。

### <a name="converting-virtual-to-physical-paths-the-servermappath-method"></a>仮想物理パスに変換する: Server.MapPath メソッド

`Server.MapPath`メソッドが仮想パスに変換します (など */default.cshtml*) を絶対物理パス (など*C:\WebSites\MyWebSiteFolder\default.cshtml*)。 このメソッドを使用すると、いつでも完全な物理パスを作成する必要がありますに。 典型的な例では、読み取りまたはテキスト ファイルまたは web サーバー上の画像ファイルを作成する場合です。

通常ホスティング サイトのサーバー上のサイトの絶対物理パスがわからない、わかっているため、このメソッドは、パスを変換できます: 仮想パス: サーバー上の対応するパスにします。 仮想パスをファイルまたはフォルダー、メソッドに渡すし、物理パスを返します。

[!code-cshtml[Main](introducing-razor-syntax-c/samples/sample46.cshtml)]

### <a name="referencing-the-virtual-root-the--operator-and-href-method"></a>仮想ルートを参照する: ~ 演算子と Href メソッド

*.Cshtml*または *.vbhtml*ファイル、仮想ルート パスを使用して、参照することができます、`~`演算子。 これは、機能は、サイトでは、内のページを移動することができ、他のページが含まれているすべてのリンクが壊れているできませんので、非常に便利です。 これまで、web サイトを別の場所に移動する場合にも便利です。 次にいくつかの例を示します。

[!code-cshtml[Main](introducing-razor-syntax-c/samples/sample47.cshtml)]

場合は、web サイトが`http://myserver/myapp`、ここでは、ASP.NET が処理する方法これらのパス、ページの実行時に。

- `myImagesFolder`: `http://myserver/myapp/images`
- `myStyleSheet` : `http://myserver/myapp/styles/Stylesheet.css`

(実際には、変数の値としてこれらのパスは表示されませんが ASP.NET は、パスとして扱うだったです)。

使用することができます、 `~` (上記) としてサーバー コードと次のように、マークアップの両方の演算子。

[!code-html[Main](introducing-razor-syntax-c/samples/sample48.html)]

マークアップでは、使用する、`~`演算子イメージ ファイル、その他の web ページ、および CSS ファイルなどのリソースへのパスを作成します。 ASP.NET が (コードとマークアップの両方) のページを探し、すべて解決、ページの実行時に、`~`適切なパスへの参照。

## <a name="conditional-logic-and-loops"></a>条件付きロジックやループ

ASP.NET サーバー コードでは、条件に基づいてタスクを実行して、特定回数のステートメントを繰り返し実行するコードを記述することができます (つまり、ループを実行するコード)。

### <a name="testing-conditions"></a>条件のテスト

使用する単純な条件をテストする、`if`ステートメントで、指定したテストに基づく true または false を返します。

[!code-cshtml[Main](introducing-razor-syntax-c/samples/sample49.cshtml)]

`if`キーワードは、ブロックを開始します。 実際のテスト (条件) はかっこであり、true または false を返します。 テストが true の場合に実行されるステートメントは、中かっこで囲まれます。 `if`ステートメントを含めることができます、`else`ブロックする条件が false の場合に実行するステートメントを指定します。

[!code-cshtml[Main](introducing-razor-syntax-c/samples/sample50.cshtml)]

使用して複数の条件を追加することができます、`else if`ブロック。

[!code-cshtml[Main](introducing-razor-syntax-c/samples/sample51.cshtml)]

場合、最初の条件の場合、この例ではブロックが true でない、`else if`条件がチェックされます。 その条件が満たされた場合、ステートメントで、`else if`ブロックが実行されます。 どの条件が満たされた場合、ステートメントで、`else`ブロックが実行されます。 場合、他の任意の数を追加する、ブロックを閉じて、`else`としてブロック、&quot;他のすべて&quot;条件。

多くの条件をテストするには、使用、`switch`ブロック。

[!code-cshtml[Main](introducing-razor-syntax-c/samples/sample52.cshtml)]

テストする値がかっこ内に示します (例では、`weekday`変数)。 各テストを使用して、`case`コロン (:) で終了するステートメント。 場合の値を`case`ステートメントには、テスト値が一致すると、その case ブロック内のコードを実行します。 各 case ステートメントで閉じる、`break`ステートメント。 (それぞれに区切りを含めるを忘れた場合`case`次のコードをブロックします`case`ステートメントも実行されます。)。A`switch`ブロックには多くの場合、`default`ステートメントの最後のケースとして、&quot;他のすべて&quot;該当するその他のケースがない場合に実行するオプション。

ブラウザーで表示される最後の 2 つの条件付きブロックの結果:

![Razor-Img10](introducing-razor-syntax-c/_static/image10.jpg)

### <a name="looping-code"></a>ループのコード

多くの場合、同じステートメントを繰り返し実行する必要があります。 そのためには、ループ処理します。 たとえば、多くの場合、ステートメントを実行する同じ項目ごとにデータのコレクション。 使用することが正確に何回ループすることがわかっている場合、`for`ループします。 この種のループはまでカウントするか、カウント ダウン特に便利です。

[!code-html[Main](introducing-razor-syntax-c/samples/sample53.html)]

ループが始まり、`for`キーワード、かっこ、3 つのステートメントに続く各をセミコロンで終了します。

- 最初のステートメントをかっこ内 (`var i=10;`) カウンターを作成し、10 を初期化します。 カウンターの名前を付ける必要はありません`i`&#8212;任意の変数を使用することができます。 ときに、`for`ループが実行され、カウンターが自動的にインクリメントされます。
- 2 番目のステートメント (`i < 21;`) 条件をカウントする距離を設定します。 20 個に移動させたいここでは、(これは、引き続き、カウンターは 21 未満)。
- 3 番目のステートメント (`i++` ) カウンターは、ループが実行されるたびに追加する 1 である必要がありますを指定するだけインクリメント演算子を使用します。

中かっこ内のループの反復ごとに実行されるコードに示します。 マークアップは、新しい段落を作成する (`<p>`要素) 時間し、の値を表示する出力に行を追加します。 各`i`(カウンター)。 このページを実行すると、例は、11 行の行のテキスト項目の数を示す行ごとに、出力の表示を作成します。

![Razor-Img11](introducing-razor-syntax-c/_static/image11.jpg)

多くの場合、使用、コレクションまたは配列で作業している場合、`foreach`ループします。 コレクションは、類似のオブジェクトのグループと`foreach`ループでコレクション内の各項目のタスクを実行することができます。 この種のループはコレクションの場合、便利なためとは異なり、`for`ループ カウンターを増分または制限を設定する必要はありません。 代わりに、`foreach`ループのコードだけを進め、コレクションが完了するまでです。

たとえば、次のコードが内の項目を返します、`Request.ServerVariables`コレクションで、web サーバーに関する情報を含むオブジェクトです。 使用して、`foreac`新たに作成して各項目の名前を表示する h ループ`<li>`HTML の箇条書きリスト内の要素。

[!code-html[Main](introducing-razor-syntax-c/samples/sample54.html)]

`foreach`キーワードに続けてかっこ、コレクション内の 1 つの項目を表す変数を宣言する (例では、 `var item`)、その後に、`in`キーワードのコレクションをループするか後にします。 本体で、`foreach`ループ、先ほど宣言した変数を使用して現在の項目にアクセスすることができます。

![Razor-Img12](introducing-razor-syntax-c/_static/image12.jpg)

汎用的なループを作成するには、使用、`while`ステートメント。

[!code-cshtml[Main](introducing-razor-syntax-c/samples/sample55.cshtml)]

A`while`でループが開始、`while`キーワード、その後にかっこ、ループの継続期間を指定する (ここでは、限り`countNum`50 未満) を繰り返すブロックします。 ループの通常のインクリメント (に追加する) または減分 (減算)、変数またはオブジェクト カウントで使用されます。 例では、`+=`演算子に 1 を追加します`countNum`たびに、ループを実行します。 (カウント ダウンをループ内の変数をデクリメントするは、デクリメント演算子を使用して`-=`)。

## <a name="objects-and-collections"></a>オブジェクトとコレクション

ほぼすべての ASP.NET web サイトでは、web ページ自体を含むオブジェクトです。 このセクションでは、使用する多くの場合、コードでいくつかの重要なオブジェクトについて説明します。

### <a name="page-objects"></a>ページのオブジェクト

ASP.NET の最も基本的なオブジェクトは、ページです。 オブジェクトを修飾せずに直接ページ オブジェクトのプロパティにアクセスすることができます。 次のコード ページのファイルのパスを取得するを使用して、`Request`ページのオブジェクト。

[!code-cshtml[Main](introducing-razor-syntax-c/samples/sample56.cshtml)]

ようにクリアのプロパティやメソッド、現在のページを参照していることができます必要に応じてキーワードを使用する`this`を表す、コード内のページ オブジェクト。 上記のコード例を次に示します`this`を追加して、ページを表します。

[!code-cshtml[Main](introducing-razor-syntax-c/samples/sample57.cshtml)]

プロパティを使用して、`Page`など多くの情報を取得するオブジェクト。

- `Request`。 既に説明したように行われる要求、ページ、ユーザー id などの URL をブラウザーの種類を含む、現在の要求に関する情報のコレクションになります。
- `Response`。 これは、サーバー コードの実行が終了したら、ブラウザーに送信される応答 (ページ) に関する情報のコレクションです。 たとえば、このプロパティを使用すると、応答に情報を書き込みます。 

    [!code-cshtml[Main](introducing-razor-syntax-c/samples/sample58.cshtml)]

<a id="ID_CollectionsAndObjects"></a>
### <a name="collection-objects-arrays-and-dictionaries"></a>コレクション オブジェクト (配列、ディクショナリ)

A*コレクション*のコレクションなど、同じ型のオブジェクトのグループは、`Customer`データベースからのオブジェクト。 このような ASP.NET に多くの組み込みコレクションが含まれています、`Request.Files`コレクション。

多くの場合、コレクション内のデータを操作します。 2 つの一般的なコレクション型は、*配列*と*ディクショナリ*します。 配列は、類似した項目のコレクションを格納するが、それぞれ個別の各項目を保持する変数を作成したくない場合に役立ちます。

[!code-cshtml[Main](introducing-razor-syntax-c/samples/sample59.cshtml)]

配列の使用、特定のデータ型を宣言するなど`string`、 `int`、または`DateTime`します。 変数が配列を含めることができます、宣言に角かっこを追加することを示すために (など`string[]`または`int[]`)。 またはを使用してそれらの位置 (インデックス) を使用して、配列内の項目にアクセスすることができます、`foreach`ステートメント。 配列のインデックスは 0 から始まる&#8212;つまり最初の項目がある 0 の位置は、2 番目の項目は、位置 1、という具合にします。

[!code-cshtml[Main](introducing-razor-syntax-c/samples/sample60.cshtml)]

取得することによって、配列内の項目の数を判断するその`Length`プロパティ。 (配列を検索します)、配列内の特定の項目の位置を取得する、`Array.IndexOf`メソッド。 行うことができますも逆の順序など、配列の内容 (、`Array.Reverse`メソッド) または内容を並べ替える (、`Array.Sort`メソッド)。

ブラウザーで表示されている文字列配列コードの出力:

![Razor-Img13](introducing-razor-syntax-c/_static/image13.jpg)

ディクショナリは設定または対応する値を取得するキー (または名前) を指定するキー/値のペアのコレクションです。

[!code-cshtml[Main](introducing-razor-syntax-c/samples/sample61.cshtml)]

使用するディクショナリを作成する、`new`新しいディクショナリ オブジェクトを作成していることを示すキーワードです。 変数を使用するディクショナリを割り当てることができます、`var`キーワード。 山かっこを使用して、ディクショナリ内の項目のデータ型を指定する ( `< >` )。 宣言の末尾には、これは実際には、新しいディクショナリを作成するメソッドなので 1 組のかっこを追加する必要があります。

項目をディクショナリに追加するを呼び出すことができます、`Add`辞書の変数のメソッド (`myScores`ここでは)、キーと値を指定します。 または、次の例のように、単純な割り当てを行い、キーを示すに角かっこを使用できます。

[!code-csharp[Main](introducing-razor-syntax-c/samples/sample62.cs)]

ディクショナリから値を取得するには、角かっこでキーを指定します。

[!code-csharp[Main](introducing-razor-syntax-c/samples/sample63.cs)]

## <a name="calling-methods-with-parameters"></a>パラメーターを持つメソッドを呼び出す

この記事で前述の読み取りとプログラミングでのオブジェクトはメソッドを持つことができます。 たとえば、`Database`オブジェクトがあります、`Database.Connect`メソッド。 多くのメソッドは、1 つまたは複数のパラメーターを指定します。 A*パラメーター*メソッドに渡す値は、タスクが完了する方法を有効にします。 たとえばの宣言を見て、`Request.MapPath`メソッドは、次の 3 つのパラメーターを受け取る。

[!code-csharp[Main](introducing-razor-syntax-c/samples/sample64.cs)]

(読みやすくする、行に折り返されています。 ほぼすべての場所内で文字列を除いて、改行を配置できることに注意してください引用符で囲まれた。)。

このメソッドは、指定した仮想パスに対応するサーバー上の物理パスを返します。 メソッドの 3 つのパラメーターは`virtualPath`、 `baseVirtualDir`、および`allowCrossAppMapping`します。 (宣言で、パラメーターは許可されるデータのデータ型の一覧に注目してください)。このメソッドを呼び出すときに、次の 3 つのすべてのパラメーターの値を指定してください。

Razor 構文は、メソッドにパラメーターを渡すための 2 つのオプションを使用:*位置指定パラメーター*と*名前付きパラメーター*します。 位置指定パラメーターを使用してメソッドを呼び出すには、メソッドの宣言で指定されている厳密な順序でパラメーターを渡します。 (が通常わかるこの順序メソッドのドキュメントを参照しています。)順序に従い、任意のパラメーターをスキップすることはできません&#8212;かどうか、必要に応じて空の文字列を渡す (`""`) または`null`位置指定パラメーターの値がないのです。

次の例は、という名前のフォルダーにあることを前提*スクリプト*web サイトにします。 コードの呼び出し、`Request.MapPath`を正しい順序で 3 つのパラメーターの値をメソッドを渡します。 結果のマップされたパスが表示されます。

[!code-cshtml[Main](introducing-razor-syntax-c/samples/sample65.cshtml)]

メソッドは、多くのパラメーターを持ち、ときにするおくと、コードより読みやすい名前付きパラメーターを使用しています。 名前付きパラメーターを使用してメソッドを呼び出すには、パラメーター名とそれに続くコロン (:)、し、値を指定します。 名前付きパラメーターの利点は、任意の順序で渡すできることです。 (欠点は、メソッドの呼び出しが、compact ではないこと) です。

次の例は、上記と同じメソッドを呼び出しますが、名前付きの値を指定するパラメーターの使用。

[!code-cshtml[Main](introducing-razor-syntax-c/samples/sample66.cshtml)]

ご覧のように、パラメーターは異なる順序で渡されます。 ただし、前の例と、この例を実行する場合、同じ値を返すします。

<a id="ID_HandlingErrors"></a>
## <a name="handling-errors"></a>エラー処理

### <a name="try-catch-statements"></a>Try-catch ステートメント

上の理由から、コントロールの外部に障害が発生する、コード内のステートメントを多くの場合、があります。 例えば:

- コードを作成またはファイルにアクセスしようとすると、あらゆる種類のエラーが発生する可能性があります。 目的のファイルが存在しない可能性があります、ロックアウトされることがあります、コードが、権限がありませんして具合可能性があります。
- 同様に場合は、コードでは、データベース内のレコードを更新しようとして、アクセス許可の問題があります、データベースへの接続が削除される可能性が、無効なおよびこれを保存するデータがあります。

プログラミング用語では、このような状況が呼び出されます*例外*します。 (スロー) が生成されますが、コードには、例外が発生すると、エラー メッセージのユーザーに最高でいたずら。

![Razor-Img14](introducing-razor-syntax-c/_static/image14.jpg)

コードが例外を発生可能性がある場合に、この種類のエラー メッセージを回避するためには、使用する`try/catch`ステートメント。 `try`ステートメントでは、チェックするコードを実行します。 1 つまたは複数`catch`ステートメントでは、特定の検索できます (特定の種類の例外の) エラーの発生した可能性があります。 多くとして含めることができます`catch`するとしてステートメントを予測しているエラーを確認する必要があります。

> [!NOTE]
> 使用を避けることをお勧めします。、`Response.Redirect`メソッド`try/catch`ステートメント、ページで、例外が生じるためです。


次の例では、最初の要求でテキスト ファイルを作成し、ファイルを開くユーザーを切り替えるボタンを表示するページを示します。 例は、例外が発生されるように意図的に不適切なファイル名を使用します。 コードが含まれています`catch`可能性のある例外の 2 つのステートメント: `FileNotFoundException`、ファイル名が間違っている場合に発生して`DirectoryNotFoundException`ASP.NET は、フォルダーにも見つからない場合に発生します。 (コメントを解除できます、ステートメントの例では、すべてが適切に動作しているときの動作を確認するためにします。)

コードが例外を処理していない場合は、前のスクリーン ショットのようなエラー ページが表示されます。 ただし、`try/catch`セクションでは、ユーザーがこの種のエラーが表示されることを防ぐのに役立ちます。

[!code-cshtml[Main](introducing-razor-syntax-c/samples/sample67.cshtml)]

## <a name="additional-resources"></a>その他のリソース

**Visual Basic を使用したプログラミング**


[付録: Visual Basic 言語と構文](https://go.microsoft.com/fwlink/?LinkId=202908)


**リファレンス ドキュメント**


[ASP.NET](https://msdn.microsoft.com/library/ee532866.aspx)

[C# 言語](https://msdn.microsoft.com/library/kx37x362.aspx)
