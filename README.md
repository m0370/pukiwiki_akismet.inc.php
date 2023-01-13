## スパムフィルタ Akismet プラグイン  

https://oncologynote.jp/?f39dac16f2

## 概要

WordPress用スパム対策プラグインとして開発されており、多数のWordPress利用者から「最強」との呼び声が高いプラグイン「Akismet」を PukiWiki で利用します。

投稿内容をチェックし、Spamを防止することができます。 通常の投稿がスパムと間違われた場合、キャプチャ認証を通してスパム取り消し報告ができます。

注：Akismet  は確実にスパムをフィルタリングするわけではありません。１つもスパムを通したくない場合は常にユーザにキャプチャ認証を求める形にする必要があります。akismet プラグインでは、ファイル中設定項目、PLUGIIN_AKISMET_USE_AKISMET を FALSE  にすることで実現できます（実際は、セッション中、最初の１回だけキャプチャ認証を求めます）

sonotsさんが作成されたプラグインですが、すでに作成から10年以上経過してakismet.netは廃止されプラグインが動作しなくなっていたので、これをGoogle recaptcha v2で動作するように改造したものを置いておきます。あくまでGoogle recaptcha v2で強制的に動くように改変したものなので、詳細なバグの確認などはできていません。

v2.0.2（2022.12.3）からPHP 8.0/8.1でも動作可能になりました。

