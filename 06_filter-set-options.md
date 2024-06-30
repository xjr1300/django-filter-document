# FilterSetオプション

<https://django-filter.readthedocs.io/en/stable/ref/filterset.html>

- [FilterSetオプション](#filtersetオプション)
  - [Metaオプション](#metaオプション)
    - [モデルで自動的にフィルターを生成する](#モデルで自動的にフィルターを生成する)
    - [フィルタ可能なフィールドを定義する](#フィルタ可能なフィールドを定義する)
    - [excludeでフィールドのフィルタを無効にする](#excludeでフィールドのフィルタを無効にする)
    - [formを使用したカスタムフォーム](#formを使用したカスタムフォーム)
    - [filter\_overridesでフィルター生成をカスタマイズする](#filter_overridesでフィルター生成をカスタマイズする)
  - [FilterSetメソッドの上書き](#filtersetメソッドの上書き)
    - [filter\_for\_lookup()](#filter_for_lookup)
  - [filterset\_factoryを使用する](#filterset_factoryを使用する)

このドキュメントは、追加的な`FilterSet`の機能を使用したガイドを提供します。

## Metaオプション

- モデル
- フィールド
- 除外(exclude)
- フォーム
- フィルターの上書き

### モデルで自動的にフィルターを生成する

`FilterSet`は、与えられたモデルのフィールドのために、自動的にフィルターを生成する能力があります。
Djangoの`ModelForm`と同様に、フィルターはモデルのフィールドの型に基づいて作成されます。
このオプションは、`fields`または`exclude`オプションのどちらかと結合されなければならず、それはDjangoの`ModelForm`と同じ要件で、詳細は[ここ](https://docs.djangoproject.com/en/5.0/topics/forms/modelforms/#selecting-the-fields-to-use)で説明されています。

```python
class UserFilter(django_filters.FilterSet):
    class Meta:
        model = User
        fields = ["username", "last_login"]
```

### フィルタ可能なフィールドを定義する

`fields`オプションは、自動的にフィルタを生成するために、`model`と組み合わされます。
生成されたフィルターは、`FilterSet`で定義されたフィルターを上書きしません。
`fields`オプションは、2つの構文を受け付けます。

- フィールド名のリスト
- ルックアップのリストとまっｔピングされたフィールド名の辞書

```python
class UserFilter(django_filters.FilterSet):
    class Meta:
        model = User
        fields = ["username", "last_login"]

# または

class UserFilter(django_filters.FieldSet):
    class Meta:
        model = User
        fields = {
            "username": ["exact", "contains"],
            "last_login": ["exact", "year__gt"],
        }
```

リスト構文は、`fields`内に含まれるそれぞれのフィールドの`exact`ルックアップフィルターを作成します。
辞書構文は、モデルフィールドに対応して宣言されたそれぞれのルックアップ表現でフィルターを作成します。
これらの表現は、変換とルックアップを含んでおり、詳細は[ルックアップリファレンス](https://docs.djangoproject.com/en/stable/ref/models/lookups/#module-django.db.models.lookups)を参照してください。

`fields`リスト内に宣言されたフィルターを含める必要はないことに注意してください。
それをすると、`FilterSet`フォームにフィールドが表示される順番にだけ影響を与えます。
`fields`辞書に宣言的なエイリアスを含めることは、エラーを発生します。

```python
class UserFilter(django_filters.FilterSet):
    username = django_filters.CharFilter()
    login_timestamp = django_filters.IsoDateTimeFilter(field_name="last_login")

    class Meta:
        model = User
        fields = {
            "username": ["exact", "contains"],
            "login_timestamp": ["exact"],
        }

TypeError("'Meta.fields' contains fields that are not defined on this FilterSet: login_timestamp")
```

### excludeでフィールドのフィルタを無効にする

`exclude`オプションは、自動フィルタ生成から除外うするフィールドの名前のブラックリストを受け付けます。
このオプションは、`FilterSet`上に直接宣言されたフィルターを無効化しません。

```python
class UserFilter(django_filters.FilterSet):
    class Meta:
        model = User
        exclude = ["password"]
```

### formを使用したカスタムフォーム

また、内部の`Meta`クラスは、オプションで`form`引数を受け取ります。
これは、`FilterSet.form`サブクラスのフォームクラスです。
これは、`ModelAdmin`の`form`オプションと同様に機能します。

### filter_overridesでフィルター生成をカスタマイズする

また、内部の`Meta`クラスは、オプションで`filter_overrides`引数を受け取ります。
これは、オプションを使用して、モデルフィールドとフィルタークラスをマッピングします。

```python
class ProductFilter(django_filters.FilterSet):
    class Meta:
        model = Product
        fields = ["name", "release_date"]
        filter_overrides = {
            models.CharField: {
                "filter_class": django_filters.CharFilter,
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

## FilterSetメソッドの上書き

クラスメソッドを上書きするとき、`super(MyFilterSet, cls)`の呼び出しは、`NameError`例外を引き起こします。
これは、`FilterSet`クラスが完全に作成される前に、これらのクラスメソッドを呼び出す`FilterSetMetaClass`のためです。
推奨する2つの回避策があります。

1. Python 3.6またはそれ以降を使用している場合、引数なしの`super()`構文を使用します。
2. Pythonの古いバージョンの場合、中間クラスを使用します。例えば・・・

```python
class Intermediate(django_filters.FilterSet):
    @classmethod
    def method(cls, arg):
        super(Intermediate, cls).method(arg)


class ProductFilter(Intermediate):
    class Meta:
        model = Product
        fields = ["..."]
```

### filter_for_lookup()

バージョン0.13.0以前では、フィルタ生成は、`lookup_expr`を使用して行われませんでした。
一般的に、これは変換されたルックアップと同様に、`isnull`、`in`そして`range`ルックアップで生成するために、不正なフィルターを発生させました。
現在の実装は次の振る舞いを提供します。

- `isnull`ルックアップは`BooleanFilter`を返します。
- `in`ルックアップは、CSVを基礎とした`BaseInFilter`から派生したフィルターを返します。
- `range`ルックアップは、CSVを基礎とした`BaseRangeFilter`から派生したフィルタを返します。

モデルフィールド用にフィルターをインスタンス化するために使用される`filter_class`と`param`を上書きしたい場合、`filter_for_lookup()`を上書きでします。例えば・・・

```python
class ProductFilter(django_filters.FilterSet):
    class Meta:
        model = Product
        fields = {
            "release_date": ["exact", "range"],
        }

    @class method
    def filter_for_lookup(cls, f, lookup_type):
        # 日付範囲ルックアップを上書きします。
        if isinstance(f, models.DateField) and lookup_type == "range":
            return django_filters.DateRangeFilter, {}
        # 他はデフォルトの振る舞いを使用します。
        return super().filter_for_lookup(f, lookup_type)
```

## filterset_factoryを使用する

また、`model`の`FilterSet`は、`filterset_factory`によって作成され、それは`FilterSet`の`Meta`を設定したモデルで`FilterSet`を作成します。
カスタマイズした`FilterSet`を`filterset_factory`にわたすことができ、そしてそれは作成された`FilterSet`を用にこの基本く明日を作成します。例えば・・・

> You can pass a customized FilterSet class to the filterset_factory, which then uses this class a a base for the created FilterSet. Ex:

```python
class CustomFilterSet(django_filters.FilterSet):
    class Meta:
        form = CustomFilterSetForm


filterset = filterset_factory(Product, filterset=CustomFilterSet)
```
