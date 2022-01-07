## 基本的な使い方
このセクションではMakoテンプレートのPython APIについて説明します。
もしPylonsのようなwebフレームワークでMakoを使用している場合は、MakoのAPIを統合する工程は完了しているので次の「[構文]()」セクションに進んでください。

テンプレートを作成して、レンダリングする最も基本的な方法は```Template```クラスの使用です：
```
from mako.template import Template

mytemplate = Template("hello world!")
print(mytemplate.render())
```

上記では、```Template```へのテキスト引数をPythonのモジュールとしてコンパイルしています。
このモジュールは、テンプレートを出力する```render_body()```という関数を含みます。
```mytemplate.render()```を呼ばれると、Makoはテンプレートの実行環境を構築し、```render_body()```関数を呼び出して、出力をバッファに取り込み、文字列内容を返します。

```render_body()```関数内のコードは、変数の名前空間にアクセスします。
変数を指定するには、```Template.render()```関数に追加のキーワード引数として送信します。
```
from mako.template import Template

mytemplate = Template("hello, ${name}!")
print(mytemplate.render(name="jack"))
```

Template.render()メソッドは、Makoを呼び出してContextオブジェクトを作成します。
この[Context]()オブジェクトには、テンプレートにアクセス可能なすべての変数名が格納されており、出力をキャプチャするためのバッファも格納されています。
この[Context]()を自分で作成し、[Template.render_context()]()関数を使って、テンプレートに作成したContextを使ってレンダリングをさせることができます。
```
from mako.template import Template
from mako.runtime import Context
from StringIO import StringIO

mytemplate = Template("hello, ${name}!")
buf = StringIO()
ctx = Context(buf, name="jack")
mytemplate.render_context(ctx)
print(buf.getvalue())
```

## ファイルベースのテンプレートの使用
[Template]()は```filename```キーワード引数を使ってファイルからテンプレートのソースコードを読み込むこともできます：
```
from mako.template import Template

mytemplate = Template(filename='/docs/mytmpl.txt')
print(mytemplate.render())
```

パフォーマンスを向上させるために、ファイルから読み込まれた[Template]()は、生成されたモジュールのソースコードを通常のPythonモジュールファイル（.pyファイル）としてファイルシステムにキャッシュすることもできます。
テンプレートに``` module_directory```の引数を追加するだけでできます：
```
from mako.template import Template

mytemplate = Template(filename='/docs/mytmpl.txt', module_directory='/tmp/mako_modules')
print(mytemplate.render())
```

上記のコードがレンダリングされると、モジュールのソースコードを含んだ``` /tmp/mako_modules/docs/mytmpl.txt.py```ファイルが作成されます。
次回、同じ引数をもつ[Template]()を作成した場合は、このモジュールファイルが自動的に使用されます。

## ```TemplateLookup```の使用
これまでの例は1つの[Template]()オブジェクトを使用しています。
これらのテンプレート内のコードが、別のテンプレートリソースを見つけようとする場合、単純なURI文字列を使用してそれらを見つける何らかの方法が必要になります。
この方法として[TemplateLookup]()クラスが用意されています。
[TemplateLookup]()クラスを使用することで、テンプレート内から他のテンプレートを見つけることができます。
このクラスはテンプレートを探すためのディレクトリのリストと、作成する[Template]()オブジェクトへ渡すキーワード引数を与えて作成されます。
```
from mako.template import Template
from mako.lookup import TemplateLookup

mylookup = TemplateLookup(directories=['/docs'])
mytemplate = Template("""<%include file="header.txt"/> hello world!""", lookup=mylookup)
```

以上のコードでは、header.txtファイルを含むテキスト形式のテンプレートを作成しました。
TemplateLookupオブジェクトにディレクトリ/docsを渡し、header.txtを探せるようにしました。

