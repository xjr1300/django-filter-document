# ヒントと解決

<https://django-filter.readthedocs.io/en/stable/guide/tips.html>

- [ヒントと解決](#ヒントと解決)
  - [宣言されたフィルターの一般的な問題](#宣言されたフィルターの一般的な問題)
    - [フィルターのfield\_nameとlookup\_exprが設定されていない](#フィルターのfield_nameとlookup_exprが設定されていない)
    - [テキスト検索フィルターでlookup\_exprがない](#テキスト検索フィルターでlookup_exprがない)
    - [フィルターとルックアップ表現のミスマッチ（in、range、isnull）](#フィルターとルックアップ表現のミスマッチinrangeisnull)
  - [空の値によるフィルタリング](#空の値によるフィルタリング)
    - [null値でフィルタリングする](#null値でフィルタリングする)
      - [解決策1: isnullでBooleanFilterを使用する](#解決策1-isnullでbooleanfilterを使用する)
      - [解決策2: ChoiceFilterのnull選択を使用する](#解決策2-choicefilterのnull選択を使用する)
      - [解決策4: MultiValueFiledを組み合わせる](#解決策4-multivaluefiledを組み合わせる)
    - [空の文字列でフィルタリングする](#空の文字列でフィルタリングする)
      - [解決策1: マジックバリュー](#解決策1-マジックバリュー)
      - [解決策2: 空文字列フィルター](#解決策2-空文字列フィルター)
  - [相対時間でフィルタリングする](#相対時間でフィルタリングする)
  - [デフォルトとしてinitial値を使用する（非推奨）](#デフォルトとしてinitial値を使用する非推奨)
  - [フィルターにモデルフィールドのhelp\_textを追加する](#フィルターにモデルフィールドのhelp_textを追加する)

## 宣言されたフィルターの一般的な問題

下は、フィルターを作成するときに発生する、いくつかの一般的な問題です。
フィルタが機能する方法をより完全な理解を提供するものとして、これを読むことを推奨します。

### フィルターのfield_nameとlookup_exprが設定されていない

`field_name`と`lookup_expr`はオプショナルである一方、それらを指定することを推奨します。
デフォルトでは、`field_name`が指定されていない場合、`FilterSet`クラスのフィルターの名前が使用されます。
さらに、`lookup_expr`のデフォルトは`exact`です。
次は、誤って設定された価格フィルターの例です。

```python
class ProductFilter(django_filters.FilterSet):
    price__gt = django_filters.NumberFilter()
```

フィルターインスタンスは、フィールド名`price__gt`と`exact`ルックアップタイプを持ちます。
内部では、これは誤って次のように解決されます。

```python
Product.objects.filter(price__gt__exact=value)
```

上記は、`FieldError`を生成する可能性が高くなります。
正しい厚生は次のとおりです。

```python
class ProductFilter(django_filters.FilterSet):
    price__gt = django_filters.NumberFilter(field_name="price", lookup_expr="gt")
```

`filterset_fields`を使用しているとき、次の通りフィールドの辞書内に`lookup_expr`を追加できます。

```python
# ... filter_backends内のDjangoFilterBackendを使用したModelViewSet

filterset_fields = {
  "price": ["gt", "exact"]
}
```

### テキスト検索フィルターでlookup_exprがない

`CharField`と`TextField`にルックアップ表現を設定することを忘れることはとても一般的で、"foo"での検索が"foobar"の結果を返さない理由に困惑します。
これは、おそらく`icontains`ルックアップを実行したいのですが、デフォルトのルックアップタイプが`exact`であることが理由です。

### フィルターとルックアップ表現のミスマッチ（in、range、isnull）

いくつかのルックアップは色々な型の値を期待するため、フィルターとモデルのフィールドの型が直接マッチすることは常に適切ではありません。
これは、`in`、`range`そして`isnull`ルックアップでよく発見される問題です。
次の製品モデルを確認しましょう。

```python
class Product(models.Model):
    category = models.ForeignKey(Category, null=True)
```

カテゴライズされていない製品の検索を有効にする理由で、`category`はオプショナルで与えられています。
次は、誤って構成された`isnull`フィルターです。

```python
class ProductFilter(django_filters.FilterSet):
    uncategorized = django_filters.NumberFilter(field_name="category", lookup_expr="isnull")
```

何が問題なのでしょうか？
`category`の基となる列の型は整数である一方で、`isnull`ルックアップは論理値を期待します。
しかし、`NumberFilter`は、数値のみを検証します。
フィルターは、"式を理解"でなく、それらの`lookup_expr`を基に振る舞いを変更できません。
モデルフィールドに基づいたデータ型の代わりに、ルックアップ表現のデータ型とマッチするフィルターを使用するべきです。
次は、正確にカテゴライズされていない製品と、カテゴリの集合の両方を検索するようにします。

```python
class NumberInFilter(django_filters.BaseInFilter, django_filters.NumberFilter):
    pass

class ProductFilter(django_filters.FilterSet):
    categories = NumberInFilter(field_name="category", lookup_expr="in")
    uncategorized = django_filters.BooleanFilter(field_name="category", lookup_expr="isnull")
```

`in`と`range`の構築に関する詳細はcsv[フィルター](https://django-filter.readthedocs.io/en/stable/ref/filters.html#base-in-filter)を参照してください。

## 空の値によるフィルタリング

空またはnull値でフィルタする必要がある場面が多くあります。
次は、これらの問題へのいくつかの一般的な解決策です。

### null値でフィルタリングする

上記"フィルターとルックアップ表現のミスマッチ"セクションで説明した通り、一般的な問題は、フィールドをnull値で正確にフィルターする方法です。

#### 解決策1: isnullでBooleanFilterを使用する

`isnull`ルックアップで`BooleanFilter`を使用することは、`FilterSet`の自動フィルター生成による、ビルトインされた解決方法です。
これを手動で行うためには、単純に追加するだけです。

```python
class ProductFilter(django_filters.FilterSet):
    uncategorized = django_filters.BooleanFilter(filed_name="category", lookup_expr="isnull")
```

> **🖊 注意事項**
> フィルタークラスは入力値を検証することを忘れないでください。
> ここでは、モデルフィールドのベースにある型は関係ありません。

`exclude`パラメーターを使用して、ロジックを反転できます。

```python
class ProductFilter(django_filters.FilterSet):
    has_category = django_filters.BooleanFilter(filed_name="category", lookup="isnull", exclude=True)
```

#### 解決策2: ChoiceFilterのnull選択を使用する

`ChoiceFilter`を使用している場合、`null_label`パラメーターを有効にすることで、null値でフィルタすることもできます。
より詳細は[ドキュメント](https://django-filter.readthedocs.io/en/stable/ref/filters.html#choice-filter)の`ChoiceFilter`を参照してください。

```python
class ProductFilter(django_filters.FilterSet):
    category = django_filters.ModelChoiceFilter(
        field_name="category",
        lookup_expr="isnull",
        null_label="Uncategorized",
        queryset=Category.objects.all(),
    )
```

#### 解決策4: MultiValueFiledを組み合わせる

代わりの方法は、null値を処理するために、`BooleanField`内に手動で追加して、Djangoの`MultiValueField`を使用することです。
概念実証は、<https://github.com/carltongibson/django-filter/issues/446>にあります。

### 空の文字列でフィルタリングする

空の値（空の文字列）は、スキップされたフィルターとして解釈されるため、空の文字列でフィルタリングすることは現在不可能です。

```text
GET http://localhost/api/my-model?myfield=
```

> 空の文字列がモデルのフィールドに登録されないように、空の文字列を設定できないモデルフィールドを実装するか、モデルの`clean`メソッドでフィールドの値が空の文字列担っていないか検証するべき。

#### 解決策1: マジックバリュー

具体的にマジックバリューを確認するために、フィルタークラスの`filter()`メソッドを上書きできます。
これは、`ChoiceFilter`のnull値を処理と同様です。

```text
GET http://localhost/api/my-model?myfield=EMPTY
```

```python
class MyCharFilter(filters.CharFilter):
    empty_value = "EMPTY"

    def filter(self, qs, value);
        if value != self.empty_value:
            return super().filter(qs, value)
        qs = self.get_method(qs)(**{"%s__%s" % (self.field_name, self.lookup_expr): ""})
        return qs.distinct() if self.distinct else qs
```

#### 解決策2: 空文字列フィルター

`isnull`フィルターと同じ振る舞いをを見せる空文字列フィルターを作成できます。

```text
GET http://localhost/api/my-model?myfield__isempty=false
```

```python
from django.core.validators import EMPTY_VALUES


class EmptyStringFilter(filters.BooleanFilter):
    def filter(self, qs, value):
        if value is EMPTY_VALUES:
            return qs
        exclude = self.exclude ^ (value is False)
        method = qs.exclude if exclude else qs.filter
        return method(**{self.field_name: ""})


class MyFilterSet(filters.FilterSet):
    myfield_isempty = EmptyStringFilter(field_name="myfield")

    class Meta:
        model = MyModel
        fields = []
```

## 相対時間でフィルタリングする

モデルにタイムスタンプフィールドが与えられた場合、相対時間に基づいてフィルターすることが便利かもしれません。
例えば、もしかしたら、過去*n*時間からデータを取得したいかもしれません。
これは、カスタムメソッドを呼び出す`NumberFilter`を使用して達成できます。

```python
from django.utils import timezone
from datetime import timedelta


class DataModel(models.Model):
    timestamp = models.DateTimeField()


class DataFilter(django_filters.FilterSet):
    hours = django_filters.NumberFilter(
        field_name="timestamp",
        method="get_past_n_hours",
        label="Past n hours",
    )

    def get_past_n_hours(self, queryset, field_name, value):
        time_threshold = timezone.now() - timedelta(hours=int(value))
        return queryset.filter(timestamp__gte=time_threshold)
```

## デフォルトとしてinitial値を使用する（非推奨）

django-filterのプレリリースの1.0バーションにおいて、値が提出されなかったとき、フィルターフィールドの`initial`値は、デフォルトとして使用されました。
この振る舞いは、公式にサポートされていなかったため、削除されました。

> **⚠ 注意事項**
> 使用性に悪影響を与えるため、下を実装することは推奨されません。
> Djangoフォームは、次の理由でこの振る舞いを提供していません。
>
> - デフォルトとして`initial`値を使用することは、Djangoフォームの振る舞いと一貫性がありません。
> - デフォルト値は、ユーザーが空の値でフィルタリングすることをできなくします。
> - デフォルト値は、ユーザーがそのフィルターをスキップできなくします。

しかし、デフォルトが必要な場合、次はプレリリースの1.0バージョンの振る舞いを模倣します。

```python
class BaseFilterSet(django_filters.FilterSet):
    def __init__(self, data=None, *args, **kwargs):
        # もしFilterSetがバインドされている場合、デフォルトとしてinitial値を使用します。
        if data is not None:
            # QueryDictの可変コピーを得ます。
            data = data.copy()
            for name, field in self.base_filters.items():
                initial = field.extra.get("initial")
                # フィルターパラメーターは、指定されていないか空です、よってデフォルトとしてinitialを使用します。
                if not data.get(name) and initial:
                    data[name] = initial
        super().__init__(data, *args, **kwargs)
```

## フィルターにモデルフィールドのhelp_textを追加する

モデルフィールドの`help_text`は、デフォルトでフィルターによって使用されません。
それは、単純な`FilterSet`基本クラスを使用して、追加することができます。

```python
class HelpfulFilterSet(django_filters.FilterSet):
    @classmethod
    def filter_for_field(cls, field, name, lookup_expr):
        expanded_filter = super(HelpfulFilterSet, cls).filter_for_field(field, name, lookup_expr)
        expanded_filter.extra["help_text"] = field.help_text
        return filter
```
