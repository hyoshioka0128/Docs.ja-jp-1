---
title: ASP.NET Core Blazor ファイルのアップロード
author: guardrex
description: InputFile コンポーネントを使用して Blazor 内のファイルをアップロードする方法について説明します。
monikerRange: '>= aspnetcore-5.0'
ms.author: riande
ms.custom: mvc
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
ms.date: 02/18/2021
uid: blazor/file-uploads
ms.openlocfilehash: a31821f03efd39d774a4a3c61d027983a1783e2d
ms.sourcegitcommit: 1436bd4d70937d6ec3140da56d96caab33c4320b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/06/2021
ms.locfileid: "102395124"
---
# <a name="aspnet-core-blazor-file-uploads"></a>ASP.NET Core Blazor ファイルのアップロード

ファイルのアップロードなど、.NET コードにブラウザー ファイル データを読み込むには、`InputFile` コンポーネントを使用します。

> [!WARNING]
> ファイルのアップロードに関するセキュリティのベスト プラクティスに常に従ってください。 詳細については、「<xref:mvc/models/file-uploads#security-considerations>」を参照してください。

## <a name="inputfile-component"></a>`InputFile` コンポーネント

`InputFile` コンポーネントは `file` 型の HTML 入力としてレンダリングされます。

既定では、ユーザーは単一ファイルを選択します。 `multiple` 属性を追加して、ユーザーが一度に複数のファイルをアップロードできるようにします。 ユーザーが 1 つ以上のファイルを選択すると、`InputFile` コンポーネントによって `OnChange` イベントが起動され、選択したファイル リストと各ファイルの詳細へのアクセスを提供する `InputFileChangeEventArgs` が渡されます。

ユーザーが選択したファイルからデータを読み取るには、次のようにします。

