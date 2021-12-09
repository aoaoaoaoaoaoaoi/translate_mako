# 元のページのURL
https://docs.makotemplates.org/en/latest/defs.html#using-blocks

## ブロックの使い方
```<%block>```タグは```<%def>```タグに工夫を加えて、レイアウト向けに調整して作成したものです。

この機能は、バージョン0.4.1で入った機能です。

ブロックの例：
```
<html>
    <body>
        <%block>
            this is a block.
        </%block>
    </body>
</html>
```

上の例では、シンプルなブロックを定義しています。
ブロックは定義された位置にコンテンツを生成します。
上記のブロックに名前は必要ないので、<b>匿名ブロック</b>と呼ばれています。
そのため、上記のテンプレートのアウトプットは以下のようになります：
```
<html>
    <body>
            this is a block.
    </body>
</html>
```

そのため、上記のブロックは全く意味がありません。
修飾と共に使用するときに、効果が出ます。
例えば、ブロックにフィルターをつけたり：
```
<html>
    <body>
        <%block filter="h">
            <html>this is some escaped html.</html>
        </%block>
    </body>
</html>
```

キャッシング・ディレクティブを使用したりできます：
```
<html>
    <body>
        <%block cached="True" cache_timeout="60">
            This content will be cached for 60 seconds.
        </%block>
    </body>
</html>
```

また、ブロックはdefsと同様に、反復や条件内でも動作します：
```
% if some_condition:
    <%block>condition is met</%block>
% endif
```

ブロックはテンプレートで定義された時点で生成されますが、Python関数は生成されたPythonコードの中に一度だけ存在するので、ブロックをループなどの中に配置しても問題はありません。

## 名前つきブロックの使い方
おそらくブロックが重要になってくるのは名前をつけるときです。
名前付きブロックは、Jinja2のblockタグに近い動作をするように調整されており、継承するテンプレートでオーバーライド可能なレイアウトの領域を定義します。
```<%def>```タグとは対照的に、ブロックに付けられた名前は、どれだけ深く入れ子になっていても、テンプレート全体でグローバルなものとなります：




