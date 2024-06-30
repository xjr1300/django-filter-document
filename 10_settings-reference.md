# Settingsリファレンス

<https://django-filter.readthedocs.io/en/stable/ref/settings.html>

- [Settingsリファレンス](#settingsリファレンス)

ここに、django-filtersとそれらのデフォルト値のすべての利用可能な設定のリストを示します。
すべての設定は、`FILTERS_`接頭辞を付けられていまっすが、これは少し冗長で、これらの設定を簡単に識別するために役立ちます。

## FILTERS_DEFAULT_LOOKUP_EXPR

デフォルトは`"exact"`です。
noneで定義されたときに、生成されるルックアップ表現のデフォルトを設定します。

## FILTERS_EMPTY_CHOICE_LABEL

デフォルトは`"---------"`です。

`ChoiceFilter.empty_label`のデフォルト値を設定します。
これを`None`に設定することで空の選択肢を向こうにできます。

## FILTERS_NULL_CHOICE_LABEL

デフォルトは`None`です。
`ChoiceFilter.null_label`のデフォルト値を設定します。
非`None`値を設定することで、null選択を有効にできます。

## FILTERS_NULL_CHOICE_VALUE

デフォルトは`"null"`です。
`ChoiceFilter.null_value`のデフォルト値を設定します。
デフォルトの`"null"`文字列が実際の選択肢と衝突する場合、この値を変更できます。

## FILTERS_DISABLE_HELP_TEXT

デフォルトは`False`です。

いくつかのフィルターは情報的な`help_text`を提供します。
例えば、CSVを基盤とするフィルター（`filters.BaseCSVFilter`）は、"複数の値はカンマによって分離されます。"をユーザーに伝えます。

レンダリングされたフォームの出力からテキストを削除するために、すべてのフィルターに対して`help_text`を無効化するために、これを`True`に設定できます。

## FILTERS_VERBOSE_LOOKUPS

> **🖊 注意事項**
> これは応用の設定と考えられ、目的が変更されます。

デフォルトは次です。

```python
# django_filters.conf.DEFAULTSを参照してください。
'VERBOSE_LOOKUPS': {
    'exact': _(''),
    'iexact': _(''),
    'contains': _('contains'),
    'icontains': _('contains'),
    ...
}
```

この設定は生成されたフィルターのラベルの詳細な出力を制御します。
"lt"や"contained_by"のような式部分の代わりに、詳細なラベルには"is less than"や"is contained by"が含まれます。
この設定は、偽の値に設定することで詳細な出力を無効にできます。

また、この設定はコールバックを受け入れます。
コールバックは引数を必要とせず、辞書を返す必要があります。
これは、設定の全セットをコピーすることなく、デフォルトの用語を拡張またはオーバーライドするために便利です。
例えば、"exact"ルックアップのための詳細な出力を追加できます。

```python
# settings.py
def FILTERS_VERBOSE_LOOKUPS():
    from django_filters.conf import DEFAULTS

    verbose_lookups = DEFAULTS['VERBOSE_LOOKUPS'].copy()
    verbose_lookups.update({
        'exact': 'is equal to',
    })
    return verbose_lookups
```
