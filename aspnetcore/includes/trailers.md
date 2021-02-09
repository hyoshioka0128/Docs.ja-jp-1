HTTP トレーラーは HTTP ヘッダーに似ていますが、応答本文の送信後に送信される点が異なります。 IIS と HTTP.sys の場合は、HTTP/2 応答トレーラーのみがサポートされています。

```csharp
if (httpContext.Response.SupportsTrailers())
{
    httpContext.Response.DeclareTrailer("trailername"); 

    // Write body
    httpContext.Response.WriteAsync("Hello world");

    httpContext.Response.AppendTrailer("trailername", "TrailerValue");
}
```

上記のコード例では次のようになっています。

* `SupportsTrailers` によって、応答におけるトレーラーが確実にサポートされます。
* `DeclareTrailer` によって、指定したトレーラー名が `Trailer` 応答ヘッダーに追加されます。 応答のトレーラーを宣言することは省略可能ですが、推奨されています。 `DeclareTrailer` を呼び出す場合、それは、応答ヘッダーを送信する前に行う必要があります。
* `AppendTrailer` によって、トレーラーが追加されます。
