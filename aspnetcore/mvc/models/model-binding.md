---
title: ASP.NET Core でのモデル バインド
author: rick-anderson
description: ASP.NET Core でのモデル バインドのしくみと、その動作のカスタマイズ方法について説明します。
ms.assetid: 0be164aa-1d72-4192-bd6b-192c9c301164
ms.author: riande
ms.date: 12/18/2019
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
uid: mvc/models/model-binding
ms.openlocfilehash: 5eaedf6dbe5df59848b9cf8a5bda67add48db2a6
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/10/2021
ms.locfileid: "102586944"
---
# <a name="model-binding-in-aspnet-core"></a>ASP.NET Core でのモデル バインド

::: moniker range=">= aspnetcore-3.0"

この記事では、モデル バインドとは何か、そのしくみ、その動作のカスタマイズ方法を説明します。

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/mvc/models/model-binding/samples)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

## <a name="what-is-model-binding"></a>モデル バインドとは何か

コントローラーと Razor ページは、HTTP 要求から取得されたデータを処理します。 たとえば、ルート データからはレコード キーが提供され、ポストされたフォーム フィールドからはモデルのプロパティ用の値が提供されます。 これらの各値を取得してそれらを文字列から .NET 型に変換するためのコードを記述するのは、面倒で間違いも起こりやすいでしょう。 モデル バインドを使用すれば、このプロセスを自動化できます。 モデル バインド システムでは次のことが行われます。

* ルート データ、フォーム フィールド、クエリ文字列などのさまざまなソースからデータを取得します。
* Razorメソッドパラメーターおよびパブリックプロパティのデータをコントローラーおよびページに提供します。
* 文字列データを .NET 型に変換します。
* 複合型のプロパティを更新します。

## <a name="example"></a>例

次のアクション メソッドがあるとします。

[!code-csharp[](model-binding/samples/3.x/ModelBindingSample/Controllers/PetsController.cs?name=snippet_DogsOnly)]

さらにアプリでは、この URL を使用して要求が受信されます。

```
http://contoso.com/api/pets/2?DogsOnly=true
```

ルーティング システムでアクション メソッドが選択されたら、モデル バインドでは次の手順が実行されます。

* `GetByID` の最初のパラメーター (`id` という名前の整数型) を検索します。
* HTTP 要求内で利用可能なソースを調べ、ルート データ内で `id` = "2" を検索します。
* 文字列 "2" を整数 2 に変換します。
* `GetByID` の次のパラメーター (`dogsOnly` という名前のブール型) を検索します。
* 該当するソース内を調べ、クエリ文字列内で "DogsOnly=true" を検索します。 名前の照合では大文字と小文字が区別されません。
* 文字列 "true" をブール型の `true` に変換します。

次にフレームワークによって `GetById` メソッドが呼び出され、`id` パラメーターには 2 が、`dogsOnly` パラメーターには `true` が渡されます。

上記の例で、モデル バインディング ターゲットは単純型のメソッド パラメーターになっています。 ターゲットは複合型のプロパティになる場合もあります。 各プロパティが正常にバインドされたら、そのプロパティに対して[モデル検証](xref:mvc/models/validation)が行われます。 どのようなデータがモデルにバインドされているかを示す記録、バインド エラー、または検証のエラーは、[ControllerBase.ModelState](xref:Microsoft.AspNetCore.Mvc.ControllerBase.ModelState) または [PageModel.ModelState](xref:Microsoft.AspNetCore.Mvc.ControllerBase.ModelState) に格納されます。 このプロセスが正常終了したかどうかを確認するために、アプリでは [ModelState.IsValid](xref:Microsoft.AspNetCore.Mvc.ModelBinding.ModelStateDictionary.IsValid) フラグが調べられます。

## <a name="targets"></a>対象サーバー

モデル バインドでは、次の種類のターゲットの値について検索が試みられます。

* 要求のルーティング先であるコントローラー アクション メソッドのパラメーター。
* Razor要求がルーティングされるページハンドラーメソッドのパラメーター。 
* 属性によって指定されている場合は、コントローラーまたは `PageModel` クラスのパブリック プロパティ。

### <a name="bindproperty-attribute"></a>[BindProperty] 属性

コントローラーまたは `PageModel` クラスのパブリック プロパティに適用できます。これによってモデル バインドはそのプロパティをターゲットとするようになります。

[!code-csharp[](model-binding/samples/3.x/ModelBindingSample/Pages/Instructors/Edit.cshtml.cs?name=snippet_BindProperty&highlight=3-4)]

### <a name="bindproperties-attribute"></a>[BindProperties] 属性

ASP.NET Core 2.1 以降で使用できます。  コントローラーまたは `PageModel` クラスに適用できます。これによってモデル バインドはクラスのすべてのパブリック プロパティをターゲットとするように指示されます。

[!code-csharp[](model-binding/samples/3.x/ModelBindingSample/Pages/Instructors/Create.cshtml.cs?name=snippet_BindProperties&highlight=1-2)]

### <a name="model-binding-for-http-get-requests"></a>HTTP GET 要求のモデル バインド

既定では、プロパティは HTTP GET 要求にバインドされません。 通常、GET 要求に必要なのはレコード ID パラメーターのみです。 レコード ID は、データベース内の項目の検索に使用されます。 そのため、モデルのインスタンスを保持するプロパティをバインドする必要はありません。 GET 要求からのデータにプロパティがバインドされるようにするシナリオでは、`SupportsGet` プロパティを `true` に設定します。

[!code-csharp[](model-binding/samples/3.x/ModelBindingSample/Pages/Instructors/Index.cshtml.cs?name=snippet_SupportsGet)]

## <a name="sources"></a>変換元

既定では、モデル バインドでは HTTP 要求内の次のソースからキーと値のペアの形式でデータが取得されます。

