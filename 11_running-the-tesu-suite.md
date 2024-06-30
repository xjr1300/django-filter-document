# テストスイートの実行

<https://django-filter.readthedocs.io/en/stable/dev/tests.html>

- [テストスイートの実行](#テストスイートの実行)
  - [リポジトリの複製](#リポジトリの複製)
  - [仮想環境のセットアップ](#仮想環境のセットアップ)
  - [テストランナーの実行](#テストランナーの実行)
  - [サポートするすべてのバージョンをテストする](#サポートするすべてのバージョンをテストする)
  - [整理](#整理)

django-filterのテストを実行する最も簡単な方法は、ソースコードをチェックアウトして、テスト依存関係をインストールできる仮想環境を作成することです。
django-filterは環境を設定するためにカスタムテストランナーを使用しているため、テストスイートをセットアップして実行するためのラッパースクリプトを利用可能です。

> **🖊 注意事項**
> 次は、virtualenvとgitがインストールされていることを前提としています。

## リポジトリの複製

次のコマンドを使用してソースコードを取得します。

```sh
git clone https://github.com/carltongibson/django-filter.git
```

django-filterディレクトリに切り替えます。

```sh
cd django-filter
```

## 仮想環境のセットアップ

テストスイートを実行するための新しい仮想環境を作成します。

```sh
virtualenv venv
```

その後、仮想環境をアクティブにして、テスト要件をインストールします。

```sh
source venv/bin/activate
pip install -r requirements/test.txt
```

## テストランナーの実行

らんなースクリプトを使用してテストを実行します。

```sh
python runtests.py
```

## サポートするすべてのバージョンをテストする

優れたテストツールであるtoxを使用して、すべてのサポートされているPythonとDjangoのバージョンに対してテストを実行できます。toxをインストールして、単に次のように実行します。

```sh
pip install tox
tox
```

## 整理

`isort`ユーティリティは、モジュールのインポートを維持するために使用されます。
適切なtox環境でモジュールのインポートをテストするか、isortを直接使用できます。

```sh
pip install tox
tox -e isort

# or

pip install isort
isort --check --diff django_filters tests
```

インポートをソートするために、単に`--check-only`オプションを削除します。

```sh
isort --recursive django_filters tests
```
