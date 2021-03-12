---
title: ASP.NET Core でのカスタム モデル バインド
author: ardalis
description: モデル バインドにより ASP.NET Core のモデルの型を使用して、コントローラー アクションが直接動作する方法について説明します。
ms.author: riande
ms.date: 01/06/2020
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
uid: mvc/advanced/custom-model-binding
ms.openlocfilehash: b674d3a034124a26e2e76f2a7f0220e3f0ecea56
ms.sourcegitcommit: 54fe1ae5e7d068e27376d562183ef9ddc7afc432
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/10/2021
ms.locfileid: "102588771"
---
# <a name="custom-model-binding-in-aspnet-core"></a>ASP.NET Core でのカスタム モデル バインド

::: moniker range=">= aspnetcore-3.0"

作成者: [Steve Smith](https://ardalis.com/)、[Kirk Larkin](https://twitter.com/serpent5)

モデル バインドにより、コントローラー アクションが HTTP 要求ではなく (メソッド引数として渡される) モデルの型を直接操作できるようになります。 受信要求データとアプリケーション モデルのマッピングは、モデル バインダーによって処理されます。 開発者は、カスタム モデル バインダーを実装することで組み込みのモデル バインド機能を拡張することができます (ただし、通常は自分のプロバイダーを記述する必要はありません)。

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/mvc/advanced/custom-model-binding/samples)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

## <a name="default-model-binder-limitations"></a>既定のモデル バインダーの制限事項

既定のモデル バインダーは、一般的な .NET Core データ型の多くをサポートし、ほとんどの開発者のニーズを満たします。 要求からのテキストベースの入力をモデルの型に直接バインドすることが見込まれています。 入力は、バインドする前に変換しなければならないことがあります。 たとえば、モデル データの検索に使用できるキーがある場合などです。 カスタム モデル バインダーを使用すると、キーに基づいてデータをフェッチすることができます。

## <a name="model-binding-review"></a>モデル バインドの確認

モデル バインドは、操作の対象とする型に特定の定義を使用します。 *単純型* は、入力の 1 つの文字列から変換されます。 *複合型* は、複数の入力値から変換されます。 フレームワークは、`TypeConverter` の存在の有無によって違いを判断します。 外部リソースを必要としない `string` -> `SomeType` の単純なマッピングがある場合は型コンバーターを作成することをお勧めします。

独自のカスタム モデル バインダーを作成する前に、既存のモデル バインダーがどのように実装されているかを確認するとよいでしょう。 Base64 でエンコードされた文字列をバイト配列に変換できる、<xref:Microsoft.AspNetCore.Mvc.ModelBinding.Binders.ByteArrayModelBinder> の使用を検討してください。 バイト配列は多くの場合、ファイルまたはデータベース BLOB のフィールドとして格納されます。

### <a name="working-with-the-bytearraymodelbinder"></a>ByteArrayModelBinder の操作

