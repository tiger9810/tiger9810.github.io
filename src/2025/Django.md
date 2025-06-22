<!-- # 腕立て伏せ100回を30日行うとどのような変化が起きたか？ -->
# DjangoでのWebサイト構築

<!--
date = "2025-06-22"
-->

## 使ったもの

- Claude AI

## やりたいこと

pythonのDjangoを使用してWebサイトを構築する

## なぜやるのか

プログラミング学習

## Djangoについて

### 学習の流れ

[Django学習のロードマップ](https://qiita.com/nossey/items/03e114a7b728a8b2ae55)のロードマップに従う

[クイックインストールガイド](https://docs.djangoproject.com/ja/2.0/intro/install/)を参考にDjangoをインストール

### memo

バージョン確認
``` zsh
python -m django --version
```

mysiteという[プロジェクトを作成](https://docs.djangoproject.com/ja/2.0/intro/tutorial01/#creating-a-project)
```zsh
django-admin startproject mysite
```

作成したprojectの構成
``` python
mysite/             # プロジェクト名、自由に変更できる。
    manage.py       # コマンドラインユーティリティ
    mysite/.        # このプロジェクトに関連するPythonファイル（モジュール）をまとめて整理するためのフォルダ
        __init__.py # このディレクトリが Python パッケージであることを Python に知らせるための空のファイル
        settings.py # Django プロジェクトの設定ファイル
        urls.py     # Django プロジェクトの URL 宣言、いうなれば Django サイトにおける「目次」に相当
        wsgi.py     # プロジェクトをサーブするためのWSGI互換Webサーバーとのエントリーポイント
```

[開発用サーバーの起動](https://docs.djangoproject.com/ja/2.0/intro/tutorial01/#the-development-server)
```zsh
python manage.py runserver
```


---
##### コマンドラインユーティリティ
コマンドラインユーティリティとは下記のサーバー起動のコマンドのようなサブコマンドを持つコマンドのまとまりのこと
```
python manage.py runserver 8080
└────┘ └───────┘ └──────┘ └──┘
Python  管理スクリプト サーバー起動 ポート番号
```

---
<details><summary>履歴</summary>


</details>