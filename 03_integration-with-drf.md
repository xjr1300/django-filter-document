# DRFとの統合

<https://django-filter.readthedocs.io/en/stable/guide/rest_framework.html>

- [DRFとの統合](#drfとの統合)
  - [クイックスタート](#クイックスタート)
  - [filterset\_classにFilterSetを追加](#filterset_classにfiltersetを追加)
  - [filterset\_fieldsショートカットの使用](#filterset_fieldsショートカットの使用)
  - [FilterSetの作成の上書き](#filtersetの作成の上書き)
  - [Core APIとOpen APIでスキーマを生成](#core-apiとopen-apiでスキーマを生成)
  - [Crispy Forms](#crispy-forms)
  - [追加のFilterSet機能](#追加のfilterset機能)

[Django REST Framework](http://www.django-rest-framework.org/)との統合は、DRF特有の`FilterSet`と[フィルターバックエンド](http://www.django-rest-framework.org/api-guide/filtering/)を介して提供されます。
これらは、`rest_framework`サブパッケージ内で見つかります。

## クイックスタート

新しい`FilterSet`を使用することは、単にインポートパスの変更を要求します。
`django_filters`からのインポートに変わり、`rest_framework`サブパッケージからインポートします。

```python
from django_filters import rest_framework as filters


class ProductFilter(filters.FilterSet):
    ...
```

またビュークラスは、`filter_backends`に`DjangoFilterBackend`を追加する必要があります。

```python
from django_filters import rest_framework as filters


class ProductList(generics.ListAPIView):
    queryset = Product.objects.all()
    serializer_class = ProductSerializer
    filter_backends = (filters.DjangoFilterBackend,)
    filterset_fields = ("category", "in_stock")
```

デフォルトでdjango-filterバックエンドを使用したい場合、`DEFAULT_FILTER_BACKENDS`設定にそれを追加してください。

```python
# settings.py
INSTALLED_APPS = [
    # ...
    "rest_framework",
    "django_filters",
]

REST_FRAMEWORK = {
    "DEFAULT_FILTER_BACKENDS": (
        "django_filters.rest_framework.DjangoFilterBackend",
        # ...
    )
}
```

## filterset_classにFilterSetを追加

`FilterSet`でフィルタリングを有効にするために、ビュークラスの`filterset_class`パラメーターにそれを追加します。

```python
from rest_framework import generic
from django_filters import rest_framework as filters

from myapp.models import Product


class ProductFilter(filters.FilterSet):
    min_price = filters.NumberFilter(field_name="price", lookup_expr="gte")
    max_price = filters.NumberFilter(field_name="price", lookup_expr="lte")

    class Meta:
        model = Product
        fields = ["category", "in_stock"]


class ProductList(generics.ListAPIView):
    queryset = Product.objects.all()
    serializer_class = ProductSerializer
    filter_backends = (filters.DjangoFilterBackend,)
    filterset_class = ProductFilter
```

## filterset_fieldsショートカットの使用

代わりにビュークラスに`filterset_fields`を追加することで、`FilterSet`の作成を迂回できます。
これは、単に[Meta.fields](https://django-filter.readthedocs.io/en/stable/ref/filterset.html#fields)で`FieldSet`を作成することと同等です。

```python
from rest_framework import generics
from django_filters import rest_framework as filters

from myapp.models import Product


class ProductList(generics.ListAPIView):
    queryset = Product.objects.all()
    filter_backends = (filters.DjangoFilterBackend,)
    filterset_fields = ("category", "in_stock")


# FilterSetと同等です。
class ProductFilter(filters.FilterSet):
    class Meta:
        model = Product
        fields = ("category", "in_stock")
```

`filterset_fields`と`filterset_class`を一緒に使用することは、サポートされていないことに注意してください。

## FilterSetの作成の上書き

`FilterSet`の作成は、バックエンドクラスの次のメソッドを上書きすることでカスタマイズできます。

- `.get_filterset(self, request, queryset, view)`
- `.get_filterset_class(self, view, queryset=None)`
- `.get_filterset_kwargs(self, request, queryset, view)`

これらのメソッドをそれぞれのビューごとに上書きして固有なバックエンドを作成したり、これらのメソッドを使用してビュークラスに独自のフックを記述することができます。

```python
class MyFilterBackend(filters.DjangoFilterBackend);
    def get_filterset_kwargs(self, request, queryset, view):
        kwargs = super().get_filterset_kwargs(request, queryset, view)

        # ビュークラスによって提供されたfiltersetのkwargsを統合
        if hasattr(view, "get_filterset_kwargs"):
            kwargs.update(view.get_filterset_kwargs())

        return kwargs


class BookFilter(filters.FilterSet):
    def __init__(self, *args, author=None, **kwargs):
        super().__init__(*args, **kwargs)
        # ...


class BookViewSet(viewsets.ModelViewSet):
    filter_backends = [MyFilterBackEnd,]
    filterset_class = BookFilter

    def get_filterset_kwargs(self):
        return {
            "author": self.get_author(),
        }
```

## Core APIとOpen APIでスキーマを生成

バックエンドクラスは、Core APIをインストールしたときに自動的に有効になる`get_schema_fields()`と`get_schema_operation_parameters()`、`get_schema_fields()`を実装することで、DRFのスキーマ生成と統合します。
`get_schema_operation_parameters()`は、Open APIで常に有効です（FRM3.9からの新機能）。
スキーマ生成は、通常シームレスに動作しますが、その実装はビューの`get_queryset()`を呼び出すことを期待していません。
ビューはスキーマ生成の間に人為的に構築されないため、`args`と`kwargs`属性は空であるという注意点があります。
URLを解析した引数に依存する場合、`get_queryset()`内でそれらの不在を処理する必要があります。

例えば、`get_queryset`メソッドは次のようにできます。

```python
class IssueViewSet(views.ModelViewSet):
    queryset = models.Issue.objects.all()

    def get_project(self):
        return models.Project.objects.get(pk=self.kwargs["project_id"])

    def get_queryset(self):
        project = self.get_project()
        return self.queryset \
            .filter(project=project) \
            .filter(author=self.request.user)
```

これは次のように書き直せます。

```python
class IssueViewSet(views.ModelViewSet):
    queryset = models.Issue.objects.all()

    def get_project(self):
        try:
            return models.Project.objects.get(pk=self.kwargs["project_id"])
        except models.Project.DoesNotExist:
            return None

    def get_queryset(self):
        project = self.get_project()
        if project is None:
            return self.queryset.none()
        return self.queryset \
            .filter(project=project) \
            .filter(author=self.request.user)
```

または、より単純に・・・

```python
class IssueViewSet(views.ModelViewSet):
    queryset = models.Issue.objects.all()

    def get_queryset(self):
        # project_idはNoneかもしれません。
        return self.queryset \
            .filter(project_id=self.kwargs.get("project_id")) \
            .filter(author=self.request.user)
```

## Crispy Forms

DRFのブラウザブルAPIまたはアドミンAPIを使用している場合、Bootstrap3 HTMLでレンダリングするHTMLビュー内のフィルターフォームの表示を強化する`django-crispy-forms`をインストールするかもしれません。
これは現在サポートされていませんが、バグを修正するプルリクエストを歓迎します。

```sh
pip install django-crispy-forms
```

インストールされ、Djangoの`INSTALLED_APPS`に追加されたcrispy formを使用して、ブラウザブルAPIは、次のように`DjangoFilterBackend`でフィルタリングを制御を表示します。

![crispy forms](https://django-filter.readthedocs.io/en/stable/_images/form.png)

## 追加のFilterSet機能

次の機能はrest frameworkの`FilterSet`に固有です。

- `BooleanFilter`は、小文字の`true/false`を受け付ける、APIフレンドリーな`BooleanWidget`を使用します。
- フィルタ生成は、日時モデルフィールドで`IsoDateTimeFilter`を使用します。
- 発生した`ValidationError`は、それらと等価なDRFの例外として再発生されます。