Base64 でエンコードされた文字列を使用してバイナリ データを表すことができます。 たとえば、イメージは文字列としてエンコードできます。 このサンプルには、[Base64String.txt](https://github.com/dotnet/AspNetCore.Docs/blob/main/aspnetcore/mvc/advanced/custom-model-binding/samples/3.x/CustomModelBindingSample/Base64String.txt) 内に Base64 でエンコードされた文字列としてのイメージが含まれています。

ASP.NET Core MVC では Base64 でエンコードされた文字列を取得し、`ByteArrayModelBinder` を使用してこれをバイト配列に変換できます。 <xref:Microsoft.AspNetCore.Mvc.ModelBinding.Binders.ByteArrayModelBinderProvider> により `byte[]` 引数が `ByteArrayModelBinder` にマップされます。

```csharp
public IModelBinder GetBinder(ModelBinderProviderContext context)
{
    if (context == null)
    {
        throw new ArgumentNullException(nameof(context));
    }

    if (context.Metadata.ModelType == typeof(byte[]))
    {
        var loggerFactory = context.Services.GetRequiredService<ILoggerFactory>();
        return new ByteArrayModelBinder(loggerFactory);
    }

    return null;
}
```

独自のカスタム モデル バインダーを作成するときには、独自の `IModelBinderProvider` 型を実装するか、<xref:Microsoft.AspNetCore.Mvc.ModelBinderAttribute> を使用することができます。

次の例は、`ByteArrayModelBinder` を使用して Base64 でエンコードされた文字列を `byte[]` に変換し、結果をファイルに保存する方法を示しています。

[!code-csharp[](custom-model-binding/samples/3.x/CustomModelBindingSample/Controllers/ImageController.cs?name=snippet_Post)]
[!INCLUDE[about the series](~/includes/code-comments-loc.md)]

[Postman](https://www.getpostman.com/) のようなツールを使用すると、この API メソッドに Base64 でエンコードされた文字列を POST することができます。

![postman](custom-model-binding/images/postman.png "postman")

バインダーが要求データを適切に命名されたプロパティまたは引数にバインドできれば、モデル バインドは成功します。 ビュー モデルを指定して `ByteArrayModelBinder` を使用する方法を次の例に示します。

[!code-csharp[](custom-model-binding/samples/3.x/CustomModelBindingSample/Controllers/ImageController.cs?name=snippet_SaveProfile&highlight=2)]

## <a name="custom-model-binder-sample"></a>カスタム モデル バインダーのサンプル

このセクションでは、次のようなカスタム モデル バインダーを実装します。

- 受信した要求データを厳密に型指定されたキーの引数に変換する。
- Entity Framework Core を使用して関連するエンティティをフェッチする。
- 関連するエンティティを引数としてアクション メソッドに渡す。

次のサンプルでは、`Author` モデルの `ModelBinder` 属性を使用しています。

[!code-csharp[](custom-model-binding/samples/3.x/CustomModelBindingSample/Data/Author.cs?highlight=6)]

上記のコードでは、`Author` アクション パラメーターのバインドで使用される必要がある `IModelBinder` の型が `ModelBinder` 属性で指定されています。

次の `AuthorEntityBinder` クラスは、Entity Framework Core と `authorId` を使用してデータ ソースからエンティティをフェッチすることで `Author` パラメーターをバインドします。

[!code-csharp[](custom-model-binding/samples/3.x/CustomModelBindingSample/Binders/AuthorEntityBinder.cs?name=snippet_Class)]

> [!NOTE]
> 前述の `AuthorEntityBinder` クラスは、カスタム モデル バインダーを示しています。 このクラスは、参照シナリオのベスト プラクティスを示すものではありません。 参照の場合は、`authorId` をバインドし、アクション メソッドでデータベースをクエリします。 この方法により、`NotFound` のケースからモデル バインドのエラーが分離されます。

アクション メソッドでの `AuthorEntityBinder` の使用方法を次のコードに示します。

[!code-csharp[](custom-model-binding/samples/3.x/CustomModelBindingSample/Controllers/BoundAuthorsController.cs?name=snippet_Get&highlight=2)]

`ModelBinder` 属性を使用すると、既定の規則を使用しないパラメーターに `AuthorEntityBinder` を適用できます。

[!code-csharp[](custom-model-binding/samples/3.x/CustomModelBindingSample/Controllers/BoundAuthorsController.cs?name=snippet_GetById&highlight=2)]

この例では、引数の名前が既定の `authorId` ではないため、`ModelBinder` 属性を使用してパラメーターに指定されています。 アクション メソッドでエンティティを検索する場合と比較して、コントローラーとアクション メソッドの両方が簡略化されています。 Entity Framework Core を使用して作成者をフェッチするためのロジックは、モデル バインダーに移動しています。 これは、`Author` モデルにバインドするメソッドがいくつかある場合、大幅な簡略化になる可能性があります。

`ModelBinder` 属性を (ビューモデルなどの) 個々のモデル プロパティまたはアクション メソッド パラメーターに適用して、その型やアクションのみを対象とする特定のモデル バインダーまたはモデル名を指定することができます。

### <a name="implementing-a-modelbinderprovider"></a>ModelBinderProvider の実装

属性を適用する代わりに、`IModelBinderProvider` を実装することができます。 これは、組み込みフレームワーク バインダーの実装方法と同じです。 バインダーが動作する型を指定するときに、バインダーが受け入れる入力 **ではなく**、生成される引数の型を指定します。 次のバインダー プロバイダーは `AuthorEntityBinder` で動作します。 プロバイダーの MVC のコレクションに追加されるときに、`Author` または `Author` の型のパラメーターで `ModelBinder` 属性を使用する必要はありません。

[!code-csharp[](custom-model-binding/samples/3.x/CustomModelBindingSample/Binders/AuthorEntityBinderProvider.cs?highlight=17-20)]

> 注: 上のコードは `BinderTypeModelBinder` を返します。 `BinderTypeModelBinder` はモデル バインダーのファクトリとして機能し、依存関係の挿入 (DI) を提供します。 `AuthorEntityBinder` は DI に EF Core へのアクセスを求めます。 モデル バインダーが DI からのサービスを必要としている場合は、`BinderTypeModelBinder` を使用してください。

カスタム モデル バインダー プロバイダーを使用する場合、次のようにしてこれを `ConfigureServices` に追加します。

[!code-csharp[](custom-model-binding/samples/3.x/CustomModelBindingSample/Startup.cs?name=snippet_ConfigureServices&highlight=5-8)]

モデル バインダーを評価するときに、プロバイダーのコレクションが順序どおりにチェックされます。 入力モデルに一致するバインダーを返す最初のプロバイダーが使用されます。 プロバイダーをコレクションの末尾に追加すると、カスタムバインダーが機能する前に、組み込みのモデルバインダーが呼び出される可能性があります。 この例では、カスタムプロバイダーがコレクションの先頭に追加され、常にアクション引数に使用されるようになってい `Author` ます。

### <a name="polymorphic-model-binding"></a>ポリモーフィック モデル バインド

派生型の異なるモデルへのバインドは、ポリモーフィック モデル バインドと呼ばれます。 ポリモーフィック カスタム モデル バインドは、要求値を特定の派生モデル型にバインドする必要がある場合に必要です。 ポリモーフィック モデル バインド:

* すべての言語と相互運用できるように設計された REST API では一般的ではありません。
* バインドされたモデルに関する判断が困難になります。

ただし、アプリにポリモーフィック モデル バインドが必要な場合、実装は次のコードのようになります。

[!code-csharp[](custom-model-binding/samples/3.x/PolymorphicModelBindingSample/ModelBinders/PolymorphicModelBinder.cs?name=snippet)]

## <a name="recommendations-and-best-practices"></a>推奨事項とベスト プラクティス

カスタム モデル バインダー:

- 状態コードの設定または結果 (たとえば 404 Not Found) のリターンを試行しないでください。 モデル バインドが失敗した場合、アクション メソッド自体の[アクション フィルター](xref:mvc/controllers/filters)またはロジックでエラーが処理される必要があります。
- アクション メソッドから繰り返しのコードや横断的な問題を排除する場合に最も役立ちます。
- 通常、文字列をカスタムの型に変換する場合に使用すべきではありません。通常は、<xref:System.ComponentModel.TypeConverter> の方がオプションとして優れています。

::: moniker-end
::: moniker range="< aspnetcore-3.0"

作成者: [Steve Smith](https://ardalis.com/)

モデル バインドにより、コントローラー アクションが HTTP 要求ではなく (メソッド引数として渡される) モデルの型を直接操作できるようになります。 受信要求データとアプリケーション モデルのマッピングは、モデル バインダーによって処理されます。 開発者は、カスタム モデル バインダーを実装することで組み込みのモデル バインド機能を拡張することができます (ただし、通常は自分のプロバイダーを記述する必要はありません)。

[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/mvc/advanced/custom-model-binding/samples)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

## <a name="default-model-binder-limitations"></a>既定のモデル バインダーの制限事項

既定のモデル バインダーは、一般的な .NET Core データ型の多くをサポートし、ほとんどの開発者のニーズを満たします。 要求からのテキストベースの入力をモデルの型に直接バインドすることが見込まれています。 入力は、バインドする前に変換しなければならないことがあります。 たとえば、モデル データの検索に使用できるキーがある場合などです。 カスタム モデル バインダーを使用すると、キーに基づいてデータをフェッチすることができます。

## <a name="model-binding-review"></a>モデル バインドの確認

モデル バインドは、操作の対象とする型に特定の定義を使用します。 *単純型* は、入力の 1 つの文字列から変換されます。 *複合型* は、複数の入力値から変換されます。 フレームワークは、`TypeConverter` の存在の有無によって違いを判断します。 外部リソースを必要としない `string` -> `SomeType` の単純なマッピングがある場合は型コンバーターを作成することをお勧めします。

独自のカスタム モデル バインダーを作成する前に、既存のモデル バインダーがどのように実装されているかを確認するとよいでしょう。 Base64 でエンコードされた文字列をバイト配列に変換できる、<xref:Microsoft.AspNetCore.Mvc.ModelBinding.Binders.ByteArrayModelBinder> の使用を検討してください。 バイト配列は多くの場合、ファイルまたはデータベース BLOB のフィールドとして格納されます。

### <a name="working-with-the-bytearraymodelbinder"></a>ByteArrayModelBinder の操作

Base64 でエンコードされた文字列を使用してバイナリ データを表すことができます。 たとえば、イメージは文字列としてエンコードできます。 このサンプルには、[Base64String.txt](https://github.com/dotnet/AspNetCore.Docs/blob/main/aspnetcore/mvc/advanced/custom-model-binding/samples/2.x/CustomModelBindingSample/Base64String.txt) 内に Base64 でエンコードされた文字列としてのイメージが含まれています。

ASP.NET Core MVC では Base64 でエンコードされた文字列を取得し、`ByteArrayModelBinder` を使用してこれをバイト配列に変換できます。 <xref:Microsoft.AspNetCore.Mvc.ModelBinding.Binders.ByteArrayModelBinderProvider> により `byte[]` 引数が `ByteArrayModelBinder` にマップされます。

```csharp
public IModelBinder GetBinder(ModelBinderProviderContext context)
{
    if (context == null)
    {
        throw new ArgumentNullException(nameof(context));
    }

    if (context.Metadata.ModelType == typeof(byte[]))
    {
        return new ByteArrayModelBinder();
    }

    return null;
}
```

独自のカスタム モデル バインダーを作成するときには、独自の `IModelBinderProvider` 型を実装するか、<xref:Microsoft.AspNetCore.Mvc.ModelBinderAttribute> を使用することができます。

次の例は、`ByteArrayModelBinder` を使用して Base64 でエンコードされた文字列を `byte[]` に変換し、結果をファイルに保存する方法を示しています。

[!code-csharp[](custom-model-binding/samples/2.x/CustomModelBindingSample/Controllers/ImageController.cs?name=post1)]

[Postman](https://www.getpostman.com/) のようなツールを使用すると、この API メソッドに Base64 でエンコードされた文字列を POST することができます。

![postman](custom-model-binding/images/postman.png "postman")

バインダーが要求データを適切に命名されたプロパティまたは引数にバインドできれば、モデル バインドは成功します。 ビュー モデルを指定して `ByteArrayModelBinder` を使用する方法を次の例に示します。

[!code-csharp[](custom-model-binding/samples/2.x/CustomModelBindingSample/Controllers/ImageController.cs?name=post2&highlight=2)]

## <a name="custom-model-binder-sample"></a>カスタム モデル バインダーのサンプル

このセクションでは、次のようなカスタム モデル バインダーを実装します。

- 受信した要求データを厳密に型指定されたキーの引数に変換する。
- Entity Framework Core を使用して関連するエンティティをフェッチする。
- 関連するエンティティを引数としてアクション メソッドに渡す。

次のサンプルでは、`Author` モデルの `ModelBinder` 属性を使用しています。

[!code-csharp[](custom-model-binding/samples/2.x/CustomModelBindingSample/Data/Author.cs?highlight=6)]

上記のコードでは、`Author` アクション パラメーターのバインドで使用される必要がある `IModelBinder` の型が `ModelBinder` 属性で指定されています。

次の `AuthorEntityBinder` クラスは、Entity Framework Core と `authorId` を使用してデータ ソースからエンティティをフェッチすることで `Author` パラメーターをバインドします。

[!code-csharp[](custom-model-binding/samples/2.x/CustomModelBindingSample/Binders/AuthorEntityBinder.cs?name=demo)]

> [!NOTE]
> 前述の `AuthorEntityBinder` クラスは、カスタム モデル バインダーを示しています。 このクラスは、参照シナリオのベスト プラクティスを示すものではありません。 参照の場合は、`authorId` をバインドし、アクション メソッドでデータベースをクエリします。 この方法により、`NotFound` のケースからモデル バインドのエラーが分離されます。

アクション メソッドでの `AuthorEntityBinder` の使用方法を次のコードに示します。

[!code-csharp[](custom-model-binding/samples/2.x/CustomModelBindingSample/Controllers/BoundAuthorsController.cs?name=demo2&highlight=2)]

`ModelBinder` 属性を使用すると、既定の規則を使用しないパラメーターに `AuthorEntityBinder` を適用できます。

[!code-csharp[](custom-model-binding/samples/2.x/CustomModelBindingSample/Controllers/BoundAuthorsController.cs?name=demo1&highlight=2)]

この例では、引数の名前が既定の `authorId` ではないため、`ModelBinder` 属性を使用してパラメーターに指定されています。 アクション メソッドでエンティティを検索する場合と比較して、コントローラーとアクション メソッドの両方が簡略化されています。 Entity Framework Core を使用して作成者をフェッチするためのロジックは、モデル バインダーに移動しています。 これは、`Author` モデルにバインドするメソッドがいくつかある場合、大幅な簡略化になる可能性があります。

`ModelBinder` 属性を (ビューモデルなどの) 個々のモデル プロパティまたはアクション メソッド パラメーターに適用して、その型やアクションのみを対象とする特定のモデル バインダーまたはモデル名を指定することができます。

### <a name="implementing-a-modelbinderprovider"></a>ModelBinderProvider の実装

属性を適用する代わりに、`IModelBinderProvider` を実装することができます。 これは、組み込みフレームワーク バインダーの実装方法と同じです。 バインダーが動作する型を指定するときに、バインダーが受け入れる入力 **ではなく**、生成される引数の型を指定します。 次のバインダー プロバイダーは `AuthorEntityBinder` で動作します。 プロバイダーの MVC のコレクションに追加されるときに、`Author` または `Author` の型のパラメーターで `ModelBinder` 属性を使用する必要はありません。

[!code-csharp[](custom-model-binding/samples/2.x/CustomModelBindingSample/Binders/AuthorEntityBinderProvider.cs?highlight=17-20)]

> 注: 上のコードは `BinderTypeModelBinder` を返します。 `BinderTypeModelBinder` はモデル バインダーのファクトリとして機能し、依存関係の挿入 (DI) を提供します。 `AuthorEntityBinder` は DI に EF Core へのアクセスを求めます。 モデル バインダーが DI からのサービスを必要としている場合は、`BinderTypeModelBinder` を使用してください。

カスタム モデル バインダー プロバイダーを使用する場合、次のようにしてこれを `ConfigureServices` に追加します。

[!code-csharp[](custom-model-binding/samples/2.x/CustomModelBindingSample/Startup.cs?name=snippet_ConfigureServices&highlight=5-10)]

モデル バインダーを評価するときに、プロバイダーのコレクションが順序どおりにチェックされます。 バインダーを返す最初のプロバイダーが使用されます。 コレクションの末尾にプロバイダーを追加すると、カスタム バインダーより前に組み込みのモデル バインダーが呼び出される可能性があります。 この例では、カスタム プロバイダーが `Author` アクションの引数で使用されるように、コレクションの先頭に追加されています。

### <a name="polymorphic-model-binding"></a>ポリモーフィック モデル バインド

派生型の異なるモデルへのバインドは、ポリモーフィック モデル バインドと呼ばれます。 ポリモーフィック カスタム モデル バインドは、要求値を特定の派生モデル型にバインドする必要がある場合に必要です。 ポリモーフィック モデル バインド:

* すべての言語と相互運用できるように設計された REST API では一般的ではありません。
* バインドされたモデルに関する判断が困難になります。

ただし、アプリにポリモーフィック モデル バインドが必要な場合、実装は次のコードのようになります。

[!code-csharp[](custom-model-binding/samples/3.x/PolymorphicModelBindingSample/ModelBinders/PolymorphicModelBinder.cs?name=snippet)]

## <a name="recommendations-and-best-practices"></a>推奨事項とベスト プラクティス

カスタム モデル バインダー:

- 状態コードの設定または結果 (たとえば 404 Not Found) のリターンを試行しないでください。 モデル バインドが失敗した場合、アクション メソッド自体の[アクション フィルター](xref:mvc/controllers/filters)またはロジックでエラーが処理される必要があります。
- アクション メソッドから繰り返しのコードや横断的な問題を排除する場合に最も役立ちます。
- 通常、文字列をカスタムの型に変換する場合に使用すべきではありません。通常は、<xref:System.ComponentModel.TypeConverter> の方がオプションとして優れています。

::: moniker-end
