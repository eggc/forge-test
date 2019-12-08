# はじめに

Emacs では git の操作をサポートする [magit](https://github.com/magit/magit) という有名なパッケージがあります。しかし magit は Github が提供するプルリクエストや issue の操作はサポートしていません。そこで magit のサブモジュールとして作られたのが [forge](https://github.com/magit/forge) です。このパッケージをインストールすれば、github でプルリクエストを出したり、issue を眺めたりすることが可能になります。本記事では forge のインストールと操作方法、実際に使った感想を紹介したいと思います。

# インストールと初期設定

`M-x list-packages` を実行し forge をインストールします。執筆時点の最新版は 20191204.1349 です。パッケージ一覧に forge が無い場合はおそらくリポジトリに melpa が追加されていないので[このあたり](https://emacs-jp.github.io/packages/package-management/package-el)を参考に設定して、リトライします。dependency に magit が含まれているので、自動で magit もインストールされるはずです。インストール後に magit を有効にするには、下記のコードを init.el に追加します。

```init.el
(with-eval-after-load 'magit
  (require 'forge))
```

use-package を使っている場合は下記のコードにします。

```init.el
(use-package forge
  :after magit)
```

その後、Github にログインし適当なリポジトリを作り、チェックアウトします。
今回は forge-test というリポジトリを作ってみることにします。
