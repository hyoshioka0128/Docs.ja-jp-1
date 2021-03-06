```console
npm run release
```

このコマンドは、アプリケーションの実行中に提供するクライアント側資産を生成します。 資産は、*wwwroot* フォルダーに配置されます。

Webpack は、次のタスクを完了しました。

* *wwwroot* ディレクトリの内容を消去しました。
* TypeScript を JavaScript に変換しました&mdash;*トランスコンパイル*と呼ばれるプロセス。
* 生成後の JavaScript のファイルのサイズを縮小しました&mdash;*縮小*と呼ばれるプロセス。
* 処理済みの JavaScript、CSS、および HTML ファイルを *src* から *wwwroot* ディレクトリにコピーしました。
* 次の要素を *wwwroot/index.html* ファイルに挿入しました。
    * *wwwroot/main.\<hash\>.css* ファイルを参照している `<link>` タグ。 このタグは、終了 `</head>` タグの直前に置かれます。
    * 縮小された *wwwroot/main.\<hash\>.js* ファイルを参照している `<script>` タグ。 このタグは、終了 `</body>` タグの直前に置かれます。
