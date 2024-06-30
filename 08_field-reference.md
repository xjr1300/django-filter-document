# フィールドリファレンス

<https://django-filter.readthedocs.io/en/stable/ref/fields.html>

- [フィールドリファレンス](#フィールドリファレンス)

## IsoDateTimeField

`django.forms.DateTimeField`の拡張で、既存の形式に加えて、ISO8601書式化日付をパースできるようにします。

`ISO_8601`をフォーマットの定数として、クラスレベル属性に定義します。

`input_formats = [ISO_8601]`を設定することは、、デフォルトで`IsoDateTimeField`はISO8601書式化日付のみを解析することを意味します。

ISO8601書式を指定するために`ISO_8610`クラスレベル属性を使用して、[DateTimeFieldドキュメント](https://docs.djangoproject.com/en/stable/ref/forms/fields/#django.forms.DateTimeField.input_formats)に従って、`input_formats`を要求する書式のリストに設定できます。

```python
f = IsoDateTimeField()
f.input_formats = [IsoDateTimeField.ISO_8601] + DateTimeField.input_formats
```
