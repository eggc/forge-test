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

その後、Github にログインし適当なリポジトリを作り、チェックアウトします。今回は [forge-test](https://github.com/eggc/forge-test) というリポジトリを作ってみることにします。もちろん、既存のリポジトリがあるなら git clone でチェックアウトしても問題ありません。`M-x magit-status` で forge-test を開いた後に `M-x forge-pull` を実行します。初めて利用するときには下記のメッセージが表示されます。

<img src="https://github.com/eggc/forge-test/blob/master/img/github-token-not-found.png?raw=true">

`y` を押すと Github API を使うための初期設定を行います。指示通りにユーザ名、パスワード、二段階認証のワンタイムパスワードを与えると Github API token を生成できるはずです。

<img src="https://github.com/eggc/forge-test/blob/master/img/username.png?raw=true">
<img src="https://github.com/eggc/forge-test/blob/master/img/password.png?raw=true">
<img src="https://github.com/eggc/forge-test/blob/master/img/2factor.png?raw=true">

## Github API token がうまく作れない場合

Github API token がうまく作れた場合、このセクションはスキップしてください。私の場合は、下記のメッセージが表示されて失敗しました。

> ghub--confirm-create-token: HTTP Error: 401, "Unauthorized", "/authorizations", ((message . "Must specify two-factor authentication OTP code.") (documentation_url . "https://developer.github.com/v3/auth#working-with-two-factor-authentication"))

原因切り分けのため [Github API](https://developer.github.com/v3/auth/#working-with-two-factor-authentication) を読んでターミナルで実験します。下記のコマンドは Github API token を自動で生成するときのパラメータとほとんど同じパラメータを使うリクエストです。

```
curl -D - --user eggc --header 'x-github-otp: 999999' https://api.github.com/user
```

github アカウントのパスワードを要求されるので、入力します。なお 999999 は適切なワンタイムパスワードと置き換えてください。もしこれに失敗するなら、ユーザ名、パスワード、ワンタイムパスワードの組み合わせが間違っているということになります。私の場合、上記のコマンドは正しく取得できたので github 側に問題はなかったようです。

内部的には [ghub](https://github.com/magit/ghub) というサブモジュールが使われており、こちらに問題があるようです。しかたがないので Github のトークン自動作成を諦めて、手動でトークンを作ります。[Github の設定](https://github.com/settings/tokens/new) を開きます。Note はとりあえず emacs-forge としておきます。権限 repo user read:org にチェックを入れ、作成します。アクセストークンをコピーしてホームディレクトリにファイル .authinfo を作成してみます。[Ghubの設定](https://magit.vc/manual/ghub/How-Ghub-uses-Auth_002dSource.html#How-Ghub-uses-Auth_002dSource) と [authinfoの説明](https://www.emacswiki.org/emacs/GnusAuthinfo) を下の一行を .authinfo に書き込みました。

```
machine api.github.com login eggc^forge password GITHUB_API_ACCESS_TOKEN
```

GITHUB_API_ACCESS_TOKEN には各自の適切な値と置き換えます。保存できたら、改めて M-x forge-pull を実行します。これで無事トークンが使われるようになりました。

# forge を使ってみる

forge は magit のサブモジュールなので、基本的に magit-status バッファを開いてから操作します。 `M-x magit-status` を実行してください。

## issue の作成

まずは issue を作成してみます。forge が正しくロードされ、初期設定を終えていれば `' c i` で新しい issue を作成するためのバッファが開かれます。これをたとえば下のように編集します。

<img src="https://github.com/eggc/forge-test/blob/master/img/new-issue.png?raw=true">

最後に `C-c C-c` で Github に投稿します。もし投稿したくない場合は `C-c C-k` で取り消す事ができます。取り消した内容は記憶されていて、再度 issue を作ろうとしたときに復元するかどうか質問されます。`r` で復元し `d` で破棄します。なお、github のテンプレート機能にも対応しています。具体的には、リポジトリに .github/ISSUE_TEMPLATE ファイルがあった場合、それが issue 作成バッファが新規作成されたときの内容になります。
