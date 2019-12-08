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

forge は magit のサブモジュールなので、基本的に magit-status バッファを開いてから操作します。forge を使う上で、ブランチを作ったり、プッシュしたりすることがありますが、それらは magit の操作なので説明を省きます。

## issue の作成

まずは issue を作成してみます。forge が正しくロードされ、初期設定を終えていれば `' c i` で新しい issue を作成するためのバッファが開かれます。これをたとえば下のように編集します。

<img src="https://github.com/eggc/forge-test/blob/master/img/new-issue.png?raw=true">

最後に `C-c C-c` で Github に投稿します。もし投稿したくない場合は `C-c C-k` で取り消す事ができます。取り消した内容は記憶されていて、再度 issue を作ろうとしたときに復元するかどうか質問されます。`r` で復元し `d` で破棄します。なお、github のテンプレート機能にも対応しています。具体的には、リポジトリに .github/ISSUE_TEMPLATE ファイルがあった場合、それが issue 作成バッファが新規作成されたときの内容になります。

## issue の表示・編集

issue を作成したあとに magit-status に戻ってくると Issues というセクションが追加されており、ここに追加した issue が表示されるようになります。折り畳まれている場合はカーソルを当てて `TAB` で開閉します。

<img src="https://github.com/eggc/forge-test/blob/master/img/magit-status-issues.png?raw=true">

ここで、さっき作成した #1 の issue にカーソルをあてて `RET` を押すと issue の詳細を見ることができます。

<img src="https://github.com/eggc/forge-test/blob/master/img/show-issue.png?raw=true">

内容に不足があり、再度編集したい場合には `C-c C-e` を入力すると、編集する事ができます。カーソルが Title, State, Labels, Marks, Assignees に当たっている場合は、その項目が編集対象になります。

- Title: タイトルを編集します。
- State: open 状態なら close します。close 状態なら reopen します。y/n で入力を求められます。
- Labels: ラベルを入力します。事前にラベルの種類（デフォルトでは bug, enhancement など）を決めておく必要があります。カンマ区切りで入力すると、複数のラベルをセットすることができます。一個だけ入力した場合は追加ではなく、上書きになるので注意が必要です。ラベルの色は Github で設定した色と同じになります。
- Marks: マークを入力します。事前に forge-create-mark でマークを作っておく必要があります。マークはラベルと似た仕組みですが、Github の持っている機能ではなく、forge が独自に導入しているもので、他の人と共有しないラベルです。
- Assignees: issue の担当者を入力します。github アカウント名を入力します。

すべて操作した結果、下のようになりました。

<img src="https://github.com/eggc/forge-test/blob/master/img/show-issue-edited.png?raw=true">

forge はマイルストーンや、プロジェクトの機能には対応していないので、それらを編集したい場合は `C-c C-o` を入力し、 issue をブラウザで開きます。

## プルリクエストの作成

forge はプルリクエストに対応しており、当然 fork してからプルリクエストを作ることもできるのですが、一人で実験することは難しいので、今回は一つのリポジトリにブランチを増やして、それをプルリクエストするやり方を紹介します。

プルリクエストを作成するために、まずは適当なブランチを push します。今回は feature-test というブランチを増やして、これを push します。さらにコミットをいくつか積み上げ、それも push します。magit-refs は下のようになります。ここまでは magit の操作だけです。

<img src="https://github.com/eggc/forge-test/blob/master/img/magit-refs.png?raw=true">

さて、ブランチ feature-test を master に向けてプルリクエストを作ります。 `' c p` と入力します。

<img src="https://github.com/eggc/forge-test/blob/master/img/pullreq-source-branch.png?raw=true">
<img src="https://github.com/eggc/forge-test/blob/master/img/pullreq-target-branch.png?raw=true">

それぞれ入力したあと、新しいプルリクエストのタイトルと本文を書き込むためのバッファが開きます。タイトルは自動的に最後のコミットメッセージになっています。

<img src="https://github.com/eggc/forge-test/blob/master/img/pullreq-create-message.png?raw=true">

最後に `C-c C-c` で Github に投稿します。ここは issue の作成と全くどうように `C-c C-k` で取り消す事ができます。取り消した内容は記憶されていて、復元するか破棄するかの操作も同じです。`r` で復元し `d` で破棄します。github のテンプレート機能にも対応している、という点も全く同じです。

## プルリクエストの表示・編集

プルリクエスト作成が済んだあと magit-status へ戻ってくると pull requests というセクションが追加されています。

<img src="https://github.com/eggc/forge-test/blob/master/img/magit-status-pull-requests.png?raw=true">

これも issue と同様です。カーソルをあてて `RET` を押すとプルリクエストの詳細を見ることができます。

<img src="https://github.com/eggc/forge-test/blob/master/img/show-pull-request.png?raw=true">
