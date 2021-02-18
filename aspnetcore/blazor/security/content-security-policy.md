---
title: ASP.NET Core Blazor のコンテンツ セキュリティ ポリシーを適用する
author: guardrex
description: ASP.NET Core Blazor アプリでコンテンツ セキュリティ ポリシー (CSP) を使用して、クロスサイト スクリプティング (XSS) 攻撃から保護する方法について説明します。
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 05/19/2020
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
uid: blazor/security/content-security-policy
ms.openlocfilehash: f44f348947e31864f5d0d44c9caf1a3aa9c3b1d4
ms.sourcegitcommit: 1166b0ff3828418559510c661e8240e5c5717bb7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/12/2021
ms.locfileid: "100280336"
---
# <a name="enforce-a-content-security-policy-for-aspnet-core-blazor"></a>ASP.NET Core Blazor のコンテンツ セキュリティ ポリシーを適用する

[クロスサイト スクリプティング (XSS)](xref:security/cross-site-scripting) は、アプリのレンダリングされたコンテンツに攻撃者が 1 つ以上の悪意のあるクライアント側スクリプトを配置するセキュリティの脆弱性です。 コンテンツ セキュリティ ポリシー (CSP) は、ブラウザーに次の有効な内容を通知することで、XSS 攻撃から保護することができます。

* スクリプト、スタイルシート、画像など、読み込まれたコンテンツのソース。
* ページによって実行され、フォームの許可された URL ターゲットを指定するアクション。
* 読み込むことができるプラグイン。

CSP をアプリに適用するために、開発者は 1 つ以上の `Content-Security-Policy` ヘッダーまたは `<meta>` タグに複数の CSP コンテンツ セキュリティ "*ディレクティブ*" を指定します。

ポリシーは、ページの読み込み中にブラウザーによって評価されます。 ブラウザーによりページのソースが検査され、コンテンツ セキュリティ ディレクティブの要件を満たしているかどうかが判断されます。 リソースのポリシー ディレクティブが満たされていない場合、ブラウザーでリソースが読み込まれません。 たとえば、サードパーティのスクリプトを許可しないポリシーについて考えてみます。 ページの `src` 属性にサードパーティから発生した `<script>` タグが含まれている場合、ブラウザーによってスクリプトの読み込みが禁止されます。

CSP は、Chrome、Microsoft Edge、Firefox、Opera、Safari など、最新のデスクトップおよびモバイル ブラウザーのほとんどでサポートされています。 CSP は、Blazor アプリで推奨されています。

## <a name="policy-directives"></a>ポリシー ディレクティブ