- オリジナル
  [sonots' pukiwiki プラグイン](http://pukiwiki.sonots.com/)
- Internet Archive
  https://web.archive.org/web/20211026082832/http://pukiwiki.sonots.com/

## ダウンロード  

ダウンロードして保存し、plugin ディレクトリにおいてください。常に開発版です。

PukiWiki EUC-JP 版をお使いの方はファイル内文字コードを UTF-8 から EUC-JP に変更して保存してください。文字コードの変換の仕方は PukiWiki に限った話ではないので省略します。

## インストール  

### 事前準備  

- WordPress API Key の取得
  - Akismet は元々 WordPress 用プラグインとして作られたもので WordPress のユーザ登録をする必要があります。
  - 以下のページから WordPress.com ユーザ登録をおこなうとAPIキーを取得できます。（基本的に無料ですが、商用に利用する場合は有料となります。）
  - [http://wordpress.com/api-keys/![img](https://web.archive.org/web/20211026090240im_/http://pukiwiki.sonots.com/www/image/plus/ext.png)](https://web.archive.org/web/20211026090240/http://wordpress.com/api-keys/) → [http://wordpress.com/signup/![img](https://web.archive.org/web/20211026090240im_/http://pukiwiki.sonots.com/www/image/plus/ext.png)](https://web.archive.org/web/20211026090240/http://wordpress.com/signup/)
- reCAPTCHA API Key の取得
  - SPAM 取り消し報告をする際に、Google reCAPTCHAを利用した CAPTCHA（歪曲文字）認証を導入しています。Google reCAPTCHA v2に対応しています。v3およびEnterpriseには対応していません。
  - 以下のページで設置予定サイトのドメインを入力すると、APIキー（publickey, privatekey）を取得できます。（無料です）
  - [https://www.google.com/recaptcha/about/](https://web.archive.org/web/20211026090240/http://recaptcha.net/api/getkey)

### プラグイン設定  

akismet.inc.php 内で設定をしておきます。

**必須**

- PLUGIN_AKISMET_API_KEY
  - 取得した Akismet APIキーを設定。
- PLUGIN_AKISMET_RECAPTCHA_PUBLIC_KEY
  - Googleで取得したサイトキーを設定。これはWebサイトのソース上で公開される公開錠です。
- PLUGIN_AKISMET_RECAPTCHA_PRIVATE_KEY
  - Googleで取得したシークレットキーを設定。これはサイト管理者以外に知られないようにしてください。

**オプション**

- PLUGIN_AKISMET_USE_AKISMET
  - Akismet を使用するか否か。FALSE とすると全てをスパムと見なし、キャプチャ認証を常に要求する形になります。
- PLUGIN_AKISMET_USE_RECAPTCHA
  - reCaptcha を使用するか否か。FALSE とすると captcha フォームの変わりにボタンだけがでてくる。
- PLUGIN_AKISMET_IGNORE_PLUGINS
  - スパムフィルタしないプラグインをカンマ区切りで指定。デフォルトでは read,vote,vote2,timestamp。何も指定されていない状態では、全ての POST をフィルタします。
- PLUGIN_AKISMET_USE_SESSION
  - 一度人間認証(captcha 等)がすめば、以降また人間認証をせずにすむように session に記録する。デフォルトで有効。環境によってはセッションが使えず機能しないかもしれない。
- PLUGIN_AKISMET_THROUGH_IF_ADMIN
  - 管理者ならばスパムチェックしない。
- PLUGIN_AKISMET_THROUGH_IF_ENROLLEE
  - 登録ユーザならばスパムチェックしない。デフォルトでは無効。
- PLUGIN_AKISMET_SPAMLOG_FILENAME
  - スパムログを取るファイル名の設定。デフォルトでは cache/spamlog.txt。PukiWiki Plus! の場合は log/spamlog.txt
- PLUGIN_AKISMET_SPAMLOG_DETAIL
  - ログを取る際に本文も保存する。デフォルトでは無効
- PLUGIN_AKISMET_ONELOG_DAYS
  - １つのログファイルに保存される日数。デフォルトで１０日
- PLUGIN_AKISMET_KEEPLOG
  - いくつのログファイルを保持しておくか。デフォルトで３。
- PLUGIN_AKISMET_AUTOPOST_AFTER_SUBMITHAM
  - スパム取り消し報告後に自動で本来の投稿内容を自動投稿。デフォルトで有効。不具合が出るようなら FALSE にしてください。本来の投稿内容を表示するだけで自動投稿はしなくなります。念のために用意しておきました。

### PukiWiki本体修正  

#### lib/pukiwiki.php

以下のような箇所に + が付いている行を追加します。

```
 if (isset($vars['cmd'])) {
         $is_cmd  = TRUE;
         $plugin = & $vars['cmd'];
 } else if (isset($vars['plugin'])) {
         $plugin = & $vars['plugin'];
 } else {
         $plugin = '';
 }
+// Akismet Spam filtering
+if (exist_plugin('akismet')) { // require_once
+    PluginAkismet::spamfilter();
+}
```

#### recaptcha.jsの読み込み

本体のスキンで下記の行を読み込んでおきます。このjsで認証画像や「ロボットではありません」のボタンを表示します。

```

<script src="https://www.google.com/recaptcha/api.js" defer></script>

```

#### pukiwiki.ini.php

Akismet スパムフィルタを利用するので、PukiWiki 1.4.8 以上で導入を予定されていたデフォルトのスパムフィルタ機能はオフにすることを推奨する。もしこの行がpukiwiki.ini.phpに無ければ無視して可。

```
-$spam = 1;      // 1 = On
+$spam = 0;      // 1 = On
```

## 付加機能  

### ログ参照  

アクション型でアクセス (index.php?cmd=akismet) するとスパムフィルタログがみれます。

ちなみに、ログファイルは PukiWiki 本家では cache/spamlog.txt に Plus! では log/spamlog.txt に保存されています。

### 管理者ならばスパムチェックしない  

PLUGIN_AKISMET_THROUGH_IF_ADMIN を TRUE (デフォルトで TRUE) にしておくと、 PukiWiki の Basic 認証機能を利用して管理者としてログインしている場合、スパムチェックを行わないようになります。

**PukiWiki Plus! における設定**

認証ユーザに、管理者(2)かコンテンツ管理者(3)権限を与えておきます。

例：auth_users.ini.php

```
<?php
$auth_users = array(
        'admin' => array('{x-php-md5}1a1dc91c907325c69271ddf0c944bc72',2),
);
?>
```

値 array の第二引数に権限レベル（この例では2 == 管理者）を追加しておきます。 より詳しい情報が知りたい方は [plus:Documents/Role![img](https://web.archive.org/web/20211026090240im_/http://pukiwiki.sonots.com/www/image/plus/ext.png)](https://web.archive.org/web/20211026090240/http://pukiwiki.cafelounge.net/plus/index.php?Documents%2FRole#ta86f8a1) を参照してください。

PukiWiki Plus! ではアクション型プラグイン形式の ?cmd=login で、Basic 認証を起動することができます。 そのプラグインへのリンクを MenuBar にでも貼っておくと便利そうな気がします（login プラグインにはボタンを表示するブロック型もあるのですが、ボタンよりはただのリンクの方が Akisimet プラグインの仕様上都合が良いです。)

さらに #role プラグインという、ログイン状態での権限レベルを表示してくれるプラグインもあります。

**PukiWiki 本家における設定**

PukiWiki 本家では Basic認証におけるユーザ権限の区別がないので、$auth_usersにおけるユーザ名 'admin' を管理者とみなすことにしました。

例：pukiwiki.ini.php

```
$auth_users = array(
        'admin' => '{x-php-md5}1a1dc91c907325c69271ddf0c944bc72',
);
```

login　プラグインはないので、この　admin　ユーザで基本認証をパスするために、adminユーザならば参照できるように参照制限した、Admin専用ページを作成しておくと便利そうな気がします。

### 登録者ならばスパムチェックしない  

akismet.inc.php　中の　PLUGIN_AKISMET_THROUGH_IF_ENROLLEE　を　TRUE　に変更すると（デフォルトではFALSE）、 $auth_users　で設定したユーザならば誰でも、一度基本認証が済んでいればスパムチェックをしないようになります。

## 技術的詳細  

### 悩み点  

どのくらいをスパムフィルタするか

- AkismetのAPIでは、以下の情報を元にスパム判定をおこなう
  - 投稿者(author)
  - E-mail
  - WebSite(投稿者のURL)
  - 本文
  - 基本的に「本文」だけを設定すればスパム判定はおこなわれる。
- が、Pukiwikiのほうではプラグインによって、本文が設定される変数名が異なる。
  - 例えばコメントプラグインなどの場合は「msg」に本文が設定されるが、トラッカープラグインの場合は変数名は、ユーザーの設定によって異なる。
- ようするに、AkismetAPIの本文に対して、Pukiwiki上で投稿された変数のうちどれをマッピングさせるかという問題が存在する。

案

- 案１：$vars['msg'] を使用
  - comment, article プラグインや edit プラグイン（通常編集）には対応できる
  - tracker の場合設定 :config ページで msg にしなければならない
  - article の subject などがスルー
- 案２：$vars 全体を使用（オリジナル小沼版）
  - 全フィールドに対応。
  - しかし、ゴミが付く（自動生成される、メッセージダイジェスト値など）。それらのせいで普通の投稿もスパムに間違われはしないか懸念がある。
  - また edit の場合元の文章全体を $vars['original'] に保存しているので Akismet に送信する情報量が２倍。重い（original はスパムではありえないはずなので送る必要がない）。
  - などなど送信しない値をこつこつと１つずつ対応していくか、無視して全体送信か。
- 案３：送信する値をこつこつと１つずつ対応
  - 案２とは逆に送信する値をこつこつと１つずつきっちり対応。
  - 大変。また、PukiwikiやAkismetの仕様変更が行われるたびに修正しなければならず、サポートも大変。
  - tracker の場合 :config 設定による
- 案４：page_write 関数の文章書き込みの直前にチェック
  - Wikiページに書き込まれる文章だけを確実に取得できる
  - しかし、たとえ comment プラグインでもそのページ本文全てを Akismet.com に送信することになり重い
    - 書き込む前の内容と差分を取って送ればよい。
    - それによって、確実に新規内容だけをチェックできる。
    - ただ、合体→分離と無駄なことをしている感は否めない。
  - page_write を使用しない、つまりwikiページへの書き込みをしないプラグイン(bbs.inc.phpやメール送信プラグイン)には対応できない。

初版は案４で不具合が発生しやすいことがわかり、現在は案２で少しだけ個々のプラグインを特別対応している。

### 調査  

WordPress Akismet ができること（ [http://akismet.com/faq/![img](https://web.archive.org/web/20211026090240im_/http://pukiwiki.sonots.com/www/image/plus/ext.png)](https://web.archive.org/web/20211026090240/http://akismet.com/faq/) ）

- はじいたログを１５日間データベースに保存。後で手動チェックできる。
  - ログを覗く機能が必要
  - Wiki なのでコメントログのようにはいかない・・・
- 間違ってスパムと判断されていた場合 akismet.com に false positive 報告ができる
  - submitHam()
- スパムと判断されていなかったが、手動でスパムチェックをすると true negative 報告ができる
  - submitSpam()
- Akismet.com がダウンしていた場合、moderation queue (手動チェックキュー）に入る
  - we've built up a highly fault-tolerant infrastructure と言っているのであまりダウンすることはないらしいが
  - moderation queue がないので、スルーして書き込みOKにするしかないだろう

### 関連  

- [sonots' Pukiwiki プラグイン/akismet.inc.php](http://pukiwiki.sonots.com/edit.php?Plugin%2Fakismet.inc.php)
- [Google reCaptcha](https://www.google.com/recaptcha/about/)
- [Akismet](https://akismet.com/)
- [dev:PukiWiki/1.4/ちょっと便利に/Akismetによるspam(スパム)防止機能](https://pukiwiki.osdn.jp/dev/?PukiWiki/1.4/%E3%81%A1%E3%82%87%E3%81%A3%E3%81%A8%E4%BE%BF%E5%88%A9%E3%81%AB/Akismet%E3%81%AB%E3%82%88%E3%82%8Bspam%28%E3%82%B9%E3%83%91%E3%83%A0%29%E9%98%B2%E6%AD%A2%E6%A9%9F%E8%83%BD)
- [PukiWiki/Akismetによるspam(スパム)防止機能](http://www.ark-web.jp/sandbox/wiki/190.html)
- [最強の呼び声高いブログ用対スパムプラグイン「Akismet」- GIGAZINE](https://gigazine.net/news/20070127_akismet/)
- [美麻Wikiでシステム的に修正している点 - spam_filter.php](http://miasa.info/index.php?%C8%FE%CB%E3Wiki%A4%C7%A5%B7%A5%B9%A5%C6%A5%E0%C5%AA%A4%CB%BD%A4%C0%B5%A4%B7%A4%C6%A4%A4%A4%EB%C5%C0#ofa18e88)
- [dev:BugTrack/772](https://pukiwiki.osdn.jp/dev/?BugTrack%2F772)
  
