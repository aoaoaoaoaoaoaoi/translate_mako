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

```render_body()```関数の
