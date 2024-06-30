# フィルターリファレンス

<https://django-filter.readthedocs.io/en/stable/ref/filters.html>

- [フィルターリファレンス](#フィルターリファレンス)
  - [主要な引数](#主要な引数)
    - [field\_name](#field_name)
    - [lookup\_expr](#lookup_expr)
  - [キーワード引数](#キーワード引数)
    - [label](#label)
    - [method](#method)
    - [distinct](#distinct)
    - [exclude](#exclude)
    - [required](#required)
    - [\*\*kwargs](#kwargs)
      - [widget](#widget)
  - [ModelChoiceFilterとModelMultipleChoiceFilter引数](#modelchoicefilterとmodelmultiplechoicefilter引数)
    - [queryset](#queryset)
    - [to\_field\_name](#to_field_name)
  - [フィルター](#フィルター)
    - [CharFilter](#charfilter)
    - [UUIDFilter](#uuidfilter)
    - [BooleanFilter](#booleanfilter)
    - [ChoiceFilter](#choicefilter)
    - [TypedChoiceFilter](#typedchoicefilter)
    - [MultipleChoiceFilter](#multiplechoicefilter)
    - [TypedMultipleChoiceFilter](#typedmultiplechoicefilter)
    - [DateFilter](#datefilter)
    - [TimeFilter](#timefilter)
    - [DateTimeFilter](#datetimefilter)
    - [IsoDateTimeFilter](#isodatetimefilter)
    - [DurationFilter](#durationfilter)
    - [ModelChoiceFilter](#modelchoicefilter)
    - [ModelMUltipleChoiceFilter](#modelmultiplechoicefilter)
    - [NumberFilter](#numberfilter)
      - [NumberFilter.get\_max\_validator()](#numberfilterget_max_validator)
    - [NumericRangeFilter](#numericrangefilter)
    - [RangeFilter](#rangefilter)
    - [DateRangeFilter](#daterangefilter)
    - [DateFromToRangeFilter](#datefromtorangefilter)
    - [DateTimeFromToRangeFilter](#datetimefromtorangefilter)
    - [IsoDateTimeFromToRangeFilter](#isodatetimefromtorangefilter)
    - [TimeRangeFilter](#timerangefilter)
    - [AllValuesFilter](#allvaluesfilter)
    - [AllValuesMultipleFilter](#allvaluesmultiplefilter)
    - [LookupChoiceFilter](#lookupchoicefilter)
    - [BAseINFilter](#baseinfilter)
    - [BaseRangeFilter](#baserangefilter)
    - [OrderingFilter](#orderingfilter)
    - [カスタムフィルターの選択肢の追加](#カスタムフィルターの選択肢の追加)

これは、フィルターとそれらの引数のリストのリファレンスドキュメントです。

## 主要な引数

次は、すべてのフィルターに適用される主要な引数です。
それらは、ORMの`.filter()`呼び出しの左辺の完全な[ルックアップ表現](https://docs.djangoproject.com/en/stable/ref/models/lookups/#module-django.db.models.lookups)を構築するために結合されます。

### field_name

フィルターされる対象のモデルフィールドの名前です。
この引数が提供されていない場合、そのデフォルトは、`FilterSet`区明日のフィルターの属性名をデフォルトにします。
フィールド名は、ORMのルックアップセパレーター(`__`)を使用して関連部分を結合することで、リレーションシップを横断できます。
例えば、製品の`manufacturer__name`です。

### lookup_expr

フィルター呼び出しで実行される[フィールドルックアップ](https://docs.djangoproject.com/en/stable/ref/models/querysets/#field-lookups)です。
デフォルトは、`exact`です。
`lookup_expr`は、表現部分がORMルックアップセパレーター(`__`)によって結合されている場合、変換を含むことができます。
例えば、その年部分を`year__gt`によって決定されるフィルターがあります。

## キーワード引数

次は、すべてのフィルターの振る舞いを修正するために使用できるオプションの引数です。

### label

フォームフィールドのラベル引数に類似した、HTMLに表示されるラベルです。
ラベルが提供されていない場合、冗長なラベルが、フィールドの`field_name`と`lookup_expr`の部分に基づいて生成されます。
[FILTERS_VERBOSE_LOOKUP](https://django-filter.readthedocs.io/en/stable/ref/settings.html#verbose-lookups-setting)を参照してください。

### method

クエリセットを処理する方法を伝えるオプションの引数です。
それは、呼び出し可能オブジェクトまたは、`FilterSet`のメソッドの名前のどちらかを受け付けます。
呼び出し可能オブジェクトは`QuerySet`、それをフィルタするためにモデルフィールドの名前、そして、フィルタするために使用するう値を受け取ります。
それは、フィルターされた`QuerySet`を返します。

値が`Filter.field`によって検証されるため、そのままの値の変換と空の値の確認は必要ないことに注意してください。

```python
class F(FilterSet):
    """発行されたかいないかによって本をフィルターします"""
    published = BooleanFilter(field_name="published_on", method="filter_published")

    def filter_published(self, queryset, name, value):
        # 完全なルックアップ表現を構築します。
        lookup = "__".join([name, "isnull"])
        return queryset.filter(**{lookup: False})
        # 代わりに、オプションでルックアップをハードコードできます。例えば、
        # return queryset.filter(published_on__isnull=False)

    class Meta:
        model = Book
        fields = ["published"]


# 呼び出し可能オブジェクトは、クラススコープの外で定義することもできます。
def filter_not_empty(queryset, name, value):
    lookup = "__".join([name, "isnull"])
    return queryset.filter(**{lookup: False})


class F(FilterSet):
    """発行されたかいないかによって本をフィルターします"""
    published = BooleanFilter(field_name="published_on", method=filter_not_empty)

    class Meta:
        model = Book
        fields = ["published"]
```

### distinct

フィルターがクエリセットで`distinct`を使用するかどうかを指定するブーリアンです。
このオプションは、リレーションシップ間のフィルターを使用するときに、重複した結果を削除するために使用されます。

### exclude

フィルターがクエリセットに`filter`または`exclude`を使用するかを指定するブーリアンです。
デフォルトは`False`です。

### required

フィルターが要求されるかどうかを指定するブーリアンです。
デフォルトは`False`です。

### **kwargs

任意のキーワード引数は、フィルターの`extra`パラメーターとして保存されます。
それらは、フォームフィールドに付随して提供され、`choice`のような引数を提供するために使用されます。
次の通りいくつかフィールドに関連した引数があります。

#### widget

Djangoのフォームのウィジェットクラスで、`Filter`を表現します。
Djangoに含まれている使用可能なウィジェットに加えて、`django-filter`が提供する便利なウィジェットが追加されています。

- [LinkWidget](https://django-filter.readthedocs.io/en/stable/ref/widgets.html#link-widget)
  これは、一連したリンクのように、Djangoアドミンがする方法と似たような方法でオプションを表示します。
  選択されたオプションのリンクは、`class="selected"`を持ちます。
- [BooleanWidget](https://django-filter.readthedocs.io/en/stable/ref/widgets.html#boolean-widget)
  このウィジェットはその入力をPythonの`True/False`値に変換します。
  それはすべてのバリエーションの`True`と`False`をPythonの内部の値に変換します。
- [CSVWidget](https://django-filter.readthedocs.io/en/stable/ref/widgets.html#csv-widget)
  このウィゲットはカンマで区切られた値を期待して、それを文字列値のリストに変換します。
  それは型変換と同様に値のリストを処理するフィールドクラスを期待しています。
- [RangeWidget](https://django-filter.readthedocs.io/en/stable/ref/widgets.html#range-widget)
  このウィゲットは単独のフィールドを使用して、2つのフォームインプット要素を生成するために、`RangeFilter`で使用されます。

## ModelChoiceFilterとModelMultipleChoiceFilter引数

これらの引数は、明確に`ModelChoiceFilter`と`ModelMultipleChoiceFilter`のみに適用されます。

### queryset

`ModelChoiceFilter`と`ModelMultipleChoiceFilter`は、操作するクエリセットを要求して、それをキーワード引数として渡す必要があります。

### to_field_name

Djangoフィールドに転送される`to_field_name`を渡した場合、それはモデルの属性として、デフォルトの`get_filter_predicate`実装でも使用されます。

## フィルター

### CharFilter

このフィルターは、単純に文字にマッチして、デフォルトで`CharField`と`TextField`で使用されます。

### UUIDFilter

このフィルターは、UUID値にマッチして、デフォルトで`models.UUIDField`で使用されます。

### BooleanFilter

このフィルターは、`True`または`False`のどちらかのブーリアンとマッチして、デフォルトで`BooleanField`と`NullBooleanField`で使用されます。

### ChoiceFilter

このフィルターは、その`choices`引数内の値とマッチします。
`choices`は、フィルターが`FilterSet`で宣言されたときに明示的に渡されなければ成りません。
例えば・・・

```python
class User(models.Model):
    username = models.CharField(max_length=255)
    first_name = SubCharField(max_length=100)
    last_name = SubSubCharField(max_length=100)

    status = models.IntegerField(choices=STATUS_CHOICES, default=0)


STATUS_CHOICES= (
    (0, "Regular"),
    (1, "Manager"),
    (2, "Admin"),
)


class F(FilterSet):
    status = ChoiceFilter(choices=STATUS_CHOICES)
    class Meta:
        model = User
        fields = ["status"]
```

また、`ChoiceFilter`は、`None`値でフィルタリングしたときと同様に、フィルタリングしない`choice`を有効にする引数もあります。
それぞれのひきすうは、対応するグローバルな設定を持ちます([設定リファレンス](https://django-filter.readthedocs.io/en/stable/ref/settings.html))。

- `empty_label`
  選択項目をフィルターしないために使用する表示ラベルです。
  この引数を`None`に設定することで、選択を無効にできます。
  デフォルトは`FILTERS_EMPTY_CHOICE_LABEL`です。
- `null_label`
  `None`値でフィルターする選択項目に使用する表示ラベルです。
  この引数を`None`に設定することで、選択を無効にできます。
  デフォルトは`FILTERS_EMPTY_CHOICE_LABEL`です。
- `null_value`
  `None`値でフィルタリングを有効にしてマッチする特別な値をです。
  この値は`FILTERS_NULL_CHOICE_VALUE`がデフォルトで、空でない値が必要です（`""`、`None`、`[]`、`()`、`{}`以外）。

### TypedChoiceFilter

変換した値とマッチする能力を追加した`ChoiceFilter`と同じです。
これは、*coerce*パラメーターを使用してできます。
ユースケースの例は、マッチするブーリアンの選択肢を制限して、事前定義したいくつかの文字列のみをビューリアンフィルターの入力値として使用します。

```python
from distutils.util import strtobool  # distutilsは3.12で削除されるため非推奨
import django_filters


BOOLEAN_CHOICES = (
    ("false", "False"),
    ("true", "True"),
)


class YourFilterSet(django_filters.FilterSet):
    # ...
    flag = django_filters.TypedChoiceFilter(
        choices=BOOLEAN_CHOICES,
        coerce=strtobool,
    )
```

### MultipleChoiceFilter

ユーザーが複数の選択肢を選択でき、フィルターがデフォルトでこれらの選択肢をORを形成して項目をマッチする以外は`ChoiceFilter`と同じです。
`conjoined=True`引数をこのクラスに渡したとき、フィルターは選択肢のANDを形成します。

複数の選択肢は、例えば"?status=Regular&status=Admin"のように、さまざまな値を持つ同じキーを再利用することで、クエリ文字列内で表現されます。

通常、対他の関連では、これが要求されるため、`distinct`のデフォルトは`True`です。

応用利用: アプリケーションロジックに依存して、すべて選択または全く選択されなかったとき、フィルタリングは無駄になるかもしれません。
フィルタイング、特に*distinct*の呼び出しのオーバーヘッドを避けたい場合があります。

インスタンス化した後に*always_filter*を`False`に設定して、デフォルトの`is_noop`テストを有効にします。

アプリケーションで様々なテストを要求する場合、`is_noop`を上書きします。

### TypedMultipleChoiceFilter

`MultipleChoiceFilter`のように、しかし`TypedChoiceFilter`の用に`coerce`引数を追加で受け付けます。

### DateFilter

日付とマッチします。
デフォルトで`DateField`で使用されます。

### TimeFilter

時刻とマッチします。
デフォルトで`TimeField`で使用されます。

### DateTimeFilter

日時とマッチします。
デフォルトで`DateTimeField`で使用されます。

### IsoDateTimeFilter

APIでよく使用され、デフォルトでDjango REST Frameworkで使用される、ISO8601フォーマット日付でフィルタリングをサポートするために`IsoDateTimeFilter`を使用します。
使用例は次です。

```python
class F(FilterSet):
    """ISO8601書式の日付を使用して、発行された日付によって本をフィルターします"""
    published = IsoDateTimeFilter()

    class Meta:
        model = Book
        fields = ["published"]
```

### DurationFilter

間隔にマッチします。
デフォルトで`DurationField`で使用されます。

Pythonの`timedelta`に受け付けられる間隔のみですが、Djangoの"%d %H:%M:%S.%f"と、ISO8601書式間隔の両方をサポートするため、"P3DT10H22M"のような年、付き、そして週指定子はサポートしていません。

### ModelChoiceFilter

関連したモデルと機能する以外は、`ChoiceFilter`と同じで、デフォルトで`ForeignKey`で使用されます。

自動的にインスタンス化された場合、`ModelChoiceFilter`は、関連したフィールドのデフォルトの`QuerySet`を使用します。
もし手動でインスタンス化した場合、`queryset`キーワード引数を提供しなければなりません。

例を次に示します。

```python
class F(FilterSet):
    """著者によって本をフィルターします"""
    author = ModelChoiceFilter(queryset=Author.objects.all())

    class Meta:
        model = Book
        fields = ["author"]
```

また`queryset`引数は、呼び出しの振る舞いをサポートする。
呼び出し可能オブジェクトが渡されると、その唯一の引数として`Filterset.request`を呼び出します。
これは、`FilterSet.__init__`を上書きすることなしで、リクエストオブジェクトの属性によって、簡単にフィルターできるようにします。

> **🖊 注意事項**
> *request*オブジェクトが*None*であることを予期する必要があります。

```python
def departments(request):
    if request is None:
        return Department.objects.none()
    company = request.user.company
    return company.department_set.all()


class EmployeeFilter(django_filters.FilterSet):
    department = filters.ModelChoiceFilter(queryset=departments)
```

### ModelMUltipleChoiceFilter

関連するモデルと機能する以外は、`MultipleChoiceFilter`と同じで、デフォルトで`ManyToManyField`で使用されます。

`ModelChoiceFilter`と同様に、自動的にインスタンス化した場合、`ModelMultipleChoiceFilter`は関連したフィールドにデフォルトの`QuerySet`を使用します。
もし、マニュアルでインスタンス化した場合、`queryset`キーワード引数を提供しなければ成りません。
`ModelChoiceFilter`野湯王に、`queryset`引数は呼び出し可能の振る舞いを持ちます。

ルックアップにカスタムフィールド名を使用するために、`to_field_name`を使用できます。

```python
class FooFilter(BaseFilterSet):
    foo = django_filters.filters.ModelMultipleChoiceFilter(
        field_name="attr__uuid",
        to_field_name="uuid",
        queryset=Foo.objects.all(),
    )
```

もし、例えば注釈されたフィールドを追加するために、カスタムクエリセットを使用したい場合、これは次の通りできます。

```python
class MyMultipleChoiceFilter(django_filters.ModelMultipleChoiceFilter):
    def get_filter_predicate(self, v);
        return {"annotated_field": v.annotated_field}

    def filter(self, qs, value):
        if value:
            qs = qs.annotate_with_custom_field()
            qs = super().fiter(qs, value)
        return qs


foo = MyMultipleChoiceFilter(
    to_field_name="annotated_field",
    queryset=Model.objects.annotate_with_custom_field(),
)
```

`annotate_with_custom_field`メソッドは、カスタムクエリセットを介して定義され、それはモデルマネージャーとして使用されます。

```python
class CustomQuerySet(models.QuerySet):
    def annotate_with_custom_field(self):
        return self.annotate(
            custom_field=Case(
                When(foo__isnull=False,
                     then=F("foo__uuid")),
                When(bar__isnull=False,
                     then=F("bar__uuid")),
                default=None,
            ),
        )


class MyModel(models.Model):
    objects = CustomQuerySet.as_manager()_
```

### NumberFilter

数値的な値に基づいてフィルターして、デフォルトで`IntegerField`、`FloatField`、`DecimalField`で使用されます。

#### NumberFilter.get_max_validator()

`field.validators`を追加された`MaxValueValidator`インスタンスを返します。
デフォルトで、`1e50`の制限値を使用します。
最大値の検証を無効にするために`None`を返します。

### NumericRangeFilter

値が2つの数値の間にある場合、または制限値が1つだけ指定されている場合は、最小より大きいか、最大より小さい値にフィルターします。
このフィルターは、`IntegerRangeField`、`BigIntegerRangeField`そして`FloatRangeField`（Django1.8から利用できる）を含む、Postgresの数値範囲フィールドと機能するように設計されています。
使用されるデフォルトのウィゲットは、`RangeField`です。

加えて、通常のフィールドルックアップは、`overlap`、`contains`そして`contained_by`を含むさまざまな封じ込めルックアップを利用できます。
詳細はDjangoの[ドキュメント](https://docs.djangoproject.com/en/stable/ref/contrib/postgres/fields/#querying-range-fields)を参照してください。

下限の値が提供された場合、フィルターはルックアップとして`startswith`、上限の値が提供された場合は`endswith`を自動的にデフォルトにします。

### RangeFilter

2つの数値の値の間、または、たった1つの制限値が与えられている場合は最小よりも大きいか、最大よりも小さい値にフィルターします。

```python
class F(FilterSet):
    """価格で本をフィルターします"""
    price = RangeFilter()

    class Meta:
        model = Book
        fields = ["price"]


qs = Book.objects.all().order_by("title")

# 範囲: 5ユーロから15ユーロの間の本
f = F({"price_min": "5", "price_max": "15"}, queryset=qs)

# 最小のみ: 本の価格が11ユーロよりたかい
f = F({"price_min": "11"}, queryset=qs)

# 最大のみ: 本の価格が19ユーロより安い
f = F({"price_max": "19"}, queryset=qs)
```

### DateRangeFilter

アドミンのチェンジリストの日付に似たフィルターで、日付フィールドを操作するための一般的な選択肢が多くあります。

### DateFromToRangeFilter

数値の代わりに日付を使用する以外は、`RangeFilter`と同じです。
それは、`DateField`で使用されます。
また、それは、`DateTimeField`で機能しますが、日付のみを考慮します。

`DateField`を使用した例を次に示します。

```python
class Comment(models.Model):
    date = models.DateField()
    time = models.TimeField()


class F(FilterSet):
    date = DateFromToRangeFilter()

    class Meta:
        model = Comment
        fields = ["date"]


# 範囲: 2016-01-01から2016-02-01の間に追加されてコメント
f = F({"date_after": "2016-01-01", "date_before": "2016-02-01"})

# 最小のみ: 2016-01-01より後に追加されたコメント
f = F({"date_after": "2016-01-01"})

# 最大のみ: 2016-02-01より前に追加されたコメント
f = F({"date_before": "2016-02-01"})
```

> **🖊 注意事項**
> DST移行日に発生する範囲をフィルタリングするとき、`DateFromToRangeFilter`は開始日時としてその日の最初の有効な時間と、終了日時としてその日の最後の有効な時間を使用します。
> ほとんどのアプリケーションでこれは大丈夫ですが、この動作をカスタマイズしたい場合は、`DateFromToRangeFilter`を拡張して、そのカスタムフィールドを作成してください。

> DSTとはサーマタイム制度を示し、日の出時刻が早まる時期に時計を1時間進めて、秋にい1時間戻すことで、日中の明るい時間を有効利用する。

> **⚠ 警告**
> Django 1.9より前を使用している場合、開始と終了の日付がそれぞれDST移行日の開始と終了日に一致する場合、`AmbiguousTimeError`または`NonExistentTimeError`が発生する可能性があります。
> これは、バージョン1.9より前はDSTの振る舞いをタイムゾーン付きの日時に変更できないことが理由で発生します。

`DateTimeField`フィールドを使用た例を次に示します。

```python
class Article(models.Model):
    published = models.DateTimeField()


class F(FilterSet):
    published = DateFromToRangeFilter()

    class Meta:
        model = Article
        fields = ["published"]


Article.objects.create(published="2016-01-01 8:00")
Article.objects.create(published="2016-01-20 10:00")
Article.objects.create(published="2016-02-10 12:00")


# 範囲: 2016-01-01から2016-02-01の間に発行された記事
f = F({"published_after": "2016-01-01", "published_before": "2016-02-01"})
assert len(f.qs) == 2

# 最小のみ: 2016-01-01より後に発行された記事
f = F({"published_after": "2016-01-01"})
assert len(f.qs) == 3

# 最大のみ: 2016-02-01より前に発行された記事
f = F({"published_before": "2016-02-01"})
assert len(f.qs) == 2
```

### DateTimeFromToRangeFilter

数値の代わりに日付書式値を使用することを除いて`RangeFilter`と同じです。
それは、`DateTimeField`で使用されます。

次に例を示します。

```python
class Article(models.Model):
    published = models.DateTimeField()


class F(FilterSet);
    published = DateTimeFromToRangeFilter()

    class Meta:
        model = Article
        fields = ["published"]


Article.objects.create(published="2016-01-01 8:00")
Article.objects.create(published="2016-01-01 9:30")
Article.objects.create(published="2016-01-02 8:00")

# 範囲: 2016-01-01の8:00から10:00の間に発行された記事
f = F({"published_after": "2016-01-01 8:00", "published_before": "2016-01-01 10:00"})
assert len(f.qs) == 2

# 最小のみ: 2016-01-01 8:00より後に発行された記事
f = F({"published_after": "2016-01-01 8:00"})
assert len(f.qs) == 3

# 最大のみ: 2016-01-01 10:00より前に発行された記事
f = F({"published_before": "2016-01-01 10:00"})
assert len(f.qs) == 2
```

### IsoDateTimeFromToRangeFilter

数値の代わりにISO8601で書式化された値を使用することを除いて`RangeFilter`と同じです。
それは`IsoDateTimeField`で使用されます。

次に例を示します。

```python
class Article(models.Model):
    published = django_filters.IsoDateTimeField()


class F(FilterSet):
    published = IsoDateTimeFromToRangeFilter()

    class Meta:
        model = Article
        fields = ["published"]


Article.objects.create(published="2016-01-01T08:00:00+01:00")
Article.objects.create(published="2016-01-01T09:30:00+01:00")
Article.objects.create(published="2016-01-02T08:30:00+01:00")

# 範囲: 2016-01-01の8:00から10:00の間に変更された記事
f = F({"published_after": "2016-01-01T08:00:00+01:00", "published_before": "2016-01-01T10:00:00+01:00"})
assert len(f.qs) == 2

# 最小のみ: 2016-01-01 8:00より後に発行された記事
f = F({"published_after": "2016-01-01T08:00:00+01:00"})
assert len(f.qs) == 3

# 最大のみ: 2016-01-01 10:00より前に発行された記事
f = F({"published_before": "2016-01-01T10:00:00+01:00"})
assert len(f.qs) == 2
```

### TimeRangeFilter

数値の代わりに時刻書式値を使用することを除いて`RangeFilter`と同じです。
それは`TimeField`で使用されます。

次に例を示します。

```python
class Comment(models.Model):
    date = models.DateField()
    time = models.TimeField()


class F(FilterSet):
    time = TimeRangeFilter()

    class Meta:
        model = Comment
        fields = ["time"]


# 範囲: 8:00から10:00の間に追加されたコメント
f = F({"time_after": "08:00", "time_before": "10:00"})

# 最小のみ: 8:00より後に追加されたコメント
f = F({"time_after": "08:00"})

# 最大のみ: 10:00より前に追加されたコメント
f = F({"time_before": "10:00"})
```

### AllValuesFilter

これは、選択肢がデータベースに現在ある値になっている`ChoiceFilter`です。
よって、データベースに5、7そして9の値を持つ与えられたフィールドがある場合、それらそれぞれが選択した落ちて表示されます。
これは、アドミンのデフォルトの振る舞いと似ています。

### AllValuesMultipleFilter

これは、選択肢がデータベースに現在ある値になっている`MultiChoiceFilter`です。
よって、データベースに5、7そして9の値を持つ与えられたフィールドがある場合、それらそれぞれが選択した落ちて表示されます。
これは、アドミンのデフォルトの振る舞いと似ています。

### LookupChoiceFilter

ユーザーがドリップダウンからルックアップ表現を選択できる結合されたフィルターです。

- `lookup_choices`は、複数の入力書式を受け取るオプションの引数で、最終的にルックアップドロップダウンで使用される選択肢として正規化されます。
  詳細は`.get_lookup_choices()`を参照してください
- `field_class`は、値を検証するために使用される内部のフォームフィールドクラスを設定できるようにするオプションの引数です。
  デフォルトは`forms.CharField`です。

例を次に示します。

```python
price = django_filters.LookupChoiceFilter(
    field_class=forms.DecimalField,
    lookup_choices=[
        ("exact", "Equals"),
        ("gt", "Greater than"),
        ("lt", "Less than"),
    ]
)
```

### BAseINFilter

これは、`IN`ルックアップフィルターを作成するために使用される基本クラスです。
それは、このフィルタークラスが、他のフィルタークラスと接続するために使用されることを期待しているため、このクラスのみで、カンマで区切られた入力値を検証します。
2つ目のフィルターは、個々の値を検証するために使用されます。

例を次に示します。

```python
class NumberInFilter(BaseInFiler, NumberFilter):
    pass


class F(FilterSet):
    id__in = NumberInFilter(field_name="id", lookup_expr="in")

    class Meta:
        model = User


User.objects.create(username="alex")
User.objects.create(username="jacob")
User.objects.create(username="aaron")
User.objects.create(username="aaron")

# In: IDに1または3を持つユーザー
assert len(f.qs) == 2
```

### BaseRangeFilter

これは、`RANGE`ルックアップフィルターを作成するために使用される基本クラスです。
その振る舞いは、カンマで区切られた2つの値のみを予期することを除いて、`BaseInFilter`と同じです。

次に例を示します。

```python
class NumberRangeFilter(BaseRangeFilter, NumberFilter):
    pass


class F(FilterSet):
    id__range = NumberRangeFilter(field_name="id", lookup_expr="range")

    class Meta:
        model = User


User.objects.create(username="alex")
User.objects.create(username="jacob")
User.objects.create(username="aaron")
User.objects.create(username="carl")

# 範囲: IDが1から3のユーザー
f ＝ F({"id__range": "1,3"})
assert len(f.qs) == 3
```

### OrderingFilter

クエリセットの並び替えを可能sにします。
`ChoiceFilter`の拡張によって、それは並び替えの選択肢を構築するために使用される2つの追加引数を受け付けます。

- `fields`は、`{モデルフィールド名 name: 引数名}`のマッピングです。
  その引数名は選択肢内に公開され、`order_by()`呼び出し内で使用されるフィールド名をマスクまたはエイリアスします。
  `choices`フィールドと同様に、`fields`は並び替えを保持する"2つのタプルのリスト"構文を受け付けます。
  `fields`は文字列のイテラブルになることもできます。
  この場合、フィールド名は単純に、公開された引数名と同じになります。
- `field_labels`は、対応する引数のために表示されるラベルをカスタマイズできるようにします。
  それは、`{フィールド名: 人が読みやすいラベル}`のマッピングを受け付けます。
  キーはフィールド名で、公開された引数名でないことを留意してください。

```python
class UserFilter(FilterSet):
    account = CharFilter(field_name="username")
    status = NumberFilter(field_name="status")

    o = OrderingFilter(
        # 並び替えを保持するタプルのマッピング
        fields=(
            ("username", "account"),
            ("first_name", "first_name"),
            ("last_name", "last_name"),
        ),
        # 並び替えを保持するために使用されないラベル
        field_labels={
            "username": "User account",
        }
    )

    class Meta:
        model = User
        fields = ["first_name", "last_name"]


>>> UserFilter().filters['o'].field.choices
[
    ('account', 'User account'),
    ('-account', 'User account (descending)'),
    ('first_name', 'First name'),
    ('-first_name', 'First name (descending)'),
    ('last_name', 'Last name'),
    ('-last_name', 'Last name (descending)'),
]
```

加えて、公開された選択肢の明示的な制御が必要な場合、独自の`choices`を提供できます。
例えば、降順ソートの選択肢を無効にしたい場合ときなどです。

```python
class UserFilter(FilterSet):
    account = CharFilter(field_name="username")
    status = NumberFilter(field_name="status")

    o = OrderingFilter(
        choices=(
            ("account:", "Account"),
        ),
        fields={
          "username": "account",
        }
    )
```

また、このフィルタはCSVを基盤としており、複数の並び替え引数を受け付けます。
デフォルトのSELECTウィジェットは、この使用を有効にできませんが、APIにとっては便利です。
`SelectMultiple`ウィジェットは、並び替えの選択を保持できないため、互換性がありません。

### カスタムフィルターの選択肢の追加

非モデルフィールドでソートしたい場合、`OrderingFilter`サブクラスのカスタム処理を追加することができます。
例えば、計算された"関連性"因子でソートしたい場合、次のようにする必要があります。

```python
class CustomOrderingFilter(django_filters.OrderingFilter):
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.extra["choices"] += [
          ("relevance", "Relevance"),
          ("-relevance", "Relevance (descending)"),
        ]

    def filter(self, qs, value):
        # OrderingFilterはCSVを基盤とするため、`value`はリストです。
        if any(v in ["relevance", "-relevance"] for v in value):
            # 関連性でクエリセットを並び替えします。
            return ...
        return super().filter(qs, value)
```