最小限として、Blazor アプリの次のディレクティブとソースを指定します。 必要に応じて、ディレクティブとソースを追加してください。 次のディレクティブは、この記事の「[ポリシーの適用](#apply-the-policy)」セクションで使用されます。そこでは、Blazor WebAssembly と Blazor Server のセキュリティ ポリシーの例を示しています。

* [base-uri](https://developer.mozilla.org/docs/Web/HTTP/Headers/Content-Security-Policy/base-uri): ページの `<base>` タグの URL を制限します。 `self` を指定して、スキームやポート番号など、アプリの配信元が有効なソースであることを示します。
* [block-all-mixed-content](https://developer.mozilla.org/docs/Web/HTTP/Headers/Content-Security-Policy/block-all-mixed-content): HTTP と HTTPS の混合コンテンツの読み込みを禁止します。
* [default-src](https://developer.mozilla.org/docs/Web/HTTP/Headers/Content-Security-Policy/default-src): ポリシーによって明示的に指定されていないソース ディレクティブのフォールバックを示します。 `self` を指定して、スキームやポート番号など、アプリの配信元が有効なソースであることを示します。
* [img-src](https://developer.mozilla.org/docs/Web/HTTP/Headers/Content-Security-Policy/img-src): 画像の有効なソースを示します。
  * `data:` の URL からの画像の読み込みを許可するには `data:` を指定します。
  * HTTPS のエンドポイントからの画像の読み込みを許可するには `https:` を指定します。
* [object-src](https://developer.mozilla.org/docs/Web/HTTP/Headers/Content-Security-Policy/object-src): `<object>`、`<embed>`、`<applet>` タグの有効なソースを示します。 すべての URL ソースを禁止するには、`none` を指定します。
* [script-src](https://developer.mozilla.org/docs/Web/HTTP/Headers/Content-Security-Policy/script-src): スクリプトの有効なソースを示します。
  * ブートストラップ スクリプトの `https://stackpath.bootstrapcdn.com/` ホスト ソースを指定します。
  * `self` を指定して、スキームやポート番号など、アプリの配信元が有効なソースであることを示します。
  * Blazor WebAssembly アプリでは以下のようにします。
    * 必要なスクリプトの読み込みを許可するハッシュを指定します。
    * `eval()` および文字列からコードを作成するメソッドを使用するには、`unsafe-eval` を指定します。
  * Blazor Server アプリでは、必要なスクリプトの読み込みを許可するハッシュを指定します。
* [style-src](https://developer.mozilla.org/docs/Web/HTTP/Headers/Content-Security-Policy/style-src): スタイルシートの有効なソースを示します。
  * ブートストラップ スタイルシートの `https://stackpath.bootstrapcdn.com/` ホスト ソースを指定します。
  * `self` を指定して、スキームやポート番号など、アプリの配信元が有効なソースであることを示します。
  * インライン スタイルの使用を許可するには `unsafe-inline` を指定します。 初期要求後にクライアントとサーバーを再接続するには、Blazor Server アプリの UI にインライン宣言が必要です。 今後のリリースでは、`unsafe-inline` が不要になるように、インライン スタイルが削除される可能性があります。
* [upgrade-insecure-requests](https://developer.mozilla.org/docs/Web/HTTP/Headers/Content-Security-Policy/upgrade-insecure-requests): セキュリティで保護されていない (HTTP) ソースからのコンテンツ URL を HTTPS 経由で安全に取得する必要があることを示します。

上記のディレクティブは、Microsoft Internet Explorer 以外のすべてのブラウザーでサポートされています。

追加のインライン スクリプトの SHA ハッシュを取得するには:

* 「[ポリシーの適用](#apply-the-policy)」セクションに示されている CSP を適用します。
* アプリをローカルで実行しながら、ブラウザーの開発者ツールのコンソールにアクセスします。 CSP ヘッダーまたは `meta` タグが存在する場合、ブラウザーではブロックされたスクリプトのハッシュが計算されて表示されます。
* ブラウザーによって提供されるハッシュを `script-src` ソースにコピーします。 単一引用符を使用して各ハッシュを囲みます。

コンテンツ セキュリティ ポリシー レベル 2 のブラウザー サポート マトリックスは、「[Can I use: Content Security Policy Level 2](https://www.caniuse.com/#feat=contentsecuritypolicy2)」(私は使用できますか? コンテンツ セキュリティ ポリシー レベル 2) を参照してください。

## <a name="apply-the-policy"></a>ポリシーの適用

`<meta>` タグを使用してポリシーを適用します。

* `http-equiv` 属性の値は `Content-Security-Policy` に設定します。
* `content` 属性値にディレクティブを配置します。 各ディレクティブをセミコロン (`;`) で区切ります。
* 常に `meta` タグを `<head>` のコンテンツに配置します。

次のセクションに、Blazor WebAssembly と Blazor Server のポリシーの例を示します。 これらの例は、Blazor のリリースごとにこの記事でバージョン管理されています。 リリースに適したバージョンを使用するには、この Web ページの **[バージョン]** ドロップ ダウン セレクターを使用してドキュメントのバージョンを選択してください。

### Blazor WebAssembly

`wwwroot/index.html` ホスト ページの `<head>` コンテンツで、「[ポリシー ディレクティブ](#policy-directives)」セクションで説明されているディレクティブを適用します。

::: moniker range=">= aspnetcore-5.0"

```html
<meta http-equiv="Content-Security-Policy" 
      content="base-uri 'self';
               block-all-mixed-content;
               default-src 'self';
               img-src data: https:;
               object-src 'none';
               script-src https://stackpath.bootstrapcdn.com/ 
                          'self' 
                          'sha256-v8v3RKRPmN4odZ1CWM5gw80QKPCCWMcpNeOmimNL2AA=' 
                          'unsafe-eval';
               style-src https://stackpath.bootstrapcdn.com/
                         'self'
                         'unsafe-inline';
               upgrade-insecure-requests;">
```

::: moniker-end

::: moniker range="< aspnetcore-5.0"

```html
<meta http-equiv="Content-Security-Policy" 
      content="base-uri 'self';
               block-all-mixed-content;
               default-src 'self';
               img-src data: https:;
               object-src 'none';
               script-src https://stackpath.bootstrapcdn.com/ 
                          'self' 
                          'sha256-v8ZC9OgMhcnEQ/Me77/R9TlJfzOBqrMTW8e1KuqLaqc=' 
                          'sha256-If//FtbPc03afjLezvWHnC3Nbu4fDM04IIzkPaf3pH0=' 
                          'sha256-v8v3RKRPmN4odZ1CWM5gw80QKPCCWMcpNeOmimNL2AA=' 
                          'unsafe-eval';
               style-src https://stackpath.bootstrapcdn.com/
                         'self'
                         'unsafe-inline';
               upgrade-insecure-requests;">
```

::: moniker-end

アプリで必要な場合に、さらに `script-src` と `style-src` ハッシュを追加します。 開発時には、オンライン ツールまたはブラウザー開発者ツールを使用して、ハッシュが自動的に計算されるようにします。 たとえば、次のブラウザー ツール コンソールのエラーは、必要なスクリプトに対するハッシュがポリシーでカバーされていないことを報告するものです。

> 次のコンテンツ セキュリティ ポリシー ディレクティブに違反しているため、インライン スクリプトの実行が拒否されました: "..."。 インライン実行を有効にするには、'unsafe-inline' キーワード、ハッシュ ('sha256-v8v3RKRPmN4odZ1CWM5gw80QKPCCWMcpNeOmimNL2AA=')、または nonce ('nonce-...') のいずれかが必要です。

エラーに関連付けられている特定のスクリプトが、コンソール内でエラーの隣に表示されます。

### Blazor Server

`Pages/_Host.cshtml` ホスト ページの `<head>` コンテンツで、「[ポリシー ディレクティブ](#policy-directives)」セクションで説明されているディレクティブを適用します。

::: moniker range=">= aspnetcore-5.0"

```cshtml
<meta http-equiv="Content-Security-Policy" 
      content="base-uri 'self';
               block-all-mixed-content;
               default-src 'self';
               img-src data: https:;
               object-src 'none';
               script-src https://stackpath.bootstrapcdn.com/ 
                          'self';
               style-src https://stackpath.bootstrapcdn.com/
                         'self' 
                         'unsafe-inline';
               upgrade-insecure-requests;">
```

::: moniker-end

::: moniker range="< aspnetcore-5.0"

```cshtml
<meta http-equiv="Content-Security-Policy" 
      content="base-uri 'self';
               block-all-mixed-content;
               default-src 'self';
               img-src data: https:;
               object-src 'none';
               script-src https://stackpath.bootstrapcdn.com/ 
                          'self' 
                          'sha256-34WLX60Tw3aG6hylk0plKbZZFXCuepeQ6Hu7OqRf8PI=';
               style-src https://stackpath.bootstrapcdn.com/
                         'self' 
                         'unsafe-inline';
               upgrade-insecure-requests;">
```

::: moniker-end

アプリで必要な場合に、さらに `script-src` と `style-src` ハッシュを追加します。 開発時には、オンライン ツールまたはブラウザー開発者ツールを使用して、ハッシュが自動的に計算されるようにします。 たとえば、次のブラウザー ツール コンソールのエラーは、必要なスクリプトに対するハッシュがポリシーでカバーされていないことを報告するものです。

> 次のコンテンツ セキュリティ ポリシー ディレクティブに違反しているため、インライン スクリプトの実行が拒否されました: "..."。 インライン実行を有効にするには、'unsafe-inline' キーワード、ハッシュ ('sha256-v8v3RKRPmN4odZ1CWM5gw80QKPCCWMcpNeOmimNL2AA=')、または nonce ('nonce-...') のいずれかが必要です。

エラーに関連付けられている特定のスクリプトが、コンソール内でエラーの隣に表示されます。

## <a name="meta-tag-limitations"></a>Meta タグに関する制限事項

`<meta>` タグ ポリシーでは、次のディレクティブがサポートされていません。

* [frame-ancestors](https://developer.mozilla.org/docs/Web/HTTP/Headers/Content-Security-Policy/frame-ancestors)
* [report-to](https://developer.mozilla.org/docs/Web/HTTP/Headers/Content-Security-Policy/report-to)
* [report-uri](https://developer.mozilla.org/docs/Web/HTTP/Headers/Content-Security-Policy/report-uri)
* [sandbox](https://developer.mozilla.org/docs/Web/HTTP/Headers/Content-Security-Policy/sandbox)

上記のディレクティブをサポートするには、`Content-Security-Policy` という名前のヘッダーを使用します。 ディレクティブ文字列は、ヘッダーの値です。

## <a name="test-a-policy-and-receive-violation-reports"></a>ポリシーをテストして違反レポートを受信する

テストは、初期ポリシーを作成するときに、サードパーティのスクリプトが誤ってブロックされていないことを確認するのに役立ちます。

ポリシー ディレクティブを適用せずに一定期間にわたってポリシーをテストするには、ヘッダーベースのポリシーの `<meta>` タグの `http-equiv` 属性またはヘッダー名を `Content-Security-Policy-Report-Only` に設定します。 エラー レポートは、JSON ドキュメントとして指定された URL に送信されます。 詳細については、[MDN Web ドキュメント:「Content-Security-Policy-Report-Only](https://developer.mozilla.org/docs/Web/HTTP/Headers/Content-Security-Policy-Report-Only)」を参照してください。

ポリシーがアクティブになっている間に違反を報告するには、次の記事を参照してください。

* [report-to](https://developer.mozilla.org/docs/Web/HTTP/Headers/Content-Security-Policy/report-to)
* [report-uri](https://developer.mozilla.org/docs/Web/HTTP/Headers/Content-Security-Policy/report-uri)

`report-uri` の使用は推奨されなくなりましたが、すべての主要なブラウザーで `report-to` がサポートされるようになるまで、両方のディレクティブを使用する必要があります。 `report-uri` のサポートは、ブラウザーから "*常に*" 削除される可能性があるため、`report-uri` を排他的に使用しないでください。 `report-to` が完全にサポートされたら、ポリシー内の `report-uri` のサポートを削除します。 `report-to` が導入されたかどうかを追跡する方法については、「[Can I use: report-to](https://caniuse.com/#feat=mdn-http_headers_csp_content-security-policy_report-to)」(私は使用できますか? report-to) を参照してください。

アプリがリリースされるたびにポリシーをテストして更新してください。

## <a name="troubleshoot"></a>トラブルシューティング

* エラーは、ブラウザーの開発者ツールのコンソールに表示されます。 ブラウザーには次に関する情報が表示されます。
  * ポリシーに準拠していない要素。
  * ブロックされた項目を許可するようにポリシーを変更する方法。
* ポリシーが完全に有効になるのは、含まれているすべてのディレクティブがクライアントのブラウザーでサポートされている場合のみです。 現在のブラウザーのサポート マトリックスについては、「[Can I use: Content-Security-Policy](https://caniuse.com/#search=Content-Security-Policy)」(私は使用できますか? Content-Security-Policy) を参照してください。

## <a name="additional-resources"></a>その他の技術情報

* [MDN Web ドキュメント:Content-Security-Policy](https://developer.mozilla.org/docs/Web/HTTP/Headers/Content-Security-Policy)
* [コンテンツ セキュリティ ポリシー レベル 2](https://www.w3.org/TR/CSP2/)
* [Google CSP Evaluator](https://csp-evaluator.withgoogle.com/)
