+++
title = "Sophos home 使ってみる"
date = 2016-07-01T00:00:00+09:00
images = []
tags = ["Sophos", "Anti-virus", "malware", "Web Protection", "Web Category Access"]
categories = ["security"]
draft = false
Summary = "最近ランサムウェアの情報が飛び交ってきているのでお試ししてみました。"
+++

はじめに
--------

最近ランサムウェアの情報が飛び交ってきているので気になっていろいろ調べたらちょっと面白そうなSophos Home for PCs and Macsを試してみます。

なぜ興味を持った？
------------------

一番興味を持ったのはSophos XG Firewall Home Edition。[^1]

これを自宅に構築してみようと思ったけど、この機能でSecurity Heartbeat™なんてのが あるじゃないですか。
じゃ、手始めにエンドポイントのお試ししてみようと思ったわけです。

ダウンロード
------------

1.  アカウントの作成

    [ここ](https://www.sophos.com/ja-jp/lp/sophos-home.aspx) のダウンロードボタン をクリックします。
    すると、アカウント作成の画面になるので名前とE-mailアドレス、パスワードを入力してアカウントを作成します。（ダウンロードじゃないじゃん。）

2.  E-Mail認証からのログイン

    登録したメールアドレスにメールが届きますので、Confirm Emailします。
    ログインのリンクを押してログイン画面を表示してE-mailとパスワードを入力してログインします。

3.  やっとダウンロード

    Add Device -\> Install で、インストールできます。

早速試してみます
----------------

早速機能を試しました。ざっくり一通り確認しただけなのでこれからちょっと利用し続けてみます。

-   ウイルススキャン

    フルスキャン試してます。Mac book Air OS X EL Capitan 1.6 GHz ( GB 1600 MHz DDR3 ですが、２時間ほど経過して75％ぐらいまで終わりました。

-   ウイルスダウンロード

    [テストウイルス EICAR](http://files.trendmicro.com/products/eicar-file/eicar.com) をダウンロードしようとするときっちりブロックしてくれます。（当たり前だけどちょっと感動）

-   マルウェア

    [The Anti-Malware Testfile](http://www.eicar.org/86-0-Intended-use.html) の68byteの文字列を適当な名前のファイルで作成します。
    するときっちり削除してくれます。（これにも当たり前だけど感動）

-   Web フィルタ

    カテゴリ毎にAllow, Warn, Blockが選択できます。
    この制御はクラウドにあるダッシュボードでデバイス毎に制御が可能です。
    子供向けに見せたくないカテゴリをBlockとかできますね。

    Warnだと一旦警告画面が表示されて前画面に戻るかページを表示するか決めることができるようです。

**Footnotes**

[^1]: [Sophos XG Firewall Home Edition](https://www.sophos.com/ja-jp/products/free-tools/sophos-xg-firewall-home-edition.aspx)
