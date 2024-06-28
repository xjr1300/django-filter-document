# django-filter

Django-filterは、よりありふれたように見えるコードを緩和するために、ジェネリック、再利用できるアプリケーションにします。
特に、Django-filterは、ユーザーがモデルフィールドに基づいてクエリセットをフィルタダウンして、これをするためにフォームにそれらを表示できるようにします。

- [django-filter](#django-filter)
  - [インストール](#インストール)
  - [要求事項](#要求事項)

## インストール

<https://django-filter.readthedocs.io/en/stable/>

Django-filterは、`pip`のようなツールを使用して、PyPIからインストールされます。

```sh
pip install django-filter
```

そして、`INSTALLED_APPS`に`django_filters`を追加します。

```python
INSTALLED_APPS = [
    # ...
    "django_filters",
]
```

## 要求事項

Django-filterは、[Django](https://www.djangoproject.com/download/#supported-versions)の現在のバージョンと、テストされたすべてのバージョンのPythonと同様に、最新バージョンのDjango REST Framework([DRF](http://www.django-rest-framework.org/))を要求します。