* ファイルで `Microsoft.AspNetCore.Components.Forms.IBrowserFile.OpenReadStream` を呼び出し、返されるストリームから読み取ります。 詳細については、「[ファイル ストリーム](#file-streams)」セクションを参照してください。
* `OpenReadStream` によって返される <xref:System.IO.Stream> には、読み取られる `Stream` の最大サイズがバイト単位で適用されます。 既定では、それ以降の読み取りで例外が発生する前に、サイズが 512,000 バイト (500 KB) 以下のファイルを読み取ることができます。 このような制限は、開発者が誤って大きいファイルをメモリに読み取ってしまうことを防ぐために存在します。 `Microsoft.AspNetCore.Components.Forms.IBrowserFile.OpenReadStream` の `maxAllowedSize` パラメーターを使用すると、必要に応じてより大きなサイズを指定できます。
* 受信ファイル ストリームを直接メモリに読み取ることは避けてください。 たとえば、ファイル バイトを <xref:System.IO.MemoryStream> にコピーしたり、バイト配列として読み取ることは避けてください。 このような方法は、特に Blazor Server で、パフォーマンスおよびセキュリティの問題を引き起こす可能性があります。 代わりに、ファイル バイトを外部ストア (BLOB、またはディスク上のファイルなど) にコピーすることを検討してください。

イメージ ファイルを受信するコンポーネントは、ファイルの便利な `RequestImageFileAsync` メソッドを呼び出して、イメージがアプリにストリームされる前に、ブラウザーの JavaScript ランタイム内のイメージ データのサイズを変更できます。

次の例は、コンポーネントでの複数のファイルのアップロードを示しています。 `InputFileChangeEventArgs.GetMultipleFiles` では、複数のファイルを読み取ることができます。 悪意のあるユーザーがアプリで想定されているよりも多くのファイルをアップロードするのを防ぐために、読み取るファイルの予想最大数を指定します。 ファイルのアップロードで複数のファイルがサポートされていない場合、`InputFileChangeEventArgs.File` を使用すると、最初のファイルのみを読み取ることができます。

> [!NOTE]
> <xref:Microsoft.AspNetCore.Components.Forms.InputFileChangeEventArgs> は <xref:Microsoft.AspNetCore.Components.Forms?displayProperty=fullName> 名前空間にあります。これは、通常、アプリの `_Imports.razor` ファイル内の名前空間の 1 つです。

`Pages/UploadFiles.razor`:

```razor
@page "/upload-files"
@using System.IO

<h3>Upload Files</h3>

<p>
    <label>
        Max file size:
        <input type="number" @bind="maxFileSize" />
    </label>
</p>

<p>
    <label>
        Max allowed files:
        <input type="number" @bind="maxAllowedFiles" />
    </label>
</p>

<p>
    <label>
        Upload up to @maxAllowedFiles files of up to @maxFileSize bytes each:
        <InputFile OnChange="@LoadFiles" multiple />
    </label>
</p>

<p>@exceptionMessage</p>

@if (isLoading)
{
    <p>Loading...</p>
}

<ul>
    @foreach (var (file, content) in loadedFiles)
    {
        <li>
            <ul>
                <li>Name: @file.Name</li>
                <li>Last modified: @file.LastModified.ToString()</li>
                <li>Size (bytes): @file.Size</li>
                <li>Content type: @file.ContentType</li>
                <li>Content: @content</li>
            </ul>
        </li>
    }
</ul>

@code {
    private Dictionary<IBrowserFile, string> loadedFiles =
        new Dictionary<IBrowserFile, string>();
    private long maxFileSize = 1024 * 15;
    private int maxAllowedFiles = 3;
    private bool isLoading;
    string exceptionMessage;

    async Task LoadFiles(InputFileChangeEventArgs e)
    {
        isLoading = true;
        loadedFiles.Clear();
        exceptionMessage = string.Empty;

        try
        {
            foreach (var file in e.GetMultipleFiles(maxAllowedFiles))
            {
                using var reader = 
                    new StreamReader(file.OpenReadStream(maxFileSize));

                loadedFiles.Add(file, await reader.ReadToEndAsync());
            }
        }
        catch (Exception ex)
        {
            exceptionMessage = ex.Message;
        }

        isLoading = false;
    }
}
```

`IBrowserFile` は、[ブラウザーによって公開される](https://developer.mozilla.org/docs/Web/API/File#Instance_properties)メタデータをプロパティとして返します。 このメタデータは、事前検証に役立ちます。

## <a name="upload-files-to-a-server"></a>サーバーへのファイルのアップロード

次の例は、ホステッド Blazor WebAssembly ソリューションを使用してサーバーにファイルをアップロードする方法を示しています。

> [!WARNING]
> サーバーにファイルをアップロードする機能をユーザーに提供するときは、十分に注意してください。 詳細については、「<xref:mvc/models/file-uploads#security-considerations>」を参照してください。

アップロードしたファイルの結果は、 **`Shared`** プロジェクトの次の `UploadResult` クラスによって保持されます。 サーバーでファイルのアップロードに失敗すると、ユーザーに表示するために `ErrorCode` でエラー コードが返されます。 安全に格納されたファイル名が、ファイルごとにサーバー上で生成され、表示するために `StoredFileName` でクライアントに返されます。 ファイルは、`FileName` の安全でないまたは信頼されていないファイル名を使用して、クライアントとサーバーの間でキー指定されます。

`UploadResult.cs`:

```csharp
public class UploadResult
{
    public bool Uploaded { get; set; }

    public string FileName { get; set; }

    public string StoredFileName { get; set; }

    public int ErrorCode { get; set; }
}
```

**`Client`** プロジェクト内の次の `UploadFiles` コンポーネント:

* クライアントからファイルをアップロードすることをユーザーに許可します。
* Razor によって文字列が自動的に HTML エンコードされるため、クライアントから提供された信頼できないまたは安全ではないファイル名を UI に表示します。 **他のシナリオでクライアントから提供されたファイル名は、信頼しないでください。**

`Pages/UploadFiles.razor`:

```razor
@page "/upload-files"
@using System.Linq
@using Microsoft.Extensions.Logging
@inject HttpClient Http
@inject ILogger<UploadFiles> logger

<h1>Upload Files</h1>

<p>
    <label>
        Upload up to @maxAllowedFiles files:
        <InputFile OnChange="@OnInputFileChange" multiple />
    </label>
</p>

@if (files.Count > 0)
{
    <div class="card">
        <div class="card-body">
            <ul>
                @foreach (var file in files)
                {
                    <li>
                        File: @file.Name
                        <br>
                        @if (FileUpload(uploadResults, file.Name, logger,
                            out var result))
                        {
                            <span>
                                Stored File Name: @result.StoredFileName
                            </span>
                        }
                        else
                        {
                            <span>
                                There was an error uploading the file
                                (Error: @result.ErrorCode).
                            </span>
                        }
                    </li>
                }
            </ul>
        </div>
    </div>
}

@code {
    private IList<File> files = new List<File>();
    private IList<UploadResult> uploadResults = new List<UploadResult>();
    private int maxAllowedFiles = 3;
    private bool shouldRender;

    protected override bool ShouldRender() => shouldRender;

    private async Task OnInputFileChange(InputFileChangeEventArgs e)
    {
        shouldRender = false;
        long maxFileSize = 1024 * 1024 * 15;
        var upload = false;

        using var content = new MultipartFormDataContent();

        foreach (var file in e.GetMultipleFiles(maxAllowedFiles))
        {
            if (uploadResults.SingleOrDefault(
                f => f.FileName == file.Name) is null)
            {
                var fileContent = new StreamContent(file.OpenReadStream());

                files.Add(
                    new File()
                    {
                        Name = file.Name,
                    });

                if (file.Size < maxFileSize)
                {
                    content.Add(
                        content: fileContent,
                        name: "\"files\"",
                        fileName: file.Name);

                    upload = true;
                }
                else
                {
                    logger.LogInformation("{FileName} not uploaded", file.Name);

                    uploadResults.Add(
                        new UploadResult()
                        {
                            FileName = file.Name,
                            ErrorCode = 6,
                            Uploaded = false,
                        });
                }
            }
        }

        if (upload)
        {
            var response = await Http.PostAsync("/Filesave", content);

            var newUploadResults = await response.Content
                .ReadFromJsonAsync<IList<UploadResult>>();

            uploadResults = uploadResults.Concat(newUploadResults).ToList();
        }

        shouldRender = true;
    }

    private static bool FileUpload(IList<UploadResult> uploadResults,
        string fileName, ILogger<UploadFiles> logger, out UploadResult result)
    {
        result = uploadResults.SingleOrDefault(f => f.FileName == fileName);

        if (result is null)
        {
            logger.LogInformation("{FileName} not uploaded", fileName);
            result = new UploadResult();
            result.ErrorCode = 5;
        }

        return result.Uploaded;
    }

    private class File
    {
        public string Name { get; set; }
    }
}
```

次のコードを使用するには、 **`Server`** プロジェクトのルートに `Development/unsafe_uploads` フォルダーを作成します。

**`Server`** プロジェクトの次の `FilesaveController` コントローラーによって、クライアントからアップロードされたファイルが保存されます。

> [!WARNING]
> この例では、内容をスキャンせずにファイルを保存します。 ほとんどの運用シナリオでは、ファイルにウイルス対策またはマルウェア対策スキャナー API を使用してから、ダウンロードまたは他のシステムで使用できるようにしています。 サーバーにファイルをアップロードする場合のセキュリティに関する考慮事項の詳細については、「<xref:mvc/models/file-uploads#security-considerations>」を参照してください。

`Controllers/FilesaveController.cs`:

```csharp
using System;
using System.Collections.Generic;
using System.IO;
using System.Net;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Hosting;
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Extensions.Logging;

[ApiController]
[Route("[controller]")]
public class FilesaveController : ControllerBase
{
    private readonly IWebHostEnvironment env;
    private readonly ILogger<FilesaveController> logger;

    public FilesaveController(IWebHostEnvironment env, 
        ILogger<FilesaveController> logger)
    {
        this.env = env;
        this.logger = logger;
    }

    [HttpPost]
    public async Task<ActionResult<IList<UploadResult>>> PostFile(
        [FromForm] IEnumerable<IFormFile> files)
    {
        var maxAllowedFiles = 3;
        long maxFileSize = 1024 * 1024 * 15;
        var filesProcessed = 0;
        var resourcePath = new Uri($"{Request.Scheme}://{Request.Host}/");
        IList<UploadResult> uploadResults = new List<UploadResult>();

        foreach (var file in files)
        {
            var uploadResult = new UploadResult();
            string trustedFileNameForFileStorage;
            var untrustedFileName = file.FileName;
            uploadResult.FileName = untrustedFileName;
            var trustedFileNameForDisplay = 
                WebUtility.HtmlEncode(untrustedFileName);

            if (filesProcessed < maxAllowedFiles)
            {
                if (file.Length == 0)
                {
                    logger.LogInformation("{FileName} length is 0", 
                        trustedFileNameForDisplay);
                    uploadResult.ErrorCode = 1;
                }
                else if (file.Length > maxFileSize)
                {
                    logger.LogInformation("{FileName} of {Length} bytes is " +
                        "larger than the limit of {Limit} bytes", 
                        trustedFileNameForDisplay, file.Length, maxFileSize);
                    uploadResult.ErrorCode = 2;
                }
                else
                {
                    try
                    {
                        trustedFileNameForFileStorage = Path.GetRandomFileName();
                        var path = Path.Combine(env.ContentRootPath, 
                            env.EnvironmentName, "unsafe_uploads", 
                            trustedFileNameForFileStorage);
                        using MemoryStream ms = new();
                        await file.CopyToAsync(ms);
                        await System.IO.File.WriteAllBytesAsync(path, ms.ToArray());
                        logger.LogInformation("{FileName} saved at {Path}", 
                            trustedFileNameForDisplay, path);
                        uploadResult.Uploaded = true;
                        uploadResult.StoredFileName = trustedFileNameForFileStorage;
                    }
                    catch (IOException ex)
                    {
                        logger.LogError("{FileName} error on upload: {Message}", 
                            trustedFileNameForDisplay, ex.Message);
                        uploadResult.ErrorCode = 3;
                    }
                }

                filesProcessed++;
            }
            else
            {
                logger.LogInformation("{FileName} not uploaded because the " +
                    "request exceeded the allowed {Count} of files", 
                    trustedFileNameForDisplay, maxAllowedFiles);
                uploadResult.ErrorCode = 4;
            }

            uploadResults.Add(uploadResult);
        }

        return new CreatedResult(resourcePath, uploadResults);
    }
}
```

## <a name="file-streams"></a>ファイル ストリーム

Blazor WebAssembly では、ファイル データはブラウザー内の .NET コードに直接ストリームされます。

Blazor Server では、ファイルがストリームから読み取られるときに、ファイル データがサーバー上の .NET コードに SignalR 接続を介してストリームされます。 <xref:Microsoft.AspNetCore.Components.Forms.RemoteBrowserFileStreamOptions> では、Blazor Server のファイル アップロード特性を構成することができます。

## <a name="additional-resources"></a>その他のリソース

* <xref:mvc/models/file-uploads#security-considerations>
* <xref:blazor/forms-validation>
