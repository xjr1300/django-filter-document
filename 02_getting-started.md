# 始める

<https://django-filter.readthedocs.io/en/stable/guide/usage.html>

- [始める](#始める)
  - [モデル](#モデル)
  - [フィルター](#フィルター)
    - [フィルターの定義](#フィルターの定義)
    - [Metaフィールドを使用したフィルタの生成](#metaフィールドを使用したフィルタの生成)
      - [デフォルトフィルターの上書き](#デフォルトフィルターの上書き)
    - [リクエストを基盤としたフィルタリング](#リクエストを基盤としたフィルタリング)
      - [プライマリ.qsのフィルタリング](#プライマリqsのフィルタリング)
      - [ModelChoiceFilterによる関連したクエリセットのフィルタリング](#modelchoicefilterによる関連したクエリセットのフィルタリング)
    - [Filter.methodを使用したフィルタリングのカスタマイズ](#filtermethodを使用したフィルタリングのカスタマイズ)
  - [ビュー](#ビュー)
  - [URLconf](#urlconf)
  - [テンプレート](#テンプレート)
  - [ジェネリックビューと設定](#ジェネリックビューと設定)

Django-filterは、ユーザーが提供したパラメーターに基づいてクエリをフィルタリングする簡単な方法を提供します。
`Product`モデルがあり、リストページでユーザーが見たい製品を、ユーザーがフィルタさせたいと考えているとします。

> **🖊 注意事項**
> Django Rest Frameworkでdjango-filterを使用している場合、このガイドの後に[DRFとの統合](https://django-filter.readthedocs.io/en/stable/guide/rest_framework.html#drf-integration)を読むことをおすすめします。

## モデル

モデルから始めましょう。

```python
from django.db import models


class Product(models.Model):
    name = models.CharField(max_length=255)
    price = models.DecimalField(max_digits=5, decimal_places=2)
    description = models.TextField()
    release_date = models.DateField()
    manufacturer = models.ForeignKey(Manufacturer, on_delete=models.CASCADE)
```

## フィルター

多くのフィールドがあり、名前、価格、または発売日に基づいて、ユーザーにフィルターさせたいと考えています。
このために、`FilterSet`を作成します。

```python
import django_filters


class ProductFilter(django_filters.FilterSet):
    name = django_filters.CharField(lookup_expr="iexact")

    class Meta:
        model = Product
        fields = ["price", "release_date"]
```

ご覧の通り、これはDjangoの`ModelForm`ととても良く似ています。
ちょうど、`ModelForm`を使用する用に、フィルターをオーバーライドすることもでき、また宣言的な構文を使用して、追加することもできます。

### フィルターの定義

宣言的な構文は、フィルターを作成するときに最高の柔軟性を提供しますが、かなり冗長です。
`FilterSet`の[主要なフィルター引数](https://django-filter.readthedocs.io/en/stable/ref/filters.html#core-arguments)の概要を説明するために、次の例を使用します。

```python
class ProductFilter(django_filters.FilterSet):
    price = django_filters.NumberFilter()
    price__gt = django_filters.NumberFilter(field_name="price", lookup="gt")
    price__lt = django_filters.NumberFilter(field_name="price", lookup_expr="lt")

    release_year = django_filters.NumberFilter(field_name="release_date", lookup_expr="year")
    release_year__gt = django_filters.NumberFilter(field_name="release_date", lookup_expr="year__gt")
    release_year__lt = django_filters.NumberFilter(field_name="release_date", lookup_expr="year__lt")

    manufacturer__name = django_filters.CharField(lookup_expr="icontains")

    class Meta:
        model = Product
        fields = ["price", "release_date", "manufacturer"]
```

フィルターには2つの主要な引数があります。

- `field_name`: フィルターするモデルのフィールドの名前です。
  関連したモデルのフィールドをフィルターするために、Djangoの`__`構文を使用して、"関連"を横断できます。
  例としては、`manufacturer__name`があります。
- `lookup_expr`: フィルタリングするときに使用する[フィールド検索](https://docs.djangoproject.com/en/stable/ref/models/querysets/#field-lookups)です。
  Djangoの`__`構文は、検索の変換をサポートするために使用されます。
  例としては、`year__gte`があります。

`field_name`と`lookup_expr`フィールドは一緒に、Djangoのルックアップ表現を完全に表現しています。
検索表現の詳細な説明は、Djangoの[ルックアップリファレンス](https://docs.djangoproject.com/en/stable/ref/models/lookups/#module-django.db.models.lookups)で提供されています。
django-filterは、変換と最終的な検索の両方を含んだ表現をサポートしています。

### Metaフィールドを使用したフィルタの生成

`FilterSet`メタクラスは、多くのコードの重複なしで、複数のフィルターを簡単に記述するために使用できる`fields`属性を提供しています。
その基礎的な構文は、複数のフィールド名のリストをサポートしています。

```python
import django_filters


class ProductFilter(django_filters.FilterSet):
    class Meta:
        model = Product
        fields = ["price", "release_date"]
```

上記は、`price`と`release_date`フィールドの両方に、"正確に一致"ルックアップを生成します。
さらに、辞書は、それぞれのフィールドに複数のルックアップ表現を指定するために使用されます。

```python
import django_filters


class ProductFilter(django_filters.FilterSet);
    class Meta:
        model = Product
        fields = {
            "price": ["lt", "gt"],
            "release_date": ["exact", "year__gt"]
        }
```

上記は、`price__lt`、`price__gt`、`release_date`、`release_date__year__gt`フィルターを生成します。

> **🖊 注意事項**
> フィルタのルックアップタイプ`exact`は、暗黙的にデフォルトで、よってフィルターの名前に追加してはなりません。
> 上記例において、`release_date`の`exact`フィルターは、`release_date`で`release_date__exact`ではありません。
> これは、`FILTERS_DEFAULT_LOOKUP_EXPR`設定で上書きできます。

`Meta`クラス内の`fields`シーケンス内の項目は、関連するモデルのフィールドをフィルターする、Djangoの`__`構文を使用した"関連パス"を含むかもしれません。

```python
class ProductFilter(django_filters.FilterSet):
    class Meta:
        model = Product
        fields = ["manufacturer__country"]
```

#### デフォルトフィルターの上書き

`django.contrib.admin.ModelAdmin`と同様に、`Meta`クラスで`filter_overrides`を使用して、同じ種類のすべてのモデルフィールドのためのデフォルトフィルターを上書きできます。

```python
class ProductFilter(django_filters.FilterSet):
    class Meta:
        model = Product
        fields = {
            "name": ["exact"],
            "release_date": ["isnull"],
        }
        filter_overrides = {
            models.CharField: {
                "filter_class": django_filters.CharField,
                "extra": lambda f: {
                    "lookup_expr": "icontains",
                },
            },
            models.BooleanField: {
                "filter_class": django_filters.BooleanFilter,
                "extra": lambda f: {
                    "widget": forms.CheckboxInput,
                },
            },
        }
```

### リクエストを基盤としたフィルタリング

`FilterSet`は、オプションの`request`引数で初期化されるかもしれません。
リクエストオブジェクトが渡された場合、フィルタリングしている間そのリクエストにアクセスできます。
これは、現在ログインしているユーザーまたは`Accepts-Languages`ヘッダーのような、リクエストの属性によってフィルターできるようにします。

> **🖊 注意事項**
> *request*は、*FilterSet*インスタンスに提供される保証はありません。
> リクエストに依存している任意のコードは、*None*の場合を処理するべきです。

#### プライマリ.qsのフィルタリング

`request`オブジェクトによるプライマリクエリセットをフィルターするために、単純に`FilterSet.qs`属性を上書きします。
例えば、公開されているブログ記事と、ログインしているユーザーによって所有されるブログ記事（おそらく著者の草稿記事）にフィルターすることができます。

```python
class ArticleFilter(django_filters.FilterSet):
    class Meta:
        model = Article
        fields = [ ... ]

    @property
    def qs(self):
        parent = super().qs
        author = getattr(self.request, "user", None)
        return parent.filter(is_published=True) \
            | parent.filter(author=author)
```

#### ModelChoiceFilterによる関連したクエリセットのフィルタリング

`ModelChoiceFilter`と`ModelMultipleChoiceFilter`の`queryset`引数は、呼び出し可能な振る舞いをサポートします。
呼び出し可能なオブジェクトが渡された場合、その唯一の引数として`request`を伴い呼び出されます。
これは、`FilterSet.__init__`のオーバーライドを再度並べ替えすることなしに、同じ種類のリクエストに基づいたフィルタリングを実行できるようにします。

```python
def departments(request):
    if request is None:
        return Department.objects.none()

    company = request.user.company
    return company.department_set.all()


class EmployeeFilter(django_filters.FilterSet):
    department = filters.ModelChoiceFilter(queryset=departments)
    ...
```

### Filter.methodを使用したフィルタリングのカスタマイズ

フィルタリングを実行するために`method`を記述することによって、フィルターの振る舞いを制御できます。
詳細な情報は[メソッドリファレンス](https://django-filter.readthedocs.io/en/stable/ref/filters.html#filter-method)で確認してください。
`request`のような、`FilterSet`の属性にアクセスするかもしれないことに注意してください。

```python
class F(django_filters.FilterSet):
    username = CharFilter(method="my_custom_filter")

    class Meta:
        model = User
        fields = ["username"]

    def my_custom_filter(self, queryset, name, value):
        return queryset.filter(**{
          name: value,
        })
```

## ビュー

ビューを記述する必要があります。

```python
def product_list(request):
    f = ProductFilter(request.GET, queryset=Product.objects.all())
    return render(request, "my_app/template.html", {"filter": f})
```

クエリセット引数が提供されていない場合、モデルのデフォルトマネージャ内のすべてのアイテムが使用されます。

例えば、ページ分けする場合など、ビュー内でフィルタされたオブジェクトにアクセスしたい場合、それはできます。
それらは、`f.qs`内にあります。

## URLconf

ビューを呼び出すためにURLパターンを使用する必要があります。

```python
path("list/", views.product_list, name="product-list")
```

## テンプレート

そして最後に、テンプレートが必要です。

```html
{% extends "base.html" %}

{% block content %}
    <form method="get">
        {{ filter.form.as_p }}
        <input type="submit" />
    </form>
    {% for obj in filter.qs %}
        {{ obj.name }} - ${{ obj.price }}<br />
    {% endfor %}
{% endblock %}
```

必要なものはこれだけです。
`form`属性は通常のDjangoのフォームを含み、`FilterSet.qs`を繰り返すとき、結果のクエリセット内のオブジェクトを得ます。

## ジェネリックビューと設定

上記の使用方法に加えて、django-filterを含んだクラスベースドジェネリックビューもあり、それは、`django_filters.views.FilterView`にあります。
Django自身の`ListView`と同じ様に、`model`または`filterset_class`引数を提供する必要があります。

```python
# urls.py
from django.urls import path
from django_filters.views import FilterView

from myapp.models import Product

urlpatterns = [
    path("list/", FilterView.as_view(model=Product), name="product-list"),
]
```

オプションで`model`を提供する場合、`FilterSet`クラスを自動的に構築するために含めたいフィールドのリストまたはタプルを指定するために、`filterset_fields`を設定できます。

`<app>/<model>_filter.html`にテンプレートを提供する必要があり、それは`filter`コンテキストパラメーターを取得します。
さらに、コンテキストは、フィルタされたクエリセットに保持した`object_list`を含みます。

従来からの関数的なジェネリックビューは、まだdjango-filter`に含まれていますが、その使用は推奨されません。
それは、`django_filters.views.object_filter`で見つけることができます。
クラスベースドビューと同じ引数をそれに提供する必要があります。

```python
# urls.py
from django.urls import path
from django_filters.views import object_filter

from myapp.models import Product

urlpatterns = [
    path("list/", object_filter, {"model": Product}, name="product-list"),
]
```

テンプレートの必要なこととそのコンテキストへんすうは、上記クラスベースドビューと同じです。
