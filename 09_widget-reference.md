# ウィジェットリファレンス

<https://django-filter.readthedocs.io/en/stable/ref/widgets.html>

- [ウィジェットリファレンス](#ウィジェットリファレンス)
  - [LinkWidget](#linkwidget)
  - [BooleanWidget](#booleanwidget)
  - [CSVWidget](#csvwidget)
  - [RangeWidget](#rangewidget)
  - [SuffixedMultiWidget](#suffixedmultiwidget)

これは、ウィジェットとそれらの引数を提供するリストを示したリファレンスドキュメントです。

## LinkWidget

このウィジェットは、実際の`<input>`の代わりに、リンクとしてそれぞれの選択肢をレンダリングします。
それは、追加するカスタマイズ性を上書きする1つのメソッドを持ちます。
`option_string()`は、3つのキーワード引数のプレースホルダーを持つ文字列を返します。

1. `attrs`: これは、最後の`<a>`タグにあるすべての属性を持つ文字列です。
2. `query_string`: これは、`<a>`要素の`href`オプション内で使用されるクエリ文字列です。
3. `label`: これはユーザーに表示されるテキストです。

## BooleanWidget

このウィジェットは、入力をPythonの`True/False`値に変換します。
それは、`True`と`False`のすべてのバリエーションを、内部的なPythonの値に変換します。
それを使用するために、`BooleanFilter`の`widgets`引数にこれを渡します。

```python
active = BooleanFilter(widget=BooleanWidget())
```

## CSVWidget

このウィジェットは、カンマで区切られた値を期待して、それを文字列値のリストに変換します。
それは、フィールドクラスが、型変換と同様に値のリストを処理することを期待しています。

## RangeWidget

このウィジェットは`RangeFilter`とそのサブクラスで使用されます。
それは、通常、範囲の開始と終了の値として動作する2つのinput要素を生成します。
これによって、それは、Djangoの`forms.TextInput`ウィジェットになり、いくつかの引数と値を受け取ります。
それを使用するために、`RangeField`の`widget`引数にそれを渡します。

```python
date_range = DateFromToRangeFilter(widget=RangeWidget(attrs={'placeholder': 'YYYY/MM/DD'}))
```

## SuffixedMultiWidget

インデックスの代わりにカスタムな接尾辞を追加するために、Djangoビルトインの`MultiWidget`の拡張です。
例えば、最小と最大の範囲を受け付ける範囲ウィジェットが挙げられます。
デフォルトでは、クエリ引数の結果は、次のようになります。

```text
GET /products?price_0=10&price_1=25 HTTP/1.1
```

代わりに`SuffixedMultiWidget`を使用することで、人間の優しい接尾辞を提供できます。

```python
class RangeWidget(SuffixedMultiWidget):
    suffixes = ["min", "max"]
```

これでクエリ名はより少し人間工学的になります。

```text
GET /products?price_min=10&price_max=25 HTTP/1.1
```