1. フォーム フィールド
1. 要求本文 ([[ApiController] 属性を持つコントローラー](xref:web-api/index#binding-source-parameter-inference) の場合)。
1. ルート データ
1. クエリ文字列パラメーター
1. アップロード済みのファイル

ターゲット パラメーターまたはプロパティごとに、前述の一覧に示されている順序でソースがスキャンされます。 次のようにいくつかの例外があります。

* ルート データとクエリ文字列の値は単純型にのみ使用されます。
* アップロード済みのファイルは、`IFormFile` または `IEnumerable<IFormFile>` を実装するターゲットの種類にのみバインドされます。

既定のソースが正しくない場合は、次のいずれかの属性を使用してソースを指定します。

* [`[FromQuery]`](xref:Microsoft.AspNetCore.Mvc.FromQueryAttribute) -クエリ文字列から値を取得します。 
* [`[FromRoute]`](xref:Microsoft.AspNetCore.Mvc.FromRouteAttribute) -ルートデータから値を取得します。
* [`[FromForm]`](xref:Microsoft.AspNetCore.Mvc.FromFormAttribute) -ポストされたフォームフィールドから値を取得します。
* [`[FromBody]`](xref:Microsoft.AspNetCore.Mvc.FromBodyAttribute) -要求本文から値を取得します。
* [`[FromHeader]`](xref:Microsoft.AspNetCore.Mvc.FromHeaderAttribute) -HTTP ヘッダーから値を取得します。

これらの属性: 

* 次の例のように、(モデル クラスにではなく) モデル プロパティに個別に追加されます。

  [!code-csharp[](model-binding/samples/3.x/ModelBindingSample/Models/Instructor.cs?name=snippet_FromQuery&highlight=5-6)]

* 必要に応じて、コンストラクター内のモデル名の値を受け取ります。 このオプションは、プロパティ名と要求内の値とが一致しない場合に指定されます。 たとえば、要求内の値は、次の例のようにその名前にハイフンが含まれている場合、ヘッダーである可能性があります。

  [!code-csharp[](model-binding/samples/3.x/ModelBindingSample/Pages/Instructors/Index.cshtml.cs?name=snippet_FromHeader)]

### <a name="frombody-attribute"></a>[FromBody] 属性

`[FromBody]` 属性をパラメーターに適用すると、HTTP 要求の本文からそのプロパティが設定されます。 ASP.NET Core ランタイムでは、本文を読み取る責任が入力フォーマッタに委任されます。 入力フォーマッタについては、[この記事で後ほど](#input-formatters)説明します。

`[FromBody]` を複合型パラメーターに適用すると、そのプロパティに適用されているバインディング ソース属性はいずれも無視されます。 たとえば、次の `Create` アクションでは、その `pet` パラメーターを本文から設定するように指定されています。

```csharp
public ActionResult<Pet> Create([FromBody] Pet pet)
```

`Pet` クラスでは、`Breed` プロパティをクエリ文字列パラメーターから設定するように指定されています。

```csharp
public class Pet
{
    public string Name { get; set; }

    [FromQuery] // Attribute is ignored.
    public string Breed { get; set; }
}
```

前の例の場合:

* `[FromQuery]` 属性は無視されます。
* `Breed` プロパティは、クエリ文字列パラメーターから設定されません。 

入力フォーマッタでは本文のみが読み取られ、バインディング ソース属性は認識されません。 本文内で適切な値が見つかった場合は、その値を使用して `Breed` プロパティが設定されます。

アクション メソッドごとに `[FromBody]` を複数のパラメーターに適用しないでください。 入力フォーマッタによって要求ストリームが読み取られると、他の `[FromBody]` パラメーターをバインドするためにそれを再度読み取ることはできません。

### <a name="additional-sources"></a>その他のソース

ソース データは、"*値プロバイダー*" によってモデル バインド システムに提供されます。 モデル バインド用に、他のソースからデータを取得するカスタムの値プロバイダーを作成して登録することができます。 たとえば、またはセッション状態のデータが必要になる場合があり cookie ます。 新しいソースからデータを取得するには: 

* `IValueProvider` を実装するクラスを作成します。
* `IValueProviderFactory` を実装するクラスを作成します。
* `Startup.ConfigureServices` 内のファクトリ クラスを登録します。

サンプルアプリには、s から値を取得する [値プロバイダー](https://github.com/dotnet/AspNetCore.Docs/blob/main/aspnetcore/mvc/models/model-binding/samples/3.x/ModelBindingSample/CookieValueProvider.cs) と [ファクトリ](https://github.com/dotnet/AspNetCore.Docs/blob/main/aspnetcore/mvc/models/model-binding/samples/3.x/ModelBindingSample/CookieValueProviderFactory.cs) の例が含まれてい cookie ます。 `Startup.ConfigureServices` 内の登録コードを次に示します。

[!code-csharp[](model-binding/samples/3.x/ModelBindingSample/Startup.cs?name=snippet_ValueProvider&highlight=4)]

表示したコードでは、すべての組み込み値プロバイダーの後にカスタムの値プロバイダーが配置されています。  それをリストの最初に持ってくるには、`Add` ではなく `Insert(0, new CookieValueProviderFactory())` を呼び出します。

## <a name="no-source-for-a-model-property"></a>モデル プロパティ用のソースがない

既定では、モデル プロパティ用の値が見つからない場合、モデル状態エラーは作成されません。 プロパティは次のように null 値または既定値に設定されます。

* null 許容単純型は `null` に設定されます。
* null 非許容値型は `default(T)` に設定されます。 たとえば、パラメーター `int id` は 0 に設定されます。
* 複合型の場合、モデル バインドでは、プロパティを設定せずに既定のコンストラクターを使用して、インスタンスが作成されます。
* 配列は `Array.Empty<T>()` に設定されます。例外として、`byte[]` 配列は `null` に設定されます。

モデルプロパティのフォームフィールドに何も見つからない場合にモデルの状態を無効にする必要がある場合は、属性を使用し [`[BindRequired]`](#bindrequired-attribute) ます。

この `[BindRequired]` 動作は、要求本文内の JSON または XML データに対してではなく、ポストされたフォーム データからのモデル バインドに適用されることに注意してください。 要求本文データは、[入力フォーマッタ](#input-formatters)によって処理されます。

## <a name="type-conversion-errors"></a>型変換エラー

ソースは見つかってもそれをターゲットの種類に変換できない場合、無効であることを示すフラグがモデル状態に付けられます。 前のセクションで説明したように、ターゲットのパラメーターまたはプロパティは null または既定値に設定されます。

`[ApiController]` 属性を持つ API コントローラーでは、モデル状態が無効であると、HTTP 400 の自動応答が生成されます。

ページで、次のエラーメッセージが表示さ Razor れたページを再び表示します。

[!code-csharp[](model-binding/samples/3.x/ModelBindingSample/Pages/Instructors/Create.cshtml.cs?name=snippet_HandleMBError&highlight=3-6)]

クライアント側の検証では、ページフォームに送信される可能性のあるほとんどの不適切なデータをキャッチ Razor します。 この検証により、前の強調表示されたコードをトリガーするのが難しくなります。 サンプル アプリには、**[Submit with Invalid Date]** ボタンが含まれており、これを使用すると、**[Hire Date]** フィールドに不適切なデータが入力され、そのフォームが送信されます。 このボタンを使用すると、データ変換エラーが発生したときにページを再表示するためのコードがどのように機能するかを表示できます。

上のコードでページが再表示されると、無効な入力はフォーム フィールドに表示されません。 これは、モデル プロパティが null または既定値に設定されているためです。 無効な入力はエラー メッセージに表示されます。 しかし、フォーム フィールドに不適切なデータを再表示したい場合は、モデル プロパティを文字列にしてデータ変換を手動で行うことを検討してください。

型変換エラーが結果的にモデル状態エラーになることを望まない場合も同じ方法をお勧めします。 その場合は、モデル プロパティを文字列にします。

## <a name="simple-types"></a>単純型

モデル バインダーでソース文字列の変換先とすることができる単純型には次のものがあります。

* [Boolean](xref:System.ComponentModel.BooleanConverter)
* [Byte](xref:System.ComponentModel.ByteConverter)、[SByte](xref:System.ComponentModel.SByteConverter)
* [Char](xref:System.ComponentModel.CharConverter)
* [DateTime](xref:System.ComponentModel.DateTimeConverter)
* [DateTimeOffset](xref:System.ComponentModel.DateTimeOffsetConverter)
* [Decimal](xref:System.ComponentModel.DecimalConverter)
* [Double](xref:System.ComponentModel.DoubleConverter)
* [Enum](xref:System.ComponentModel.EnumConverter)
* [Guid](xref:System.ComponentModel.GuidConverter)
* [Int16](xref:System.ComponentModel.Int16Converter)、[Int32](xref:System.ComponentModel.Int32Converter)、[Int64](xref:System.ComponentModel.Int64Converter)
* [Single](xref:System.ComponentModel.SingleConverter)
* [TimeSpan](xref:System.ComponentModel.TimeSpanConverter)
* [UInt16](xref:System.ComponentModel.UInt16Converter)、[UInt32](xref:System.ComponentModel.UInt32Converter)、[UInt64](xref:System.ComponentModel.UInt64Converter)
* [Uri](xref:System.UriTypeConverter)
* [Version](xref:System.ComponentModel.VersionConverter)

## <a name="complex-types"></a>複合型

複合型には、バインドする既定のパブリック コンストラクターと書き込み可能なパブリック プロパティが必要です。 モデル バインドが行われると、クラスは既定のパブリック コンストラクターを使用してインスタンス化されます。 

複合型のプロパティごとに、モデル バインドでは名前パターン *prefix.property_name* がないかソースが調べられます。 何も見つからない場合は、プレフィックスなしで *property_name* だけが探索されます。

パラメーターにバインドする場合、プレフィックスはパラメーター名です。 `PageModel` パブリック プロパティにバインドする場合、プレフィックスはパブリック プロパティ名です。 一部の属性には、パラメーター名またはプロパティ名の既定の使用をオーバーライドするための `Prefix` プロパティがあります。

たとえば、複合型が次の `Instructor` クラスであるとします。

  ```csharp
  public class Instructor
  {
      public int ID { get; set; }
      public string LastName { get; set; }
      public string FirstName { get; set; }
  }
  ```

### <a name="prefix--parameter-name"></a>プレフィックス = パラメーター名

バインドされるモデルが `instructorToUpdate` という名前のパラメーターである場合: 

```csharp
public IActionResult OnPost(int? id, Instructor instructorToUpdate)
```

モデル バインドでは、キー `instructorToUpdate.ID` がないかソースを調べることから始まります。 見つからない場合は、プレフィックスなしで `ID` が探索されます。

### <a name="prefix--property-name"></a>プレフィックス = プロパティ名

バインドされるモデルがコントローラーの `Instructor` という名前のプロパティか、または `PageModel` クラスである場合: 

```csharp
[BindProperty]
public Instructor Instructor { get; set; }
```

モデル バインドでは、キー `Instructor.ID` がないかソースを調べることから始まります。 見つからない場合は、プレフィックスなしで `ID` が探索されます。

### <a name="custom-prefix"></a>カスタム プレフィックス

バインドされるモデルが `instructorToUpdate` という名前のパラメーターであり、かつ `Bind` 属性でプレフィックスとして `Instructor` が指定されている場合: 

```csharp
public IActionResult OnPost(
    int? id, [Bind(Prefix = "Instructor")] Instructor instructorToUpdate)
```

モデル バインドでは、キー `Instructor.ID` がないかソースを調べることから始まります。 見つからない場合は、プレフィックスなしで `ID` が探索されます。

### <a name="attributes-for-complex-type-targets"></a>複合型のターゲットの属性

複合型のモデル バインドを制御するために利用できる組み込みの属性がいくつかあります。

* `[Bind]`
* `[BindRequired]`
* `[BindNever]`

> [!WARNING]
> ポストされたフォーム データが値のソースである場合、これらの属性はモデル バインドに影響します。 これらは、ポストされた JSON および XML 要求本文を処理する入力フォーマッタには影響し ***ません*** 。 入力フォーマッタについては、[この記事で後ほど](#input-formatters)説明します。

### <a name="bind-attribute"></a>[Bind] 属性

クラスまたはメソッド パラメーターに適用できます。 モデルのどのプロパティをモデル バインドに含めるかを指定します。 `[Bind]` 入力フォーマッタには影響し ***ません*** 。

次の例では、任意のハンドラーまたはアクション メソッドが呼び出されると、`Instructor` モデルの指定されたプロパティのみがバインドされます。

```csharp
[Bind("LastName,FirstMidName,HireDate")]
public class Instructor
```

次の例では、`OnPost` メソッドが呼び出されると、`Instructor` モデルの指定されたプロパティのみがバインドされます。

```csharp
[HttpPost]
public IActionResult OnPost([Bind("LastName,FirstMidName,HireDate")] Instructor instructor)
```

`[Bind]` 属性を使用すれば、"*作成*" シナリオにおいて過剰ポスティングから保護することができます。 除外されたプロパティはそのままにしておくのではなく null または既定値に設定されるので、この属性は編集シナリオではうまく機能しません。 過剰ポスティングを防ぐ場合は、`[Bind]` 属性ではなくビュー モデルをお勧めします。 詳細については、「[過剰ポスティングに関するセキュリティの注意事項](xref:data/ef-mvc/crud#security-note-about-overposting)」を参照してください。

### <a name="modelbinder-attribute"></a>[ModelBinder] 属性

<xref:Microsoft.AspNetCore.Mvc.ModelBinderAttribute> 型、プロパティ、またはパラメーターに適用できます。 特定のインスタンスまたは型をバインドするために使用するモデルバインダーの種類を指定できます。 次に例を示します。

```C#
[HttpPost]
public IActionResult OnPost([ModelBinder(typeof(MyInstructorModelBinder))] Instructor instructor)
```

属性を使用して、 `[ModelBinder]` モデルがバインドされているときに、プロパティまたはパラメーターの名前を変更することもできます。

```C#
public class Instructor
{
    [ModelBinder(Name = "instructor_id")]
    public string Id { get; set; }
    
    public string Name { get; set; }
}
```

### <a name="bindrequired-attribute"></a>[BindRequired] 属性

モデルのプロパティにのみに適用でき、メソッドのパラメーターには適用できません。 モデルのプロパティに対してバインドを実行できない場合に、モデル バインドがモデル状態エラーを追加できるようにします。 次に例を示します。

[!code-csharp[](model-binding/samples/3.x/ModelBindingSample/Models/InstructorWithCollection.cs?name=snippet_BindRequired&highlight=8-9)]

[モデル検証](xref:mvc/models/validation#required-attribute)に関するページにある `[Required]` 属性の説明も参照してください。

### <a name="bindnever-attribute"></a>[BindNever] 属性

モデルのプロパティにのみに適用でき、メソッドのパラメーターには適用できません。 モデル バインドがモデルのプロパティを設定できないようにします。 次に例を示します。

[!code-csharp[](model-binding/samples/3.x/ModelBindingSample/Models/InstructorWithDictionary.cs?name=snippet_BindNever&highlight=3-4)]

## <a name="collections"></a>コレクション

ターゲットが単純型のコレクションである場合、モデル バインドでは *parameter_name* または *property_name* との一致が探索されます。 一致が見つからない場合は、サポートされているいずれかの形式がプレフィックスなしで探索されます。 次に例を示します。

* バインドされるパラメーターが `selectedCourses` という名前の配列であるとした場合: 

  ```csharp
  public IActionResult OnPost(int? id, int[] selectedCourses)
  ```

* フォームまたはクエリ文字列データは、次のいずれかの形式とすることができます。
   
  ```
  selectedCourses=1050&selectedCourses=2000 
  ```

  ```
  selectedCourses[0]=1050&selectedCourses[1]=2000
  ```

  ```
  [0]=1050&[1]=2000
  ```

  ```
  selectedCourses[a]=1050&selectedCourses[b]=2000&selectedCourses.index=a&selectedCourses.index=b
  ```

  ```
  [a]=1050&[b]=2000&index=a&index=b
  ```

* 次の形式は、フォーム データでのみサポートされます。

  ```
  selectedCourses[]=1050&selectedCourses[]=2000
  ```

* 上記のすべてのフォーマット例において、モデル バインドでは 2 つの項目から成る配列が `selectedCourses` パラメーターに渡されます。

  * selectedCourses[0]=1050
  * selectedCourses[1]=2000

  添え字番号 (... [0] ... [1] ...) を使用するデータ フォーマットでは、確実にそれらがゼロから始まる連続した番号になるようにする必要があります。 添え字の番号付けで欠落している番号がある場合、欠落している番号の後の項目はすべて無視されます。 たとえば、添え字が 0、1 の並びではなく、0、2 の並びで振られている場合、2 番目の項目は無視されます。

## <a name="dictionaries"></a>辞書

`Dictionary` ターゲットの場合、モデル バインドでは *parameter_name* または *property_name* との一致が探索されます。 一致が見つからない場合は、サポートされているいずれかの形式がプレフィックスなしで探索されます。 次に例を示します。

* ターゲット パラメーターが `selectedCourses` という名前の `Dictionary<int, string>` であるとします: 

  ```csharp
  public IActionResult OnPost(int? id, Dictionary<int, string> selectedCourses)
  ```

* ポストされたフォームまたはクエリ文字列データは、次のいずれかの例のようになります。

  ```
  selectedCourses[1050]=Chemistry&selectedCourses[2000]=Economics
  ```

  ```
  [1050]=Chemistry&selectedCourses[2000]=Economics
  ```

  ```
  selectedCourses[0].Key=1050&selectedCourses[0].Value=Chemistry&
  selectedCourses[1].Key=2000&selectedCourses[1].Value=Economics
  ```

  ```
  [0].Key=1050&[0].Value=Chemistry&[1].Key=2000&[1].Value=Economics
  ```

* 上記のすべてのフォーマット例において、モデル バインドでは 2 つの項目から成る辞書が `selectedCourses` パラメーターに渡されます。

  * selectedCourses["1050"]="Chemistry"
  * selectedCourses["2000"]="Economics"
  
::: moniker-end

::: moniker range=">= aspnetcore-5.0"

## <a name="constructor-binding-and-record-types"></a>コンストラクターのバインドとレコードの種類

モデルバインドでは、複合型にパラメーターなしのコンストラクターが含まれている必要があります。 `System.Text.Json`と `Newtonsoft.Json` ベースの入力フォーマッタはどちらも、パラメーターなしのコンストラクターを持たないクラスの逆シリアル化をサポートしています。 

C# 9 では、レコードの種類が導入されています。これは、ネットワーク上でデータを簡潔に表現するための優れた方法です。 ASP.NET Core は、1つのコンストラクターを使用して、モデルバインドとレコードの種類の検証のサポートを追加します。

```csharp
public record Person([Required] string Name, [Range(0, 150)] int Age);

public class PersonController
{
   public IActionResult Index() => View();

   [HttpPost]
   public IActionResult Index(Person person)
   {
       ...
   }
}
```

`Person/Index.cshtml`:

```cshtml
@model Person

Name: <input asp-for="Name" />
...
Age: <input asp-for="Age" />
```

レコードの種類を検証するとき、ランタイムは、プロパティではなく、パラメーターに対して明示的に検証メタデータを検索します。

::: moniker-end

::: moniker range=">= aspnetcore-3.0"

<a name="glob"></a>

## <a name="globalization-behavior-of-model-binding-route-data-and-query-strings"></a>モデル バインド ルート データとクエリ文字列のグローバリゼーション動作

ASP.NET Core ルート値プロバイダーとクエリ文字列値プロバイダーでは、次のことが行われます。

* 値をインバリアント カルチャとして扱います。
* URL はカルチャに依存しないものと想定します。

これに対し、フォーム データからの値は、カルチャに依存した変換にかけられます。 URL がロケール間で共有可能なように、設計上そのようになっています。

ASP.NET Core ルート値プロバイダーとクエリ文字列値プロバイダーでカルチャ依存の変換が行われるようにするには、次のようにします。

* <xref:Microsoft.AspNetCore.Mvc.ModelBinding.IValueProviderFactory> から継承します
* [QueryStringValueProviderFactory](https://github.com/dotnet/AspNetCore/blob/main/src/Mvc/Mvc.Core/src/ModelBinding/QueryStringValueProviderFactory.cs) または [RouteValueValueProviderFactory](https://github.com/dotnet/AspNetCore/blob/main/src/Mvc/Mvc.Core/src/ModelBinding/RouteValueProviderFactory.cs) からコードをコピーします。
* 値プロバイダー コンストラクターに渡された[カルチャ値](https://github.com/dotnet/AspNetCore/blob/e625fe29b049c60242e8048b4ea743cca65aa7b5/src/Mvc/Mvc.Core/src/ModelBinding/QueryStringValueProviderFactory.cs#L30)を [CultureInfo.CurrentCulture](xref:System.Globalization.CultureInfo.CurrentCulture) に置き換えます。
* MVC オプション内の既定値プロバイダー ファクトリを、新しいものに置き換えます。

[!code-csharp[](model-binding/samples_snapshot/3.x/Startup.cs?name=snippet)]
[!code-csharp[](model-binding/samples_snapshot/3.x/Startup.cs?name=snippet1)]

## <a name="special-data-types"></a>特別なデータ型

モデル バインドで処理できる特殊なデータ型がいくつかあります。

### <a name="iformfile-and-iformfilecollection"></a>IFormFile と IFormFileCollection

HTTP 要求に含まれたアップロード済みファイル。  また、複数のファイルに対して `IEnumerable<IFormFile>` もサポートされています。

### <a name="cancellationtoken"></a>CancellationToken

アクションでは、オプションでを `CancellationToken` パラメーターとしてバインドできます。 これ <xref:Microsoft.AspNetCore.Http.HttpContext.RequestAborted> は、HTTP 要求の基になる接続が中止されたときに、そのことを通知します。 アクションでは、このパラメーターを使用して、コントローラーアクションの一部として実行される実行時間の長い非同期操作を取り消すことができます。

### <a name="formcollection"></a>FormCollection

ポストされたフォーム データからすべての値を取得するために使用します。

## <a name="input-formatters"></a>入力フォーマッタ

要求本文内のデータは、JSON、XML、またはその他のいくつかの形式にすることができます。 このデータを解析するために、モデル バインドでは、特定のコンテンツの種類を処理するように構成された "*入力フォーマッタ*" が使用されます。 既定では、ASP.NET Core には JSON データ処理用の JSON ベースの入力フォーマッタが含まれます。 他のコンテンツの種類については対応する他のフォーマッタを追加することができます。

ASP.NET Core では、[Consumes](xref:Microsoft.AspNetCore.Mvc.ConsumesAttribute) 属性に基づいて入力フォーマッタが選択されます。 属性が存在しない場合は、[Content-Type ヘッダー](https://www.w3.org/Protocols/rfc1341/4_Content-Type.html)が使用されます。

組み込みの XML 入力フォーマッタを使用するには: 

* `Microsoft.AspNetCore.Mvc.Formatters.Xml` NuGet パッケージをインストールします。

* `Startup.ConfigureServices` で、<xref:Microsoft.Extensions.DependencyInjection.MvcXmlMvcCoreBuilderExtensions.AddXmlSerializerFormatters*> または <xref:Microsoft.Extensions.DependencyInjection.MvcXmlMvcCoreBuilderExtensions.AddXmlDataContractSerializerFormatters*> を呼び出します。

  [!code-csharp[](model-binding/samples/3.x/ModelBindingSample/Startup.cs?name=snippet_ValueProvider&highlight=10)]

* 要求本文で XML を必要とするコントローラー クラスまたはアクション メソッドに `Consumes` 属性を適用します。

  ```csharp
  [HttpPost]
  [Consumes("application/xml")]
  public ActionResult<Pet> Create(Pet pet)
  ```

  詳細については、「[XML シリアル化の概要](/dotnet/standard/serialization/introducing-xml-serialization)」を参照してください。

### <a name="customize-model-binding-with-input-formatters"></a>入力フォーマッタを使用してモデル バインドをカスタマイズする

入力フォーマッタは、要求本文からデータを読み取るためのすべての役割を担います。 このプロセスをカスタマイズするには、入力フォーマッタによって使用される API を構成します。 このセクションでは、`ObjectId` という名前のカスタム型を理解するために、`System.Text.Json` ベースの入力フォーマッタをカスタマイズする方法について説明します。 

`Id` という名前のカスタム `ObjectId` プロパティが含まれている、次のモデルを考えてみます。

[!code-csharp[](model-binding/samples/3.x/ModelBindingSample/Models/ModelWithObjectId.cs?name=snippet_Class&highlight=3)]

`System.Text.Json` を使用する際のモデル バインド プロセスをカスタマイズするために、<xref:System.Text.Json.Serialization.JsonConverter%601> から派生するクラスを作成します。

[!code-csharp[](model-binding/samples/3.x/ModelBindingSample/JsonConverters/ObjectIdConverter.cs?name=snippet_Class)]

カスタム コンバーターを使用するために、型に <xref:System.Text.Json.Serialization.JsonConverterAttribute> 属性を適用します。 次の例では、`ObjectId` 型は、`ObjectIdConverter` をそのカスタム コンバーターとして構成されています。

[!code-csharp[](model-binding/samples/3.x/ModelBindingSample/Models/ObjectId.cs?name=snippet_Class&highlight=1)]

詳細については、[カスタム コンバーターを記述する方法](/dotnet/standard/serialization/system-text-json-converters-how-to)に関する記事をご覧ください。

## <a name="exclude-specified-types-from-model-binding"></a>指定された型をモデル バインドから除外する

モデル バインドおよび検証システムの動作は、[ModelMetadata](/dotnet/api/microsoft.aspnetcore.mvc.modelbinding.modelmetadata) によって駆動されます。 `ModelMetadata` については、詳細プロバイダーを [MvcOptions.ModelMetadataDetailsProviders](xref:Microsoft.AspNetCore.Mvc.MvcOptions.ModelMetadataDetailsProviders) に追加してカスタマイズできます。 組み込みの詳細プロバイダーは、指定された型に対してモデル バインドまたは検証を無効にする場合に使用できます。

指定された型のすべてのモデルに対してモデル バインドを無効にするには、`Startup.ConfigureServices` に <xref:Microsoft.AspNetCore.Mvc.ModelBinding.Metadata.ExcludeBindingMetadataProvider> を追加します。 たとえば、`System.Version` 型のすべてのモデルに対してモデル バインドを無効にするには、次のようにします。

[!code-csharp[](model-binding/samples/3.x/ModelBindingSample/Startup.cs?name=snippet_ValueProvider&highlight=5-6)]

指定された型のプロパティに対して検証を無効にするには、`Startup.ConfigureServices` に <xref:Microsoft.AspNetCore.Mvc.ModelBinding.SuppressChildValidationMetadataProvider> を追加します。 たとえば、`System.Guid` 型のプロパティに対して検証を無効にするには、次のようにします。

[!code-csharp[](model-binding/samples/3.x/ModelBindingSample/Startup.cs?name=snippet_ValueProvider&highlight=7-8)]

## <a name="custom-model-binders"></a>カスタム モデル バインダー

モデル バインドを拡張するには、カスタム モデル バインダーを記述し、`[ModelBinder]` 属性を使用してそれを特定のターゲット向けに選択します。 詳細については、「[custom model binding](xref:mvc/advanced/custom-model-binding)」 (カスタム モデル バインド) を参照してください。

## <a name="manual-model-binding"></a>手動によるモデル バインド 

モデル バインドは、<xref:Microsoft.AspNetCore.Mvc.ControllerBase.TryUpdateModelAsync*> メソッドを使用して手動で呼び出すことができます。 このメソッドは `ControllerBase` クラスと `PageModel` クラスの両方で定義されています。 メソッドのオーバーロードにより、使用するプレフィックスと値プロバイダーを指定できます。 モデル バインドが失敗した場合は、メソッドから `false` が返されます。 次に例を示します。

[!code-csharp[](model-binding/samples/3.x/ModelBindingSample/Pages/InstructorsWithCollection/Create.cshtml.cs?name=snippet_TryUpdate&highlight=1-4)]

<xref:Microsoft.AspNetCore.Mvc.ControllerBase.TryUpdateModelAsync*> では、フォーム本文、クエリ文字列、およびルート データからデータを取得するために、値プロバイダーが使用されます。 `TryUpdateModelAsync` は通常、次のようになります。 

* Razor過剰ポストを防ぐために、コントローラーとビューを使用してページおよび MVC アプリで使用します。
* フォーム データ、クエリ文字列、およびルート データから使用される場合を除き、Web API では使用されません。 JSON を使用する Web API エンドポイントでは、[入力フォーマッタ](#input-formatters)を使用して要求本文がオブジェクトに逆シリアル化されます。

詳細については、「[TryUpdateModelAsync](xref:data/ef-rp/crud#TryUpdateModelAsync)」をご覧ください。

## <a name="fromservices-attribute"></a>[FromServices] 属性

この属性の名前は、データ ソースを指定するモデル バインド属性のパターンに従います。 ただし、それは、値プロバイダーからのデータ バインドを説明するものではありません。 [依存関係挿入](xref:fundamentals/dependency-injection)コンテナーから型のインスタンスが取得されます。 その目的は、特定のメソッドが呼び出された場合にのみサービスを必要するときにコンストラクターの挿入の代替手段を提供することにあります。

## <a name="additional-resources"></a>その他のリソース

* <xref:mvc/models/validation>
* <xref:mvc/advanced/custom-model-binding>

::: moniker-end

::: moniker range="< aspnetcore-3.0"

この記事では、モデル バインドとは何か、そのしくみ、その動作のカスタマイズ方法を説明します。

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/mvc/models/model-binding/samples)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

## <a name="what-is-model-binding"></a>モデル バインドとは何か

コントローラーと Razor ページは、HTTP 要求から取得されたデータを処理します。 たとえば、ルート データからはレコード キーが提供され、ポストされたフォーム フィールドからはモデルのプロパティ用の値が提供されます。 これらの各値を取得してそれらを文字列から .NET 型に変換するためのコードを記述するのは、面倒で間違いも起こりやすいでしょう。 モデル バインドを使用すれば、このプロセスを自動化できます。 モデル バインド システムでは次のことが行われます。

* ルート データ、フォーム フィールド、クエリ文字列などのさまざまなソースからデータを取得します。
* Razorメソッドパラメーターおよびパブリックプロパティのデータをコントローラーおよびページに提供します。
* 文字列データを .NET 型に変換します。
* 複合型のプロパティを更新します。

## <a name="example"></a>例

次のアクション メソッドがあるとします。

[!code-csharp[](model-binding/samples/2.x/ModelBindingSample/Controllers/PetsController.cs?name=snippet_DogsOnly)]

さらにアプリでは、この URL を使用して要求が受信されます。

```
http://contoso.com/api/pets/2?DogsOnly=true
```

ルーティング システムでアクション メソッドが選択されたら、モデル バインドでは次の手順が実行されます。

* `GetByID` の最初のパラメーター (`id` という名前の整数型) を検索します。
* HTTP 要求内で利用可能なソースを調べ、ルート データ内で `id` = "2" を検索します。
* 文字列 "2" を整数 2 に変換します。
* `GetByID` の次のパラメーター (`dogsOnly` という名前のブール型) を検索します。
* 該当するソース内を調べ、クエリ文字列内で "DogsOnly=true" を検索します。 名前の照合では大文字と小文字が区別されません。
* 文字列 "true" をブール型の `true` に変換します。

次にフレームワークによって `GetById` メソッドが呼び出され、`id` パラメーターには 2 が、`dogsOnly` パラメーターには `true` が渡されます。

上記の例で、モデル バインディング ターゲットは単純型のメソッド パラメーターになっています。 ターゲットは複合型のプロパティになる場合もあります。 各プロパティが正常にバインドされたら、そのプロパティに対して[モデル検証](xref:mvc/models/validation)が行われます。 どのようなデータがモデルにバインドされているかを示す記録、バインド エラー、または検証のエラーは、[ControllerBase.ModelState](xref:Microsoft.AspNetCore.Mvc.ControllerBase.ModelState) または [PageModel.ModelState](xref:Microsoft.AspNetCore.Mvc.ControllerBase.ModelState) に格納されます。 このプロセスが正常終了したかどうかを確認するために、アプリでは [ModelState.IsValid](xref:Microsoft.AspNetCore.Mvc.ModelBinding.ModelStateDictionary.IsValid) フラグが調べられます。

## <a name="targets"></a>対象サーバー

モデル バインドでは、次の種類のターゲットの値について検索が試みられます。

* 要求のルーティング先であるコントローラー アクション メソッドのパラメーター。
* Razor要求がルーティングされるページハンドラーメソッドのパラメーター。 
* 属性によって指定されている場合は、コントローラーまたは `PageModel` クラスのパブリック プロパティ。

### <a name="bindproperty-attribute"></a>[BindProperty] 属性

コントローラーまたは `PageModel` クラスのパブリック プロパティに適用できます。これによってモデル バインドはそのプロパティをターゲットとするようになります。

[!code-csharp[](model-binding/samples/2.x/ModelBindingSample/Pages/Instructors/Edit.cshtml.cs?name=snippet_BindProperty&highlight=3-4)]

### <a name="bindproperties-attribute"></a>[BindProperties] 属性

ASP.NET Core 2.1 以降で使用できます。  コントローラーまたは `PageModel` クラスに適用できます。これによってモデル バインドはクラスのすべてのパブリック プロパティをターゲットとするように指示されます。

[!code-csharp[](model-binding/samples/2.x/ModelBindingSample/Pages/Instructors/Create.cshtml.cs?name=snippet_BindProperties&highlight=1-2)]

### <a name="model-binding-for-http-get-requests"></a>HTTP GET 要求のモデル バインド

既定では、プロパティは HTTP GET 要求にバインドされません。 通常、GET 要求に必要なのはレコード ID パラメーターのみです。 レコード ID は、データベース内の項目の検索に使用されます。 そのため、モデルのインスタンスを保持するプロパティをバインドする必要はありません。 GET 要求からのデータにプロパティがバインドされるようにするシナリオでは、`SupportsGet` プロパティを `true` に設定します。

[!code-csharp[](model-binding/samples/2.x/ModelBindingSample/Pages/Instructors/Index.cshtml.cs?name=snippet_SupportsGet)]

## <a name="sources"></a>変換元

既定では、モデル バインドでは HTTP 要求内の次のソースからキーと値のペアの形式でデータが取得されます。

1. フォーム フィールド
1. 要求本文 ([[ApiController] 属性を持つコントローラー](xref:web-api/index#binding-source-parameter-inference) の場合)。
1. ルート データ
1. クエリ文字列パラメーター
1. アップロード済みのファイル

ターゲット パラメーターまたはプロパティごとに、前述の一覧に示されている順序でソースがスキャンされます。 次のようにいくつかの例外があります。

* ルート データとクエリ文字列の値は単純型にのみ使用されます。
* アップロード済みのファイルは、`IFormFile` または `IEnumerable<IFormFile>` を実装するターゲットの種類にのみバインドされます。

既定のソースが正しくない場合は、次のいずれかの属性を使用してソースを指定します。

* [`[FromQuery]`](xref:Microsoft.AspNetCore.Mvc.FromQueryAttribute) -クエリ文字列から値を取得します。 
* [`[FromRoute]`](xref:Microsoft.AspNetCore.Mvc.FromRouteAttribute) -ルートデータから値を取得します。
* [`[FromForm]`](xref:Microsoft.AspNetCore.Mvc.FromFormAttribute) -ポストされたフォームフィールドから値を取得します。
* [`[FromBody]`](xref:Microsoft.AspNetCore.Mvc.FromBodyAttribute) -要求本文から値を取得します。
* [`[FromHeader]`](xref:Microsoft.AspNetCore.Mvc.FromHeaderAttribute) -HTTP ヘッダーから値を取得します。

これらの属性: 

* 次の例のように、(モデル クラスにではなく) モデル プロパティに個別に追加されます。

  [!code-csharp[](model-binding/samples/2.x/ModelBindingSample/Models/Instructor.cs?name=snippet_FromQuery&highlight=5-6)]

* 必要に応じて、コンストラクター内のモデル名の値を受け取ります。 このオプションは、プロパティ名と要求内の値とが一致しない場合に指定されます。 たとえば、要求内の値は、次の例のようにその名前にハイフンが含まれている場合、ヘッダーである可能性があります。

  [!code-csharp[](model-binding/samples/2.x/ModelBindingSample/Pages/Instructors/Index.cshtml.cs?name=snippet_FromHeader)]

### <a name="frombody-attribute"></a>[FromBody] 属性

`[FromBody]` 属性をパラメーターに適用すると、HTTP 要求の本文からそのプロパティが設定されます。 ASP.NET Core ランタイムでは、本文を読み取る責任が入力フォーマッタに委任されます。 入力フォーマッタについては、[この記事で後ほど](#input-formatters)説明します。

`[FromBody]` を複合型パラメーターに適用すると、そのプロパティに適用されているバインディング ソース属性はいずれも無視されます。 たとえば、次の `Create` アクションでは、その `pet` パラメーターを本文から設定するように指定されています。

```csharp
public ActionResult<Pet> Create([FromBody] Pet pet)
```

`Pet` クラスでは、`Breed` プロパティをクエリ文字列パラメーターから設定するように指定されています。

```csharp
public class Pet
{
    public string Name { get; set; }

    [FromQuery] // Attribute is ignored.
    public string Breed { get; set; }
}
```

前の例の場合:

* `[FromQuery]` 属性は無視されます。
* `Breed` プロパティは、クエリ文字列パラメーターから設定されません。 

入力フォーマッタでは本文のみが読み取られ、バインディング ソース属性は認識されません。 本文内で適切な値が見つかった場合は、その値を使用して `Breed` プロパティが設定されます。

アクション メソッドごとに `[FromBody]` を複数のパラメーターに適用しないでください。 入力フォーマッタによって要求ストリームが読み取られると、他の `[FromBody]` パラメーターをバインドするためにそれを再度読み取ることはできません。

### <a name="additional-sources"></a>その他のソース

ソース データは、"*値プロバイダー*" によってモデル バインド システムに提供されます。 モデル バインド用に、他のソースからデータを取得するカスタムの値プロバイダーを作成して登録することができます。 たとえば、またはセッション状態のデータが必要になる場合があり cookie ます。 新しいソースからデータを取得するには: 

* `IValueProvider` を実装するクラスを作成します。
* `IValueProviderFactory` を実装するクラスを作成します。
* `Startup.ConfigureServices` 内のファクトリ クラスを登録します。

サンプルアプリには、s から値を取得する [値プロバイダー](https://github.com/dotnet/AspNetCore.Docs/blob/main/aspnetcore/mvc/models/model-binding/samples/2.x/ModelBindingSample/CookieValueProvider.cs) と [ファクトリ](https://github.com/dotnet/AspNetCore.Docs/blob/main/aspnetcore/mvc/models/model-binding/samples/2.x/ModelBindingSample/CookieValueProviderFactory.cs) の例が含まれてい cookie ます。 `Startup.ConfigureServices` 内の登録コードを次に示します。

[!code-csharp[](model-binding/samples/2.x/ModelBindingSample/Startup.cs?name=snippet_ValueProvider&highlight=3)]

表示したコードでは、すべての組み込み値プロバイダーの後にカスタムの値プロバイダーが配置されています。  それをリストの最初に持ってくるには、`Add` ではなく `Insert(0, new CookieValueProviderFactory())` を呼び出します。

## <a name="no-source-for-a-model-property"></a>モデル プロパティ用のソースがない

既定では、モデル プロパティ用の値が見つからない場合、モデル状態エラーは作成されません。 プロパティは次のように null 値または既定値に設定されます。

* null 許容単純型は `null` に設定されます。
* null 非許容値型は `default(T)` に設定されます。 たとえば、パラメーター `int id` は 0 に設定されます。
* 複合型の場合、モデル バインドでは、プロパティを設定せずに既定のコンストラクターを使用して、インスタンスが作成されます。
* 配列は `Array.Empty<T>()` に設定されます。例外として、`byte[]` 配列は `null` に設定されます。

モデルプロパティのフォームフィールドに何も見つからない場合にモデルの状態を無効にする必要がある場合は、属性を使用し [`[BindRequired]`](#bindrequired-attribute) ます。

この `[BindRequired]` 動作は、要求本文内の JSON または XML データに対してではなく、ポストされたフォーム データからのモデル バインドに適用されることに注意してください。 要求本文データは、[入力フォーマッタ](#input-formatters)によって処理されます。

## <a name="type-conversion-errors"></a>型変換エラー

ソースは見つかってもそれをターゲットの種類に変換できない場合、無効であることを示すフラグがモデル状態に付けられます。 前のセクションで説明したように、ターゲットのパラメーターまたはプロパティは null または既定値に設定されます。

`[ApiController]` 属性を持つ API コントローラーでは、モデル状態が無効であると、HTTP 400 の自動応答が生成されます。

ページで、次のエラーメッセージが表示さ Razor れたページを再び表示します。

[!code-csharp[](model-binding/samples/2.x/ModelBindingSample/Pages/Instructors/Create.cshtml.cs?name=snippet_HandleMBError&highlight=3-6)]

クライアント側の検証では、ページフォームに送信される可能性のあるほとんどの不適切なデータをキャッチ Razor します。 この検証により、前の強調表示されたコードをトリガーするのが難しくなります。 サンプル アプリには、**[Submit with Invalid Date]** ボタンが含まれており、これを使用すると、**[Hire Date]** フィールドに不適切なデータが入力され、そのフォームが送信されます。 このボタンを使用すると、データ変換エラーが発生したときにページを再表示するためのコードがどのように機能するかを表示できます。

上のコードでページが再表示されると、無効な入力はフォーム フィールドに表示されません。 これは、モデル プロパティが null または既定値に設定されているためです。 無効な入力はエラー メッセージに表示されます。 しかし、フォーム フィールドに不適切なデータを再表示したい場合は、モデル プロパティを文字列にしてデータ変換を手動で行うことを検討してください。

型変換エラーが結果的にモデル状態エラーになることを望まない場合も同じ方法をお勧めします。 その場合は、モデル プロパティを文字列にします。

## <a name="simple-types"></a>単純型

モデル バインダーでソース文字列の変換先とすることができる単純型には次のものがあります。

* [Boolean](xref:System.ComponentModel.BooleanConverter)
* [Byte](xref:System.ComponentModel.ByteConverter)、[SByte](xref:System.ComponentModel.SByteConverter)
* [Char](xref:System.ComponentModel.CharConverter)
* [DateTime](xref:System.ComponentModel.DateTimeConverter)
* [DateTimeOffset](xref:System.ComponentModel.DateTimeOffsetConverter)
* [Decimal](xref:System.ComponentModel.DecimalConverter)
* [Double](xref:System.ComponentModel.DoubleConverter)
* [Enum](xref:System.ComponentModel.EnumConverter)
* [Guid](xref:System.ComponentModel.GuidConverter)
* [Int16](xref:System.ComponentModel.Int16Converter)、[Int32](xref:System.ComponentModel.Int32Converter)、[Int64](xref:System.ComponentModel.Int64Converter)
* [Single](xref:System.ComponentModel.SingleConverter)
* [TimeSpan](xref:System.ComponentModel.TimeSpanConverter)
* [UInt16](xref:System.ComponentModel.UInt16Converter)、[UInt32](xref:System.ComponentModel.UInt32Converter)、[UInt64](xref:System.ComponentModel.UInt64Converter)
* [Uri](xref:System.UriTypeConverter)
* [Version](xref:System.ComponentModel.VersionConverter)

## <a name="complex-types"></a>複合型

複合型には、バインドする既定のパブリック コンストラクターと書き込み可能なパブリック プロパティが必要です。 モデル バインドが行われると、クラスは既定のパブリック コンストラクターを使用してインスタンス化されます。 

複合型のプロパティごとに、モデル バインドでは名前パターン *prefix.property_name* がないかソースが調べられます。 何も見つからない場合は、プレフィックスなしで *property_name* だけが探索されます。

パラメーターにバインドする場合、プレフィックスはパラメーター名です。 `PageModel` パブリック プロパティにバインドする場合、プレフィックスはパブリック プロパティ名です。 一部の属性には、パラメーター名またはプロパティ名の既定の使用をオーバーライドするための `Prefix` プロパティがあります。

たとえば、複合型が次の `Instructor` クラスであるとします。

  ```csharp
  public class Instructor
  {
      public int ID { get; set; }
      public string LastName { get; set; }
      public string FirstName { get; set; }
  }
  ```

### <a name="prefix--parameter-name"></a>プレフィックス = パラメーター名

バインドされるモデルが `instructorToUpdate` という名前のパラメーターである場合: 

```csharp
public IActionResult OnPost(int? id, Instructor instructorToUpdate)
```

モデル バインドでは、キー `instructorToUpdate.ID` がないかソースを調べることから始まります。 見つからない場合は、プレフィックスなしで `ID` が探索されます。

### <a name="prefix--property-name"></a>プレフィックス = プロパティ名

バインドされるモデルがコントローラーの `Instructor` という名前のプロパティか、または `PageModel` クラスである場合: 

```csharp
[BindProperty]
public Instructor Instructor { get; set; }
```

モデル バインドでは、キー `Instructor.ID` がないかソースを調べることから始まります。 見つからない場合は、プレフィックスなしで `ID` が探索されます。

### <a name="custom-prefix"></a>カスタム プレフィックス

バインドされるモデルが `instructorToUpdate` という名前のパラメーターであり、かつ `Bind` 属性でプレフィックスとして `Instructor` が指定されている場合: 

```csharp
public IActionResult OnPost(
    int? id, [Bind(Prefix = "Instructor")] Instructor instructorToUpdate)
```

モデル バインドでは、キー `Instructor.ID` がないかソースを調べることから始まります。 見つからない場合は、プレフィックスなしで `ID` が探索されます。

### <a name="attributes-for-complex-type-targets"></a>複合型のターゲットの属性

複合型のモデル バインドを制御するために利用できる組み込みの属性がいくつかあります。

* `[BindRequired]`
* `[BindNever]`
* `[Bind]`

> [!NOTE]
> ポストされたフォーム データが値のソースである場合、これらの属性はモデル バインドに影響します。 ポストされた JSON および XML 要求本文を処理する入力フォーマッタには影響しません。 入力フォーマッタについては、[この記事で後ほど](#input-formatters)説明します。
>
> [モデル検証](xref:mvc/models/validation#required-attribute)に関するページにある `[Required]` 属性の説明も参照してください。

### <a name="bindrequired-attribute"></a>[BindRequired] 属性

モデルのプロパティにのみに適用でき、メソッドのパラメーターには適用できません。 モデルのプロパティに対してバインドを実行できない場合に、モデル バインドがモデル状態エラーを追加できるようにします。 次に例を示します。

[!code-csharp[](model-binding/samples/2.x/ModelBindingSample/Models/InstructorWithCollection.cs?name=snippet_BindRequired&highlight=8-9)]

### <a name="bindnever-attribute"></a>[BindNever] 属性

モデルのプロパティにのみに適用でき、メソッドのパラメーターには適用できません。 モデル バインドがモデルのプロパティを設定できないようにします。 次に例を示します。

[!code-csharp[](model-binding/samples/2.x/ModelBindingSample/Models/InstructorWithDictionary.cs?name=snippet_BindNever&highlight=3-4)]

### <a name="bind-attribute"></a>[Bind] 属性

クラスまたはメソッド パラメーターに適用できます。 モデルのどのプロパティをモデル バインドに含めるかを指定します。

次の例では、任意のハンドラーまたはアクション メソッドが呼び出されると、`Instructor` モデルの指定されたプロパティのみがバインドされます。

```csharp
[Bind("LastName,FirstMidName,HireDate")]
public class Instructor
```

次の例では、`OnPost` メソッドが呼び出されると、`Instructor` モデルの指定されたプロパティのみがバインドされます。

```csharp
[HttpPost]
public IActionResult OnPost([Bind("LastName,FirstMidName,HireDate")] Instructor instructor)
```

`[Bind]` 属性を使用すれば、"*作成*" シナリオにおいて過剰ポスティングから保護することができます。 除外されたプロパティはそのままにしておくのではなく null または既定値に設定されるので、この属性は編集シナリオではうまく機能しません。 過剰ポスティングを防ぐ場合は、`[Bind]` 属性ではなくビュー モデルをお勧めします。 詳細については、「[過剰ポスティングに関するセキュリティの注意事項](xref:data/ef-mvc/crud#security-note-about-overposting)」を参照してください。

## <a name="collections"></a>コレクション

ターゲットが単純型のコレクションである場合、モデル バインドでは *parameter_name* または *property_name* との一致が探索されます。 一致が見つからない場合は、サポートされているいずれかの形式がプレフィックスなしで探索されます。 次に例を示します。

* バインドされるパラメーターが `selectedCourses` という名前の配列であるとした場合: 

  ```csharp
  public IActionResult OnPost(int? id, int[] selectedCourses)
  ```

* フォームまたはクエリ文字列データは、次のいずれかの形式とすることができます。
   
  ```
  selectedCourses=1050&selectedCourses=2000 
  ```

  ```
  selectedCourses[0]=1050&selectedCourses[1]=2000
  ```

  ```
  [0]=1050&[1]=2000
  ```

  ```
  selectedCourses[a]=1050&selectedCourses[b]=2000&selectedCourses.index=a&selectedCourses.index=b
  ```

  ```
  [a]=1050&[b]=2000&index=a&index=b
  ```

* 次の形式は、フォーム データでのみサポートされます。

  ```
  selectedCourses[]=1050&selectedCourses[]=2000
  ```

* 上記のすべてのフォーマット例において、モデル バインドでは 2 つの項目から成る配列が `selectedCourses` パラメーターに渡されます。

  * selectedCourses[0]=1050
  * selectedCourses[1]=2000

  添え字番号 (... [0] ... [1] ...) を使用するデータ フォーマットでは、確実にそれらがゼロから始まる連続した番号になるようにする必要があります。 添え字の番号付けで欠落している番号がある場合、欠落している番号の後の項目はすべて無視されます。 たとえば、添え字が 0、1 の並びではなく、0、2 の並びで振られている場合、2 番目の項目は無視されます。

## <a name="dictionaries"></a>辞書

`Dictionary` ターゲットの場合、モデル バインドでは *parameter_name* または *property_name* との一致が探索されます。 一致が見つからない場合は、サポートされているいずれかの形式がプレフィックスなしで探索されます。 次に例を示します。

* ターゲット パラメーターが `selectedCourses` という名前の `Dictionary<int, string>` であるとします: 

  ```csharp
  public IActionResult OnPost(int? id, Dictionary<int, string> selectedCourses)
  ```

* ポストされたフォームまたはクエリ文字列データは、次のいずれかの例のようになります。

  ```
  selectedCourses[1050]=Chemistry&selectedCourses[2000]=Economics
  ```

  ```
  [1050]=Chemistry&selectedCourses[2000]=Economics
  ```

  ```
  selectedCourses[0].Key=1050&selectedCourses[0].Value=Chemistry&
  selectedCourses[1].Key=2000&selectedCourses[1].Value=Economics
  ```

  ```
  [0].Key=1050&[0].Value=Chemistry&[1].Key=2000&[1].Value=Economics
  ```

* 上記のすべてのフォーマット例において、モデル バインドでは 2 つの項目から成る辞書が `selectedCourses` パラメーターに渡されます。

  * selectedCourses["1050"]="Chemistry"
  * selectedCourses["2000"]="Economics"

<a name="glob"></a>

## <a name="globalization-behavior-of-model-binding-route-data-and-query-strings"></a>モデル バインド ルート データとクエリ文字列のグローバリゼーション動作

ASP.NET Core ルート値プロバイダーとクエリ文字列値プロバイダーでは、次のことが行われます。

* 値をインバリアント カルチャとして扱います。
* URL はカルチャに依存しないものと想定します。

これに対し、フォーム データからの値は、カルチャに依存した変換にかけられます。 URL がロケール間で共有可能なように、設計上そのようになっています。

ASP.NET Core ルート値プロバイダーとクエリ文字列値プロバイダーでカルチャ依存の変換が行われるようにするには、次のようにします。

* <xref:Microsoft.AspNetCore.Mvc.ModelBinding.IValueProviderFactory> から継承します
* [QueryStringValueProviderFactory](https://github.com/dotnet/AspNetCore/blob/main/src/Mvc/Mvc.Core/src/ModelBinding/QueryStringValueProviderFactory.cs) または [RouteValueValueProviderFactory](https://github.com/dotnet/AspNetCore/blob/main/src/Mvc/Mvc.Core/src/ModelBinding/RouteValueProviderFactory.cs) からコードをコピーします。
* 値プロバイダー コンストラクターに渡された[カルチャ値](https://github.com/dotnet/AspNetCore/blob/e625fe29b049c60242e8048b4ea743cca65aa7b5/src/Mvc/Mvc.Core/src/ModelBinding/QueryStringValueProviderFactory.cs#L30)を [CultureInfo.CurrentCulture](xref:System.Globalization.CultureInfo.CurrentCulture) に置き換えます。
* MVC オプション内の既定値プロバイダー ファクトリを、新しいものに置き換えます。

[!code-csharp[](model-binding/samples_snapshot/2.x/Startup.cs?name=snippet)]
[!code-csharp[](model-binding/samples_snapshot/2.x/Startup.cs?name=snippet1)]

## <a name="special-data-types"></a>特別なデータ型

モデル バインドで処理できる特殊なデータ型がいくつかあります。

### <a name="iformfile-and-iformfilecollection"></a>IFormFile と IFormFileCollection

HTTP 要求に含まれたアップロード済みファイル。  また、複数のファイルに対して `IEnumerable<IFormFile>` もサポートされています。

### <a name="cancellationtoken"></a>CancellationToken

非同期コントローラーでアクティビティをキャンセルするために使用されます。

### <a name="formcollection"></a>FormCollection

ポストされたフォーム データからすべての値を取得するために使用します。

## <a name="input-formatters"></a>入力フォーマッタ

要求本文内のデータは、JSON、XML、またはその他のいくつかの形式にすることができます。 このデータを解析するために、モデル バインドでは、特定のコンテンツの種類を処理するように構成された "*入力フォーマッタ*" が使用されます。 既定では、ASP.NET Core には JSON データ処理用の JSON ベースの入力フォーマッタが含まれます。 他のコンテンツの種類については対応する他のフォーマッタを追加することができます。

ASP.NET Core では、[Consumes](xref:Microsoft.AspNetCore.Mvc.ConsumesAttribute) 属性に基づいて入力フォーマッタが選択されます。 属性が存在しない場合は、[Content-Type ヘッダー](https://www.w3.org/Protocols/rfc1341/4_Content-Type.html)が使用されます。

組み込みの XML 入力フォーマッタを使用するには: 

* `Microsoft.AspNetCore.Mvc.Formatters.Xml` NuGet パッケージをインストールします。

* `Startup.ConfigureServices` で、<xref:Microsoft.Extensions.DependencyInjection.MvcXmlMvcCoreBuilderExtensions.AddXmlSerializerFormatters*> または <xref:Microsoft.Extensions.DependencyInjection.MvcXmlMvcCoreBuilderExtensions.AddXmlDataContractSerializerFormatters*> を呼び出します。

  [!code-csharp[](model-binding/samples/2.x/ModelBindingSample/Startup.cs?name=snippet_ValueProvider&highlight=9)]

* 要求本文で XML を必要とするコントローラー クラスまたはアクション メソッドに `Consumes` 属性を適用します。

  ```csharp
  [HttpPost]
  [Consumes("application/xml")]
  public ActionResult<Pet> Create(Pet pet)
  ```

  詳細については、「[XML シリアル化の概要](/dotnet/standard/serialization/introducing-xml-serialization)」を参照してください。

## <a name="exclude-specified-types-from-model-binding"></a>指定された型をモデル バインドから除外する

モデル バインドおよび検証システムの動作は、[ModelMetadata](/dotnet/api/microsoft.aspnetcore.mvc.modelbinding.modelmetadata) によって駆動されます。 `ModelMetadata` については、詳細プロバイダーを [MvcOptions.ModelMetadataDetailsProviders](xref:Microsoft.AspNetCore.Mvc.MvcOptions.ModelMetadataDetailsProviders) に追加してカスタマイズできます。 組み込みの詳細プロバイダーは、指定された型に対してモデル バインドまたは検証を無効にする場合に使用できます。

指定された型のすべてのモデルに対してモデル バインドを無効にするには、`Startup.ConfigureServices` に <xref:Microsoft.AspNetCore.Mvc.ModelBinding.Metadata.ExcludeBindingMetadataProvider> を追加します。 たとえば、`System.Version` 型のすべてのモデルに対してモデル バインドを無効にするには、次のようにします。

[!code-csharp[](model-binding/samples/2.x/ModelBindingSample/Startup.cs?name=snippet_ValueProvider&highlight=4-5)]

指定された型のプロパティに対して検証を無効にするには、`Startup.ConfigureServices` に <xref:Microsoft.AspNetCore.Mvc.ModelBinding.SuppressChildValidationMetadataProvider> を追加します。 たとえば、`System.Guid` 型のプロパティに対して検証を無効にするには、次のようにします。

[!code-csharp[](model-binding/samples/2.x/ModelBindingSample/Startup.cs?name=snippet_ValueProvider&highlight=6-7)]

## <a name="custom-model-binders"></a>カスタム モデル バインダー

モデル バインドを拡張するには、カスタム モデル バインダーを記述し、`[ModelBinder]` 属性を使用してそれを特定のターゲット向けに選択します。 詳細については、「[custom model binding](xref:mvc/advanced/custom-model-binding)」 (カスタム モデル バインド) を参照してください。

## <a name="manual-model-binding"></a>手動によるモデル バインド

モデル バインドは、<xref:Microsoft.AspNetCore.Mvc.ControllerBase.TryUpdateModelAsync*> メソッドを使用して手動で呼び出すことができます。 このメソッドは `ControllerBase` クラスと `PageModel` クラスの両方で定義されています。 メソッドのオーバーロードにより、使用するプレフィックスと値プロバイダーを指定できます。 モデル バインドが失敗した場合は、メソッドから `false` が返されます。 次に例を示します。

[!code-csharp[](model-binding/samples/2.x/ModelBindingSample/Pages/InstructorsWithCollection/Create.cshtml.cs?name=snippet_TryUpdate&highlight=1-4)]

## <a name="fromservices-attribute"></a>[FromServices] 属性

この属性の名前は、データ ソースを指定するモデル バインド属性のパターンに従います。 ただし、それは、値プロバイダーからのデータ バインドを説明するものではありません。 [依存関係挿入](xref:fundamentals/dependency-injection)コンテナーから型のインスタンスが取得されます。 その目的は、特定のメソッドが呼び出された場合にのみサービスを必要するときにコンストラクターの挿入の代替手段を提供することにあります。

## <a name="additional-resources"></a>その他のリソース

* <xref:mvc/models/validation>
* <xref:mvc/advanced/custom-model-binding>

::: moniker-end
