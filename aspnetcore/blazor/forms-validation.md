---
title: ASP.NET Core Blazor のフォームと検証
author: guardrex
description: Blazor でフォームとフィールドの検証シナリオを使用する方法について説明します。
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 09/17/2020
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
uid: blazor/forms-validation
ms.openlocfilehash: 1287ab5ce61e58848329c96393c3ee8c37610245
ms.sourcegitcommit: cc405f20537484744423ddaf87bd1e7d82b6bdf0
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/21/2021
ms.locfileid: "98658691"
---
# <a name="aspnet-core-no-locblazor-forms-and-validation"></a>ASP.NET Core Blazor のフォームと検証

作成者: [Daniel Roth](https://github.com/danroth27)、[Rémi Bourgarel](https://remibou.github.io/)、[Luke Latham](https://github.com/guardrex)

フォームと検証は、Blazor で[データ注釈](xref:mvc/models/validation)を使用してサポートされています。

次の `ExampleModel` 型は、データ注釈を使用して検証ロジックを定義します。

```csharp
using System.ComponentModel.DataAnnotations;

public class ExampleModel
{
    [Required]
    [StringLength(10, ErrorMessage = "Name is too long.")]
    public string Name { get; set; }
}
```

フォームは、<xref:Microsoft.AspNetCore.Components.Forms.EditForm> コンポーネントを使用して定義されます。 次のフォームは、一般的な要素、コンポーネント、および Razor コードを示しています。

```razor
<EditForm Model="@exampleModel" OnValidSubmit="@HandleValidSubmit">
    <DataAnnotationsValidator />
    <ValidationSummary />

    <InputText id="name" @bind-Value="exampleModel.Name" />

    <button type="submit">Submit</button>
</EditForm>

@code {
    private ExampleModel exampleModel = new ExampleModel();

    private void HandleValidSubmit()
    {
        ...
    }
}
```

前の例の場合:

* フォームは、`ExampleModel` 型で定義されている検証を使用して、`name` フィールドのユーザー入力を検証します。 モデルはコンポーネントの `@code` ブロック内に作成され、プライベート フィールド (`exampleModel`) に保持されます。 フィールドは、`<EditForm>` 要素の `Model` 属性に割り当てられます。
* <xref:Microsoft.AspNetCore.Components.Forms.InputText> コンポーネントの `@bind-Value` は次のようにバインドします。
  * モデル プロパティ (`exampleModel.Name`) を <xref:Microsoft.AspNetCore.Components.Forms.InputText> コンポーネントの `Value` プロパティへ。 プロパティのバインドの詳細については、<xref:blazor/components/data-binding#binding-with-component-parameters> を参照してください。
  * 変更イベント デリゲートを <xref:Microsoft.AspNetCore.Components.Forms.InputText> コンポーネントの `ValueChanged` プロパティへ。
* <xref:Microsoft.AspNetCore.Components.Forms.DataAnnotationsValidator> [検証コンポーネント](#validator-components)は、データ注釈を使用して検証サポートをアタッチします。
* <xref:Microsoft.AspNetCore.Components.Forms.ValidationSummary> コンポーネントは、検証メッセージの概要を示します。
* `HandleValidSubmit` は、フォームが正常に送信される (検証に合格する) とトリガーされます。

## <a name="built-in-forms-components"></a>組み込みフォームのコンポーネント

一連の組み込みコンポーネントを、ユーザー入力の受信と検証に使用できます。 入力は、それらが変更されたときとフォームが送信されたときに検証されます。 次の表に、使用できる入力コンポーネントを示しています。

::: moniker range=">= aspnetcore-5.0"

| 入力コンポーネント | &hellip; とレンダリング |
| --------------- | ------------------- |
| <xref:Microsoft.AspNetCore.Components.Forms.InputCheckbox> | `<input type="checkbox">` |
| <xref:Microsoft.AspNetCore.Components.Forms.InputDate%601> | `<input type="date">` |
| [`InputFile`](xref:blazor/file-uploads) | `<input type="file">` |
| <xref:Microsoft.AspNetCore.Components.Forms.InputNumber%601> | `<input type="number">` |
| [`InputRadio`](#radio-buttons) | `<input type="radio">` |
| [`InputRadioGroup`](#radio-buttons) | `<input type="radio">` |
| <xref:Microsoft.AspNetCore.Components.Forms.InputSelect%601> | `<select>` |
| <xref:Microsoft.AspNetCore.Components.Forms.InputText> | `<input>` |
| <xref:Microsoft.AspNetCore.Components.Forms.InputTextArea> | `<textarea>` |

::: moniker-end

::: moniker range="< aspnetcore-5.0"

| 入力コンポーネント | &hellip; とレンダリング |
| --------------- | ------------------- |
| <xref:Microsoft.AspNetCore.Components.Forms.InputCheckbox> | `<input type="checkbox">` |
| <xref:Microsoft.AspNetCore.Components.Forms.InputDate%601> | `<input type="date">` |
| <xref:Microsoft.AspNetCore.Components.Forms.InputNumber%601> | `<input type="number">` |
| <xref:Microsoft.AspNetCore.Components.Forms.InputSelect%601> | `<select>` |
| <xref:Microsoft.AspNetCore.Components.Forms.InputText> | `<input>` |
| <xref:Microsoft.AspNetCore.Components.Forms.InputTextArea> | `<textarea>` |

> [!NOTE]
> `InputRadio` と `InputRadioGroup` は、ASP.NET Core 5.0 以降で使用できます。 詳細については、この記事のバージョン 5.0 以降を選択してください。

::: moniker-end

すべての入力コンポーネント (<xref:Microsoft.AspNetCore.Components.Forms.EditForm> を含む) で、任意の属性がサポートされています。 コンポーネント パラメーターに一致しない属性は、レンダリングされる HTML 要素に追加されます。

入力コンポーネントにより、フィールド変更時に検証するための既定の動作が提供されます。これには、フィールドの状態を反映するようにフィールドの CSS クラスを更新することが含まれます。 一部のコンポーネントには、便利な解析ロジックが含まれます。 たとえば、<xref:Microsoft.AspNetCore.Components.Forms.InputDate%601> と <xref:Microsoft.AspNetCore.Components.Forms.InputNumber%601> では、解析不能な値を検証エラーとして登録することによって、解析不能な値を適切に処理します。 NULL 値を受け入れることができる型では、ターゲット フィールドの NULL 値の許容もサポートしています (`int?` など)。

次の `Starship` 型では、以前の `ExampleModel` よりも大きなプロパティとデータ注釈のセットを使用して検証ロジックを定義しています。

```csharp
using System;
using System.ComponentModel.DataAnnotations;

public class Starship
{
    [Required]
    [StringLength(16, ErrorMessage = "Identifier too long (16 character limit).")]
    public string Identifier { get; set; }

    public string Description { get; set; }

    [Required]
    public string Classification { get; set; }

    [Range(1, 100000, ErrorMessage = "Accommodation invalid (1-100000).")]
    public int MaximumAccommodation { get; set; }

    [Required]
    [Range(typeof(bool), "true", "true", 
        ErrorMessage = "This form disallows unapproved ships.")]
    public bool IsValidatedDesign { get; set; }

    [Required]
    public DateTime ProductionDate { get; set; }
}
```

前の例では、データ注釈が存在しないため、`Description` は省略可能です。

次のフォームでは、`Starship` モデルで定義されている検証を使用してユーザー入力を検証します。

```razor
@page "/FormsValidation"

<h1>Starfleet Starship Database</h1>

<h2>New Ship Entry Form</h2>

<EditForm Model="@starship" OnValidSubmit="@HandleValidSubmit">
    <DataAnnotationsValidator />
    <ValidationSummary />

    <p>
        <label>
            Identifier:
            <InputText @bind-Value="starship.Identifier" />
        </label>
    </p>
    <p>
        <label>
            Description (optional):
            <InputTextArea @bind-Value="starship.Description" />
        </label>
    </p>
    <p>
        <label>
            Primary Classification:
            <InputSelect @bind-Value="starship.Classification">
                <option value="">Select classification ...</option>
                <option value="Exploration">Exploration</option>
                <option value="Diplomacy">Diplomacy</option>
                <option value="Defense">Defense</option>
            </InputSelect>
        </label>
    </p>
    <p>
        <label>
            Maximum Accommodation:
            <InputNumber @bind-Value="starship.MaximumAccommodation" />
        </label>
    </p>
    <p>
        <label>
            Engineering Approval:
            <InputCheckbox @bind-Value="starship.IsValidatedDesign" />
        </label>
    </p>
    <p>
        <label>
            Production Date:
            <InputDate @bind-Value="starship.ProductionDate" />
        </label>
    </p>

    <button type="submit">Submit</button>

    <p>
        <a href="http://www.startrek.com/">Star Trek</a>, 
        &copy;1966-2019 CBS Studios, Inc. and 
        <a href="https://www.paramount.com">Paramount Pictures</a>
    </p>
</EditForm>

@code {
    private Starship starship = new Starship() { ProductionDate = DateTime.UtcNow };

    private void HandleValidSubmit()
    {
        ...
    }
}
```

<xref:Microsoft.AspNetCore.Components.Forms.EditForm> では、変更されたフィールドと現在の検証メッセージを含む、編集プロセスに関するメタデータを追跡する[カスケード値](xref:blazor/components/cascading-values-and-parameters) として <xref:Microsoft.AspNetCore.Components.Forms.EditContext> を作成します。

<xref:Microsoft.AspNetCore.Components.Forms.EditContext> **または** <xref:Microsoft.AspNetCore.Components.Forms.EditForm.Model?displayProperty=nameWithType> の **いずれか** を <xref:Microsoft.AspNetCore.Components.Forms.EditForm> に割り当てます。 両方とも割り当てることはサポートされておらず、**ランタイム エラー** が生成されます。

<xref:Microsoft.AspNetCore.Components.Forms.EditForm> では、有効および無効のフォームを送信するための便利なイベントが提供されます。

* <xref:Microsoft.AspNetCore.Components.Forms.EditForm.OnValidSubmit>
* <xref:Microsoft.AspNetCore.Components.Forms.EditForm.OnInvalidSubmit>

カスタム コードを使用して検証をトリガーし、フィールドの値をチェックするには、<xref:Microsoft.AspNetCore.Components.Forms.EditForm.OnSubmit> を使用します。

次に例を示します。

* **`Submit`** ボタンが選択されると、`HandleSubmit` メソッドが実行されます。
* <xref:Microsoft.AspNetCore.Components.Forms.EditContext.Validate%2A?displayProperty=nameWithType> を呼び出して、フォームを検証します。
* 検証結果に応じて、追加のコードが実行されます。 <xref:Microsoft.AspNetCore.Components.Forms.EditForm.OnSubmit> に割り当てられたメソッドにビジネス ロジックを配置します。

```razor
<EditForm EditContext="@editContext" OnSubmit="@HandleSubmit">

    ...

    <button type="submit">Submit</button>
</EditForm>

@code {
    private Starship starship = new Starship() { ProductionDate = DateTime.UtcNow };
    private EditContext editContext;

    protected override void OnInitialized()
    {
        editContext = new EditContext(starship);
    }

    private async Task HandleSubmit()
    {
        var isValid = editContext.Validate();

        if (isValid)
        {
            ...
        }
        else
        {
            ...
        }
    }
}
```

> [!NOTE]
> 検証メッセージを <xref:Microsoft.AspNetCore.Components.Forms.EditContext> から直接クリアする Framework API は存在しません。 このため、通常、フォームの新しい <xref:Microsoft.AspNetCore.Components.Forms.ValidationMessageStore> に検証メッセージを追加することはお勧めしません。 検証メッセージを管理するには、この記事で説明するように、[ビジネス ロジック検証コード](#business-logic-validation)で[検証コンポーネント](#validator-components)を使用します。

::: moniker range=">= aspnetcore-5.0"

## <a name="display-name-support"></a>表示名のサポート

"*このセクションは、.NET 5 リリース候補 1 (RC1) 以降の ASP.NET Core に適用されます。* "

次の組み込みコンポーネントでは、`DisplayName` パラメーターを使用した表示名がサポートされています。

* <xref:Microsoft.AspNetCore.Components.Forms.InputDate%601>
* <xref:Microsoft.AspNetCore.Components.Forms.InputNumber%601>
* <xref:Microsoft.AspNetCore.Components.Forms.InputSelect%601>

`InputDate` コンポーネントの例を次に示します。

* 表示名 (`DisplayName`) が `birthday` に設定されています。
* コンポーネントは、`DateTime` 型として `BirthDate` プロパティにバインドされています。

```razor
<InputDate @bind-Value="BirthDate" DisplayName="birthday" />

@code {
    public DateTime BirthDate { get; set; }
}
```

ユーザーが日付の値を指定しなかった場合、次のように検証エラーが表示されます。

```
The birthday must be a date.
```

::: moniker-end

## <a name="validator-components"></a>検証コンポーネント

検証コンポーネントでは、フォームの <xref:Microsoft.AspNetCore.Components.Forms.EditContext>の <xref:Microsoft.AspNetCore.Components.Forms.ValidationMessageStore> を管理することで、フォームの検証をサポートします。

Blazor フレームワークでは、[検証属性 (データ注釈)](xref:mvc/models/validation#validation-attributes)に基づいて検証サポートをフォームにアタッチする <xref:Microsoft.AspNetCore.Components.Forms.DataAnnotationsValidator> コンポーネントを提供します。 カスタム検証コンポーネントを作成して、同じページの異なるフォームの検証メッセージを処理するか、フォーム処理の異なるステップ (たとえば、クライアント側の検証の後のサーバー側の検証) で同じフォームの検証メッセージを処理します。 このセクションで示す検証コンポーネントの例 (`CustomValidator`) は、この記事の次のセクションで使用します。

* [ビジネス ロジックの検証](#business-logic-validation)
* [サーバーの検証](#server-validation)

> [!NOTE]
> 多くの場合、カスタムのデータ注釈検証属性をカスタム検証コンポーネントの代わりに使用できます。 フォームのモデルに適用されるカスタム属性は、<xref:Microsoft.AspNetCore.Components.Forms.DataAnnotationsValidator> コンポーネントを使用してアクティブ化されます。 サーバー側の検証で使用する場合、モデルに適用されるカスタム属性はすべてサーバー上で実行可能である必要があります。 詳細については、「<xref:mvc/models/validation#alternatives-to-built-in-attributes>」を参照してください。

<xref:Microsoft.AspNetCore.Components.ComponentBase>から検証コンポーネントを作成します。

* フォームの <xref:Microsoft.AspNetCore.Components.Forms.EditContext> は、コンポーネントの[カスケード パラメーターです](xref:blazor/components/cascading-values-and-parameters)。
* 検証コンポーネントが初期化されると、フォーム エラーの現在の一覧を保持するために新しい <xref:Microsoft.AspNetCore.Components.Forms.ValidationMessageStore> が作成されます。
* フォームのコンポーネント内の開発者コードで `DisplayErrors` メソッドが呼び出されると、メッセージ ストアでエラーが受け取られます。 そのエラーは、[`Dictionary<string, List<string>>`](xref:System.Collections.Generic.Dictionary`2)内の `DisplayErrors` メソッドに渡されます。 ディクショナリでは、キーは、1 つ以上のエラーがあるフォーム フィールドの名前です。 値は、エラー一覧です。
* 次のいずれかが発生すると、メッセージはクリアされます。
  * <xref:Microsoft.AspNetCore.Components.Forms.EditContext> で検証が要求され、<xref:Microsoft.AspNetCore.Components.Forms.EditContext.OnValidationRequested> イベントが発生した場合。 すべてのエラーがクリアされます。
  * フォームのフィールドが変更され、<xref:Microsoft.AspNetCore.Components.Forms.EditContext.OnFieldChanged> イベントが発生した場合。 そのフィールドのエラーのみがクリアされます。
  * 開発者コードによって `ClearErrors` メソッドが呼び出された場合。 すべてのエラーがクリアされます。

```csharp
using System;
using System.Collections.Generic;
using Microsoft.AspNetCore.Components;
using Microsoft.AspNetCore.Components.Forms;

public class CustomValidator : ComponentBase
{
    private ValidationMessageStore messageStore;

    [CascadingParameter]
    private EditContext CurrentEditContext { get; set; }

    protected override void OnInitialized()
    {
        if (CurrentEditContext == null)
        {
            throw new InvalidOperationException(
                $"{nameof(CustomValidator)} requires a cascading " +
                $"parameter of type {nameof(EditContext)}. " +
                $"For example, you can use {nameof(CustomValidator)} " +
                $"inside an {nameof(EditForm)}.");
        }

        messageStore = new ValidationMessageStore(CurrentEditContext);

        CurrentEditContext.OnValidationRequested += (s, e) => 
            messageStore.Clear();
        CurrentEditContext.OnFieldChanged += (s, e) => 
            messageStore.Clear(e.FieldIdentifier);
    }

    public void DisplayErrors(Dictionary<string, List<string>> errors)
    {
        foreach (var err in errors)
        {
            messageStore.Add(CurrentEditContext.Field(err.Key), err.Value);
        }

        CurrentEditContext.NotifyValidationStateChanged();
    }

    public void ClearErrors()
    {
        messageStore.Clear();
        CurrentEditContext.NotifyValidationStateChanged();
    }
}
```

## <a name="business-logic-validation"></a>ビジネス ロジックの検証

ビジネス ロジックの検証は、ディクショナリ内のフォーム エラーを受け取る[検証コンポーネント](#validator-components)を使用して実現されます。

次に例を示します。

* この記事の「[検証コンポーネント](#validator-components)」セクションの`CustomValidator` コンポーネントが使用されています。
* 検証では、ユーザーが`Defense`船の分類 (`Classification`) を選択した場合に船の説明 (`Description`) の値を要求します。

検証メッセージは、コンポーネント内で設定されると、検証コンポーネントの <xref:Microsoft.AspNetCore.Components.Forms.ValidationMessageStore> に追加され、<xref:Microsoft.AspNetCore.Components.Forms.EditForm> に表示されます。

```razor
@page "/FormsValidation"

<h1>Starfleet Starship Database</h1>

<h2>New Ship Entry Form</h2>

<EditForm Model="@starship" OnValidSubmit="@HandleValidSubmit">
    <DataAnnotationsValidator />
    <CustomValidator @ref="customValidator" />
    <ValidationSummary />

    ...

</EditForm>

@code {
    private CustomValidator customValidator;
    private Starship starship = new Starship() { ProductionDate = DateTime.UtcNow };

    private void HandleValidSubmit()
    {
        customValidator.ClearErrors();

        var errors = new Dictionary<string, List<string>>();

        if (starship.Classification == "Defense" &&
                string.IsNullOrEmpty(starship.Description))
        {
            errors.Add(nameof(starship.Description),
                new List<string>() { "For a 'Defense' ship classification, " +
                "'Description' is required." });
        }

        if (errors.Count() > 0)
        {
            customValidator.DisplayErrors(errors);
        }
        else
        {
            // Process the form
        }
    }
}
```

> [!NOTE]
> [検証コンポーネント](#validator-components)を使用する代わりに、データ注釈検証属性を使用することもできます。 フォームのモデルに適用されるカスタム属性は、<xref:Microsoft.AspNetCore.Components.Forms.DataAnnotationsValidator> コンポーネントを使用してアクティブ化されます。 サーバー側の検証で使用する場合、属性はサーバーで実行できる必要があります。 詳細については、「<xref:mvc/models/validation#alternatives-to-built-in-attributes>」を参照してください。

## <a name="server-validation"></a>サーバーの検証

サーバーの検証は、サーバー[検証コンポーネント](#validator-components)を使用して実現できます。

* <xref:Microsoft.AspNetCore.Components.Forms.DataAnnotationsValidator> コンポーネントを使用して、フォーム内のクライアント側の検証を処理します。
* フォームによってクライアント側の検証が渡されると (<xref:Microsoft.AspNetCore.Components.Forms.EditForm.OnValidSubmit> が呼び出されると)、フォーム処理のために、<xref:Microsoft.AspNetCore.Components.Forms.EditContext.Model?displayProperty=nameWithType> がバックエンド サーバー API に送信されます。
* サーバーでモデルの検証を処理します。
* サーバー API には、組み込みのフレームワーク データ注釈検証と開発者によって提供されるカスタムの検証ロジックの両方が含まれています。 サーバーで検証が成功すると、フォームが処理され、成功の状態コード (*200 - OK*) が返されます。 検証が失敗すると、失敗の状態コード (*400 - 無効な要求*) とフィールド検証エラーが返されます。
* 成功時にフォームを無効にするか、エラーを表示します。

次の例は、以下のものに基づいています。

* [Blazorホストされているプロジェクト テンプレート](xref:blazor/hosting-models#blazor-webassembly)によって作成された、ホストされている Blazor ソリューション。 この例は、[セキュリティと Identity のドキュメント](xref:blazor/security/webassembly/index#implementation-guidance)で説明されている、安全でホストされている Blazor ソリューションのいずれかで使用できます。
* 前述の「[組み込みフォームのコンポーネント](#built-in-forms-components)」セクションの *Starfleet Starship Database* フォーム例。
* Blazor フレームワークの <xref:Microsoft.AspNetCore.Components.Forms.DataAnnotationsValidator> コンポーネント。
* 「[検証コンポーネント](#validator-components)」セクションで示した `CustomValidator` コンポーネント。

次の例では、サーバー API で、ユーザーが `Defense` 船の分類 (`Classification`) を選択した場合に船の説明 (`Description`) の値が提供されるかどうかを検証します。

`Starship` モデルをソリューションの `Shared` プロジェクトに配置して、クライアントとサーバーの両方のアプリでモデルを使用できるようにします。 モデルにはデータ注釈が必要であるため、[`System.ComponentModel.Annotations`](https://www.nuget.org/packages/System.ComponentModel.Annotations) のパッケージ参照を `Shared` プロジェクトのプロジェクト ファイルに追加します。

```xml
<ItemGroup>
  <PackageReference Include="System.ComponentModel.Annotations" Version="{VERSION}" />
</ItemGroup>
```

パッケージのプレビュー版以外の最新バージョンを確認するには、[NuGet.org](https://www.nuget.org/packages/System.ComponentModel.Annotations)でパッケージの **バージョン履歴** を参照してください。

サーバー API プロジェクトでは、starship 検証要求 (`Controllers/StarshipValidation.cs`) を処理し、失敗した検証のメッセージを返すコントローラーを追加します。

```csharp
using System;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Extensions.Logging;
using BlazorSample.Shared;

namespace {ASSEMBLY NAME}.Controllers
{
    [Authorize]
    [ApiController]
    [Route("[controller]")]
    public class StarshipValidationController : ControllerBase
    {
        private readonly ILogger<StarshipValidationController> logger;

        public StarshipValidationController(
            ILogger<StarshipValidationController> logger)
        {
            this.logger = logger;
        }

        [HttpPost]
        public async Task<IActionResult> Post(Starship starship)
        {
            try
            {
                if (starship.Classification == "Defense" && 
                    string.IsNullOrEmpty(starship.Description))
                {
                    ModelState.AddModelError(nameof(starship.Description),
                        "For a 'Defense' ship " +
                        "classification, 'Description' is required.");
                }
                else
                {
                    // Process the form asynchronously
                    // async ...

                    return Ok(ModelState);
                }
            }
            catch (Exception ex)
            {
                logger.LogError("Validation Error: {Message}", ex.Message);
            }

            return BadRequest(ModelState);
        }
    }
}
```

前の例では、プレースホルダー `{ASSEMBLY NAME}` はアプリのアセンブリ名です (たとえば `BlazorSample.Server`)。

サーバーでモデル バインド検証エラーが発生した場合、通常、[`ApiController`](xref:web-api/index) (<xref:Microsoft.AspNetCore.Mvc.ApiControllerAttribute>) により、[既定の無効な要求応答](xref:web-api/index#default-badrequest-response)と <xref:Microsoft.AspNetCore.Mvc.ValidationProblemDetails> が返されます。 *Starfleet Starship Database* フォームのすべてのフィールドが送信されず、フォームの検証が失敗した場合、次の例に示すように、応答には、検証エラー以外のデータも含まれます。

```json
{
  "title": "One or more validation errors occurred.",
  "status": 400,
  "errors": {
    "Identifier": ["The Identifier field is required."],
    "Classification": ["The Classification field is required."],
    "IsValidatedDesign": ["This form disallows unapproved ships."],
    "MaximumAccommodation": ["Accommodation invalid (1-100000)."]
  }
}
```

サーバー API によって、前述の既定の JSON 応答が返された場合、クライアントでは、応答を解析して、`errors` ノードの子を取得することができます。 ただし、ファイルの解析は容易ではありません。 JSON を解析するには、フォーム検証エラーを処理するためのエラーの [`Dictionary<string, List<string>>`](xref:System.Collections.Generic.Dictionary`2) を生成するために、<xref:System.Net.Http.Json.HttpContentJsonExtensions.ReadFromJsonAsync%2A> を呼び出した後に追加のコードが必要です。 サーバー API で検証エラーのみを返すのが理想的です。

```json
{
  "Identifier": ["The Identifier field is required."],
  "Classification": ["The Classification field is required."],
  "IsValidatedDesign": ["This form disallows unapproved ships."],
  "MaximumAccommodation": ["Accommodation invalid (1-100000)."]
}
```

サーバー API の応答を変更して、検証エラーのみが返されるようにするには、`Startup.ConfigureServices` の <xref:Microsoft.AspNetCore.Mvc.ApiControllerAttribute> で注釈が付けられたアクションで呼び出されるデリゲートを変更します。 API エンドポイント (`/StarshipValidation`) の場合、<xref:Microsoft.AspNetCore.Mvc.BadRequestObjectResult> と <xref:Microsoft.AspNetCore.Mvc.ModelBinding.ModelStateDictionary> が返されます。 他の API エンドポイントの場合、オブジェクトの結果と新しい <xref:Microsoft.AspNetCore.Mvc.ValidationProblemDetails>が返され、既定の動作が保持されます。

```csharp
using Microsoft.AspNetCore.Mvc;

...

services.AddControllersWithViews()
    .ConfigureApiBehaviorOptions(options =>
    {
        options.InvalidModelStateResponseFactory = context =>
        {
            if (context.HttpContext.Request.Path == "/StarshipValidation")
            {
                return new BadRequestObjectResult(context.ModelState);
            }
            else
            {
                return new BadRequestObjectResult(
                    new ValidationProblemDetails(context.ModelState));
            }
        };
    });
```

詳細については、「<xref:web-api/handle-errors#validation-failure-error-response>」を参照してください。

クライアント プロジェクトでは、「[検証コンポーネント](#validator-components)」セクションで示した検証コンポーネントを追加します。

また、*Starfleet Starship Database* フォームが、`CustomValidator` コンポーネントを使用してサーバー検証エラーを示すように更新されます。 サーバー API によって検証メッセージが返されると、それらは、`CustomValidator` コンポーネントの <xref:Microsoft.AspNetCore.Components.Forms.ValidationMessageStore>に追加されます。 エラーは、フォームの <xref:Microsoft.AspNetCore.Components.Forms.ValidationSummary>で表示するために、フォームの <xref:Microsoft.AspNetCore.Components.Forms.EditContext> で使用することができます。

```razor
@page "/FormValidation"
@using System.Net
@using System.Net.Http.Json
@using Microsoft.AspNetCore.Authorization
@using Microsoft.AspNetCore.Components.WebAssembly.Authentication
@using Microsoft.Extensions.Logging
@using BlazorSample.Shared
@attribute [Authorize]
@inject HttpClient Http
@inject ILogger<FormValidation> Logger

<h1>Starfleet Starship Database</h1>

<h2>New Ship Entry Form</h2>

<EditForm Model="@starship" OnValidSubmit="@HandleValidSubmit">
    <DataAnnotationsValidator />
    <CustomValidator @ref="customValidator" />
    <ValidationSummary />

    <p>
        <label>
            Identifier:
            <InputText @bind-Value="starship.Identifier" disabled="@disabled" />
        </label>
    </p>
    <p>
        <label>
            Description (optional):
            <InputTextArea @bind-Value="starship.Description" 
                disabled="@disabled" />
        </label>
    </p>
    <p>
        <label>
            Primary Classification:
            <InputSelect @bind-Value="starship.Classification" disabled="@disabled">
                <option value="">Select classification ...</option>
                <option value="Exploration">Exploration</option>
                <option value="Diplomacy">Diplomacy</option>
                <option value="Defense">Defense</option>
            </InputSelect>
        </label>
    </p>
    <p>
        <label>
            Maximum Accommodation:
            <InputNumber @bind-Value="starship.MaximumAccommodation" 
                disabled="@disabled" />
        </label>
    </p>
    <p>
        <label>
            Engineering Approval:
            <InputCheckbox @bind-Value="starship.IsValidatedDesign" 
                disabled="@disabled" />
        </label>
    </p>
    <p>
        <label>
            Production Date:
            <InputDate @bind-Value="starship.ProductionDate" disabled="@disabled" />
        </label>
    </p>

    <button type="submit" disabled="@disabled">Submit</button>

    <p style="@messageStyles">
        @message
    </p>

    <p>
        <a href="http://www.startrek.com/">Star Trek</a>,
        &copy;1966-2019 CBS Studios, Inc. and
        <a href="https://www.paramount.com">Paramount Pictures</a>
    </p>
</EditForm>

@code {
    private bool disabled;
    private string message;
    private string messageStyles = "visibility:hidden";
    private CustomValidator customValidator;
    private Starship starship = new Starship() { ProductionDate = DateTime.UtcNow };

    private async Task HandleValidSubmit(EditContext editContext)
    {
        customValidator.ClearErrors();

        try
        {
            var response = await Http.PostAsJsonAsync<Starship>(
                "StarshipValidation", (Starship)editContext.Model);

            var errors = await response.Content
                .ReadFromJsonAsync<Dictionary<string, List<string>>>();

            if (response.StatusCode == HttpStatusCode.BadRequest && 
                errors.Count() > 0)
            {
                customValidator.DisplayErrors(errors);
            }
            else if (!response.IsSuccessStatusCode)
            {
                throw new HttpRequestException(
                    $"Validation failed. Status Code: {response.StatusCode}");
            }
            else
            {
                disabled = true;
                messageStyles = "color:green";
                message = "The form has been processed.";
            }
        }
        catch (AccessTokenNotAvailableException ex)
        {
            ex.Redirect();
        }
        catch (Exception ex)
        {
            Logger.LogError("Form processing error: {Message}", ex.Message);
            disabled = true;
            messageStyles = "color:red";
            message = "There was an error processing the form.";
        }
    }
}
```

> [!NOTE]
> [検証コンポーネント](#validator-components)の代わりに、データ注釈検証属性を使用することもできます。 フォームのモデルに適用されるカスタム属性は、<xref:Microsoft.AspNetCore.Components.Forms.DataAnnotationsValidator> コンポーネントを使用してアクティブ化されます。 サーバー側の検証で使用する場合、属性はサーバーで実行できる必要があります。 詳細については、「<xref:mvc/models/validation#alternatives-to-built-in-attributes>」を参照してください。

> [!NOTE]
> このセクションで説明したサーバー側の検証は、このドキュメント セットにある、Blazor WebAssemblyでホストされるどのソリューション例にも適しています。
>
> * [Azure Active Directory (AAD)](xref:blazor/security/webassembly/hosted-with-azure-active-directory)
> * [Azure Active Directory (AAD) B2C](xref:blazor/security/webassembly/hosted-with-azure-active-directory-b2c)
> * [Identity サーバー](xref:blazor/security/webassembly/hosted-with-identity-server)

## <a name="inputtext-based-on-the-input-event"></a>入力イベントに基づく InputText

`change` イベントではなく、`input` イベントを使用するカスタム コンポーネントを作成するには、<xref:Microsoft.AspNetCore.Components.Forms.InputText> コンポーネントを使用します。

次の例では、`CustomInputText` コンポーネントによってフレームワークの `InputText` コンポーネントが継承され、イベント バインディングが `oninput` イベントに設定されます。

`Shared/CustomInputText.razor`:

```razor
@inherits InputText

<input @attributes="AdditionalAttributes" class="@CssClass" 
    @bind="CurrentValueAsString" @bind:event="oninput" />
```

`CustomInputText` コンポーネントは、<xref:Microsoft.AspNetCore.Components.Forms.InputText> が使用される場所であればどこでも使用できます。

`Pages/TestForm.razor`:

```razor
@page "/testform"
@using System.ComponentModel.DataAnnotations;

<EditForm Model="@exampleModel" OnValidSubmit="@HandleValidSubmit">
    <DataAnnotationsValidator />
    <ValidationSummary />

    <CustomInputText @bind-Value="exampleModel.Name" />

    <button type="submit">Submit</button>
</EditForm>

<p>
    CurrentValue: @exampleModel.Name
</p>

@code {
    private ExampleModel exampleModel = new ExampleModel();

    private void HandleValidSubmit()
    {
        ...
    }

    public class ExampleModel
    {
        [Required]
        [StringLength(10, ErrorMessage = "Name is too long.")]
        public string Name { get; set; }
    }
}
```

## <a name="radio-buttons"></a>ラジオ ボタン

::: moniker range=">= aspnetcore-5.0"

ラジオ ボタン グループを作成するには、`InputRadioGroup` コンポーネントと共に `InputRadio` コンポーネントを使用します。 次の例では、「[組み込みフォームのコンポーネント](#built-in-forms-components)」セクションで説明されている `Starship` モデルにプロパティが追加されています。

```csharp
[Required]
[Range(typeof(Manufacturer), nameof(Manufacturer.SpaceX), 
    nameof(Manufacturer.VirginGalactic), ErrorMessage = "Pick a manufacturer.")]
public Manufacturer Manufacturer { get; set; } = Manufacturer.Unknown;

[Required, EnumDataType(typeof(Color))]
public Color? Color { get; set; } = null;

[Required, EnumDataType(typeof(Engine))]
public Engine? Engine { get; set; } = null;
```

アプリに以下の `enums` を追加します。 `enums` を保持する新しいファイルを作成するか、`Starship.cs` ファイルに `enums` を追加します。 `Starship` モデルと *Starfleet Starship Database* フォームから `enums` にアクセスできるようにします。

```csharp
public enum Manufacturer { SpaceX, NASA, ULA, VirginGalactic, Unknown }
public enum Color { ImperialRed, SpacecruiserGreen, StarshipBlue, VoyagerOrange }
public enum Engine { Ion, Plasma, Fusion, Warp }
```

「[組み込みフォームのコンポーネント](#built-in-forms-components)」セクションで説明している *Starfleet Starship Database* フォームを更新します。 生成するコンポーネントを追加します。

* 船舶製造元のラジオ ボタン グループ。
* 船の色およびエンジン用の入れ子になったラジオ ボタン グループ。

```razor
<p>
    <InputRadioGroup @bind-Value="starship.Manufacturer">
        Manufacturer:
        <br>
        @foreach (var manufacturer in (Manufacturer[])Enum
            .GetValues(typeof(Manufacturer)))
        {
            <InputRadio Value="manufacturer" />
            @manufacturer
            <br>
        }
    </InputRadioGroup>
</p>

<p>
    Pick one color and one engine:
    <InputRadioGroup Name="engine" @bind-Value="starship.Engine">
        <InputRadioGroup Name="color" @bind-Value="starship.Color">
            <InputRadio Name="color" Value="Color.ImperialRed" />Imperial Red<br>
            <InputRadio Name="engine" Value="Engine.Ion" />Ion<br>
            <InputRadio Name="color" Value="Color.SpacecruiserGreen" />
                Spacecruiser Green<br>
            <InputRadio Name="engine" Value="Engine.Plasma" />Plasma<br>
            <InputRadio Name="color" Value="Color.StarshipBlue" />Starship Blue<br>
            <InputRadio Name="engine" Value="Engine.Fusion" />Fusion<br>
            <InputRadio Name="color" Value="Color.VoyagerOrange" />
                Voyager Orange<br>
            <InputRadio Name="engine" Value="Engine.Warp" />Warp<br>
        </InputRadioGroup>
    </InputRadioGroup>
</p>
```

> [!NOTE]
> `Name` を省略した場合、`InputRadio` コンポーネントは最新の先祖を基準にグループ化されます。

::: moniker-end

::: moniker range="< aspnetcore-5.0"

フォームでオプション ボタンを使用する場合、オプション ボタンはグループとして評価されるため、データ バインディングが他の要素と異なる方法で処理されます。 各オプション ボタンの値は固定ですが、オプション ボタン グループの値は、選択されたオプション ボタンの値です。 以下の例では、次のことを行っています。

* オプション ボタン グループのデータバインディングを処理する。
* カスタム `InputRadio` コンポーネントを使用した検証をサポートする。

```razor
@using System.Globalization
@typeparam TValue
@inherits InputBase<TValue>

<input @attributes="AdditionalAttributes" type="radio" value="@SelectedValue" 
       checked="@(SelectedValue.Equals(Value))" @onchange="OnChange" />

@code {
    [Parameter]
    public TValue SelectedValue { get; set; }

    private void OnChange(ChangeEventArgs args)
    {
        CurrentValueAsString = args.Value.ToString();
    }

    protected override bool TryParseValueFromString(string value, 
        out TValue result, out string errorMessage)
    {
        var success = BindConverter.TryConvertTo<TValue>(
            value, CultureInfo.CurrentCulture, out var parsedValue);
        if (success)
        {
            result = parsedValue;
            errorMessage = null;

            return true;
        }
        else
        {
            result = default;
            errorMessage = $"{FieldIdentifier.FieldName} field isn't valid.";

            return false;
        }
    }
}
```

次の <xref:Microsoft.AspNetCore.Components.Forms.EditForm> では、前の `InputRadio` コンポーネントを使用して、ユーザーから評価を取得して検証しています。

```razor
@page "/RadioButtonExample"
@using System.ComponentModel.DataAnnotations

<h1>Radio Button Group Test</h1>

<EditForm Model="@model" OnValidSubmit="@HandleValidSubmit">
    <DataAnnotationsValidator />
    <ValidationSummary />

    @for (int i = 1; i <= 5; i++)
    {
        <label>
            <InputRadio name="rate" SelectedValue="i" @bind-Value="model.Rating" />
            @i
        </label>
    }

    <button type="submit">Submit</button>
</EditForm>

<p>You chose: @model.Rating</p>

@code {
    private Model model = new Model();

    private void HandleValidSubmit()
    {
        ...
    }

    public class Model
    {
        [Range(1, 5)]
        public int Rating { get; set; }
    }
}
```

::: moniker-end

## <a name="binding-select-element-options-to-c-object-null-values"></a>C# オブジェクトの `null` 値への `<select>` 要素オプションのバインド

次の理由により、`<select>` 要素のオプション値を C# オブジェクトの `null` 値として表すよい方法はありません。

* HTML 属性には `null` 値を設定できません。 HTML で `null` に最も近いのは、`<option>` 要素から HTML の `value` 属性を除去することです。
* `value` 属性のない `<option>` を選択すると、ブラウザーではその `<option>` の要素の "*テキスト コンテンツ*" として値が扱われます。

Blazor フレームワークでは、次の処理が必要になるため、既定の動作の抑制は試みられません。

* フレームワークでの特殊なケースの回避策のチェーンの作成。
* 現在のフレームワークの動作に対する破壊的変更。

::: moniker range=">= aspnetcore-5.0"

HTML で `null` に最も近いと思われるものは、"*空の文字列*" の `value` です。 Blazor フレームワークでは、`<select>` の値への双方向バインドに対する空の文字列変換として `null` が処理されます。

::: moniker-end

::: moniker range="< aspnetcore-5.0"

Blazor フレームワークでは、`<select>` の値への双方向バインドを試みるときに、`null` は空の文字列変換として自動的に処理されません。 詳細については、「[null 値への `<select>` のバインドの修正 (dotnet/aspnetcore #23221)](https://github.com/dotnet/aspnetcore/pull/23221)」を参照してください。

::: moniker-end

## <a name="validation-support"></a>検証のサポート

<xref:Microsoft.AspNetCore.Components.Forms.DataAnnotationsValidator> コンポーネントは、データ注釈を使用した検証サポートをカスケードされた <xref:Microsoft.AspNetCore.Components.Forms.EditContext> にアタッチします。 データ注釈を使用した検証のサポートを有効にするには、この明示的なジェスチャが必要です。 データ注釈と異なる検証システムを使用するには、<xref:Microsoft.AspNetCore.Components.Forms.DataAnnotationsValidator> をカスタム実装に置き換えます。 参照ソース [`DataAnnotationsValidator`](https://github.com/dotnet/AspNetCore/blob/master/src/Components/Forms/src/DataAnnotationsValidator.cs)/[`AddDataAnnotationsValidation`](https://github.com/dotnet/AspNetCore/blob/master/src/Components/Forms/src/EditContextDataAnnotationsExtensions.cs) での検査に、ASP.NET Core 実装を使用できます。 上記の参照ソースへのリンクからは、リポジトリの `master` ブランチが提供されます。このブランチは、ASP.NET Core の次回リリースのための製品単位の現行開発を表します。 別のリリースのブランチを選択するには、GitHub ブランチ セレクターを使用します (`release/3.1` など)。

Blazor は 2 種類の検証を実行します。

* *フィールド検証* は、ユーザーがタブでフィールドを離れたときに実行されます。 フィールドの検証時に、<xref:Microsoft.AspNetCore.Components.Forms.DataAnnotationsValidator> コンポーネントによって、報告されたすべての検証結果がフィールドに関連付けられます。
* *モデル検証* は、ユーザーがフォームを送信したときに実行されます。 モデルの検証時に、<xref:Microsoft.AspNetCore.Components.Forms.DataAnnotationsValidator> コンポーネントは、検証結果で報告されたメンバー名に基づいてフィールドを判断しようとします。 個々のメンバーに関連付けられていない検証結果は、フィールドではなくモデルに関連付けられます。

### <a name="validation-summary-and-validation-message-components"></a>検証概要コンポーネントと検証メッセージ コンポーネント

<xref:Microsoft.AspNetCore.Components.Forms.ValidationSummary> コンポーネントは、すべての検証メッセージを要約します。これは[検証概要タグヘルパー](xref:mvc/views/working-with-forms#the-validation-summary-tag-helper)と似ています。

```razor
<ValidationSummary />
```

`Model` パラメーターを使用して、特定のモデルの検証メッセージを出力します。
  
```razor
<ValidationSummary Model="@starship" />
```

<xref:Microsoft.AspNetCore.Components.Forms.ValidationMessage%601> コンポーネントは、特定のフィールドの検証メッセージを表示します。これは、[検証メッセージ タグ ヘルパー](xref:mvc/views/working-with-forms#the-validation-message-tag-helper)に似ています。 `For` 属性と、モデル プロパティに名前を付けるラムダ式で、検証するフィールドを指定します。

```razor
<ValidationMessage For="@(() => starship.MaximumAccommodation)" />
```

<xref:Microsoft.AspNetCore.Components.Forms.ValidationMessage%601> コンポーネントと <xref:Microsoft.AspNetCore.Components.Forms.ValidationSummary> コンポーネントでは、任意の属性をサポートしています。 コンポーネント パラメーターに一致しない属性は、生成された `<div>` 要素または `<ul>` 要素に追加されます。

アプリのスタイルシート (`wwwroot/css/app.css` または `wwwroot/css/site.css`) での検証メッセージのスタイルを制御します。 既定の `validation-message` クラスでは、検証メッセージのテキストの色が赤に設定されます。

```css
.validation-message {
    color: red;
}
```

### <a name="custom-validation-attributes"></a>カスタム検証属性

[カスタム検証属性](xref:mvc/models/validation#custom-attributes)を使用するときに、検証結果がフィールドに正しく関連付けられるようにするには、<xref:System.ComponentModel.DataAnnotations.ValidationResult> の作成時に検証コンテキストの <xref:System.ComponentModel.DataAnnotations.ValidationContext.MemberName> を渡します。

```csharp
using System;
using System.ComponentModel.DataAnnotations;

private class CustomValidator : ValidationAttribute
{
    protected override ValidationResult IsValid(object value, 
        ValidationContext validationContext)
    {
        ...

        return new ValidationResult("Validation message to user.",
            new[] { validationContext.MemberName });
    }
}
```

> [!NOTE]
> <xref:System.ComponentModel.DataAnnotations.ValidationContext.GetService%2A?displayProperty=nameWithType> が `null`です。 `IsValid` メソッドでの検証に対する挿入サービスはサポートされていません。

::: moniker range=">= aspnetcore-5.0"

## <a name="custom-validation-class-attributes"></a>カスタム検証クラス属性

カスタム検証クラス名は、[Bootstrap](https://getbootstrap.com/) などの CSS フレームワークと統合する場合に便利です。 カスタム検証クラス名を指定するには、`FieldCssClassProvider` から派生したクラスを作成し、そのクラスを <xref:Microsoft.AspNetCore.Components.Forms.EditContext> インスタンスに設定します。

```csharp
var editContext = new EditContext(model);
editContext.SetFieldCssClassProvider(new MyFieldClassProvider());

...

private class MyFieldClassProvider : FieldCssClassProvider
{
    public override string GetFieldCssClass(EditContext editContext, 
        in FieldIdentifier fieldIdentifier)
    {
        var isValid = !editContext.GetValidationMessages(fieldIdentifier).Any();

        return isValid ? "good field" : "bad field";
    }
}
```

::: moniker-end

### <a name="no-locblazor-data-annotations-validation-package"></a>Blazor データ注釈検証パッケージ

[`Microsoft.AspNetCore.Components.DataAnnotations.Validation`](https://www.nuget.org/packages/Microsoft.AspNetCore.Components.DataAnnotations.Validation) は、<xref:Microsoft.AspNetCore.Components.Forms.DataAnnotationsValidator> コンポーネントを使用して検証エクスペリエンスのギャップを埋めるパッケージです。 パッケージは現在、*試験段階* です。

> [!NOTE]
> [`Microsoft.AspNetCore.Components.DataAnnotations.Validation`](https://www.nuget.org/packages/Microsoft.AspNetCore.Components.DataAnnotations.Validation) パッケージには、[Nuget.org](https://www.nuget.org/packages/Microsoft.AspNetCore.Components.DataAnnotations.Validation) に "*リリース候補*" の最新バージョンが含まれています。現時点では、"*試験的*" リリース候補パッケージを継続して使用します。 パッケージのアセンブリは、将来のリリースでフレームワークまたはランタイムのいずれかに移動される可能性があります。 更新の詳細については、[Announcements GitHub リポジトリ](https://github.com/aspnet/Announcements)、[dotnet/aspnetcore GitHub リポジトリ](https://github.com/dotnet/aspnetcore)、またはこのトピック セクションを参照してください。

::: moniker range="< aspnetcore-5.0"

### <a name="compareproperty-attribute"></a>`[CompareProperty]` 属性

<xref:System.ComponentModel.DataAnnotations.CompareAttribute> は、検証結果を特定のメンバーに関連付けないため、<xref:Microsoft.AspNetCore.Components.Forms.DataAnnotationsValidator> コンポーネントで正しく機能しません。 これにより、フィールドレベルの検証と、送信時のモデル全体が検証されたときの動作に一貫性がなくなることがあります。 [`Microsoft.AspNetCore.Components.DataAnnotations.Validation`](https://www.nuget.org/packages/Microsoft.AspNetCore.Components.DataAnnotations.Validation) "*試験的*" パッケージでは、これらの制限を回避する追加の検証属性 `ComparePropertyAttribute` が導入されています。 Blazor アプリでは、`[CompareProperty]` は [`[Compare]`](xref:System.ComponentModel.DataAnnotations.CompareAttribute) 属性の直接の代わりとなるものです。

::: moniker-end

### <a name="nested-models-collection-types-and-complex-types"></a>入れ子になったモデル、コレクション型、および複合型

Blazor では、組み込みの <xref:Microsoft.AspNetCore.Components.Forms.DataAnnotationsValidator> によるデータ注釈を使用したフォーム入力の検証をサポートしています。 ただし、<xref:Microsoft.AspNetCore.Components.Forms.DataAnnotationsValidator>で は、コレクション型または複合型のプロパティではないフォームにバインドされているモデルの最上位レベルのプロパティのみが検証されます。

コレクション型と複合型のプロパティを含む、バインドされたモデルのオブジェクト グラフ全体を検証するには、"*試験的*" [`Microsoft.AspNetCore.Components.DataAnnotations.Validation`](https://www.nuget.org/packages/Microsoft.AspNetCore.Components.DataAnnotations.Validation) パッケージによって提供される `ObjectGraphDataAnnotationsValidator` を使用します。

```razor
<EditForm Model="@model" OnValidSubmit="@HandleValidSubmit">
    <ObjectGraphDataAnnotationsValidator />
    ...
</EditForm>
```

`[ValidateComplexType]` でモデルのプロパティに注釈を付けます。 次のモデル クラスでは、`ShipDescription` クラスに、モデルがフォームにバインドされたときに検証する追加のデータ注釈が含まれています。

`Starship.cs`:

```csharp
using System;
using System.ComponentModel.DataAnnotations;

public class Starship
{
    ...

    [ValidateComplexType]
    public ShipDescription ShipDescription { get; set; } = 
        new ShipDescription();

    ...
}
```

`ShipDescription.cs`:

```csharp
using System;
using System.ComponentModel.DataAnnotations;

public class ShipDescription
{
    [Required]
    [StringLength(40, ErrorMessage = "Description too long (40 char).")]
    public string ShortDescription { get; set; }

    [Required]
    [StringLength(240, ErrorMessage = "Description too long (240 char).")]
    public string LongDescription { get; set; }
}
```

### <a name="enable-the-submit-button-based-on-form-validation"></a>フォームの検証に基づいて送信ボタンを有効にする

フォームの検証に基づいて送信ボタンを有効または無効にするには:

* コンポーネントを初期化するときに、フォームの <xref:Microsoft.AspNetCore.Components.Forms.EditContext> を使用してモデルを割り当てます。
* コンテキストの <xref:Microsoft.AspNetCore.Components.Forms.EditContext.OnFieldChanged> コールバックでフォームを検証して、送信ボタンを有効または無効にします。
* `Dispose` メソッドでイベント ハンドラーをアンフックします。 詳細については、「<xref:blazor/components/lifecycle#component-disposal-with-idisposable>」を参照してください。

> [!NOTE]
> さらに、<xref:Microsoft.AspNetCore.Components.Forms.EditContext>を使用する場合は、<xref:Microsoft.AspNetCore.Components.Forms.EditForm.Model> を <xref:Microsoft.AspNetCore.Components.Forms.EditForm>に割り当てないでください。

```razor
@implements IDisposable

<EditForm EditContext="@editContext">
    <DataAnnotationsValidator />
    <ValidationSummary />

    ...

    <button type="submit" disabled="@formInvalid">Submit</button>
</EditForm>

@code {
    private Starship starship = new Starship() { ProductionDate = DateTime.UtcNow };
    private bool formInvalid = true;
    private EditContext editContext;

    protected override void OnInitialized()
    {
        editContext = new EditContext(starship);
        editContext.OnFieldChanged += HandleFieldChanged;
    }

    private void HandleFieldChanged(object sender, FieldChangedEventArgs e)
    {
        formInvalid = !editContext.Validate();
        StateHasChanged();
    }

    public void Dispose()
    {
        editContext.OnFieldChanged -= HandleFieldChanged;
    }
}
```

前の例では、次の場合に `formInvalid` を `false` に設定します。

* フォームには、有効な既定値が事前に読み込まれています。
* フォームが読み込まれるときに、送信ボタンを有効にしたいと考えます。

上記の方法の副作用として、ユーザーがいずれかのフィールドを操作した後に、<xref:Microsoft.AspNetCore.Components.Forms.ValidationSummary> コンポーネントに無効なフィールドが設定されます。 このシナリオは、次のいずれかの方法で対処できます。

* フォームでは <xref:Microsoft.AspNetCore.Components.Forms.ValidationSummary> コンポーネントを使用しないでください。
* 送信ボタンが選択された (たとえば、`HandleValidSubmit` メソッドで) ときに <xref:Microsoft.AspNetCore.Components.Forms.ValidationSummary> コンポーネントを表示します。

```razor
<EditForm EditContext="@editContext" OnValidSubmit="@HandleValidSubmit">
    <DataAnnotationsValidator />
    <ValidationSummary style="@displaySummary" />

    ...

    <button type="submit" disabled="@formInvalid">Submit</button>
</EditForm>

@code {
    private string displaySummary = "display:none";

    ...

    private void HandleValidSubmit()
    {
        displaySummary = "display:block";
    }
}
```

## <a name="troubleshoot"></a>トラブルシューティング

> InvalidOperationException:EditForm には、Model パラメーターまたは EditContext パラメーター (両方ではなく) が必要です。

<xref:Microsoft.AspNetCore.Components.Forms.EditForm> に、<xref:Microsoft.AspNetCore.Components.Forms.EditForm.Model> **または** <xref:Microsoft.AspNetCore.Components.Forms.EditContext> が指定されていることを確認します。 同じフォームに両方を使用しないでください。

フォームに <xref:Microsoft.AspNetCore.Components.Forms.EditForm.Model> を割り当てるときは、次の例に示すように、そのモデルの種類がインスタンス化されていることを確認します。

```csharp
private ExampleModel exampleModel = new ExampleModel();
```

::: moniker range=">= aspnetcore-5.0"

## <a name="additional-resources"></a>その他の技術情報

* <xref:blazor/file-uploads>

::: moniker-end