通常、アプリケーションはテンプレートのほとんどまたはすべてをファイルシステム上のテキストファイルとして保存します。
ここまでの例は、基本的な概念を説明するために工夫されていました。
しかし、実際のアプリケーションでは、TemplateLookup.get_template()という適切な名前のメソッドを使って、TemplateLookupから直接、ほとんどあるいはすべてのテンプレートを取得することになります。このメソッドでは、希望するテンプレートのURIを受け取れます：
```python
from mako.template import Template
from mako.lookup import TemplateLookup

mylookup = TemplateLookup(directories=['/docs'], module_directory='/tmp/mako_modules')

def serve_template(templatename, **kwargs):
    mytemplate = mylookup.get_template(templatename)
    print(mytemplate.render(**kwargs))
```

上記の例では、/docs ディレクトリにテンプレートを探す TemplateLookup を作成し、生成されたモジュールファイルを /tmp/mako_modules ディレクトリに保存しています。
lookupは、与えられたURIを検索ディレクトリのそれぞれに付加してテンプレートを探します。
したがって、/etc/beans/info.txt という URI を与えた場合、ファイル /docs/etc/beans/info.txt を検索し、それ以外の場合はカスタム Mako 例外である TopLevelNotFound 例外を出します。

lookupがテンプレートを見つけるとき、TemplateLookup.get_template()呼び出しに渡されたURIであるuriプロパティもテンプレートに割り当てられます。
テンプレートは、このURIを使ってモジュールファイルの名前を計算します。したがって、上記の例では、templatename の引数に /etc/beans/info.txt を指定すると、モジュールファイル /tmp/mako_modules/etc/beans/info.txt.py が作成されます

### コレクションサイズの設定
また、TemplateLookupは、与えられた時間にテンプレートの固定セットをメモリにキャッシュするという重要なニーズも提供します。
そのため、連続したURI検索では、リクエストごとのテンプレートの完全コンパイルやモジュールの再ロードを行いません。
デフォルトでは、TemplateLookupのサイズは無制限です。
collection_size引数で固定サイズを指定することができます：
```python
mylookup = TemplateLookup(directories=['/docs'],
                module_directory='/tmp/mako_modules', collection_size=500)
```

上記のlookupは、約500のカウントに達するまで、テンプレートをメモリにロードし続けます。
その際、最低限の直近に使用されたスキームを使用して、一定の割合でテンプレートを削除します。

### ファイルシステム・チェックの設定
TemplateLookupのもう一つの重要なフラグはfilesystem_checksです。
デフォルトはTrueで、TemplateLookup.get_template()メソッドでテンプレートが返されるたびに、オリジナルのテンプレートファイルの改訂時刻とテンプレートが最後にロードされた時刻がチェックされ、ファイルがより新しければその内容を再ロードしてテンプレートを再コンパイルします。
実運用システムでは、filesystem_checks を False に設定することで、(使用するファイルシステムの種類によりますが) 性能が多少向上する可能性があります。

## ユニコードとエンコードの使用
TemplateとTemplateLookupはoutput_encodingとencoding_errorsパラメータを受け取り、これを用いて出力をPythonがサポートする任意のコーデックでエンコードすることができます：
```python
from mako.template import Template
from mako.lookup import TemplateLookup

mylookup = TemplateLookup(directories=['/docs'], output_encoding='utf-8', encoding_errors='replace')

mytemplate = mylookup.get_template("foo.txt")
print(mytemplate.render())
```

Python3を使用している場合、output_encoding が設定されていれば Template.render() メソッドは bytes オブジェクトを返します。
そうでなければ、string を返します。

加えて、Template.render_unicode()メソッドがあり、テンプレートの出力をPythonのunicodeオブジェクトとして、またはPython 3ではstringとして返します：
```
print(mytemplate.render_unicode())
```

上記の方法は、出力エンコードキーワードの引数を無視します。
以下の方法でエンコードできます：
```
print(mytemplate.render_unicode().encode('utf-8', 'replace'))
```


Makoが任意のエンコーディングやユニコードでデータを返すことができるのは、テンプレートの基礎となる出力ストリームがPythonのunicodeオブジェクトであることを意味することに注意してください。
この動作は、[ユニコードの章]()で詳しく説明します。

## 例外処理


