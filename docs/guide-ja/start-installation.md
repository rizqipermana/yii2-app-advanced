インストール
============

## 必要条件

このプロジェクトテンプレートが要求する最低限の必要条件は、あなたのウェブサーバが PHP 5.4.0 をサポートしていることです。

## Composer を使ってインストールする

[Composer](http:/[Composer](http://getcomposer.org/) を持っていない場合は、決定版ガイドの [Yii をインストールする](https://github.com/yiisoft/yii2/blob/master/docs/guide-ja/start-installation.md#installing-via-composer) の節の指示に従ってインストールしてください。

Composer がインストールされていれば、次のコマンドを使ってアプリケーションをインストールすることが出来ます。

    composer global require "fxp/composer-asset-plugin:^1.2.0"
    composer create-project --prefer-dist yiisoft/yii2-app-advanced yii-application

最初のコマンドは [composer asset plugin](https://github.com/francoispluchino/composer-asset-plugin/) をインストールします。
これにより、Composer を通じて bower と npm の依存パッケージを管理することが出来るようになります。
このコマンドは全体で一度だけ走らせれば十分です。
第二のコマンドは `yii-application` という名前のディレクトリにアドバンストアプリケーションをインストールします。
望むなら別のディレクトリ名を選ぶことも出来ます。


## アーカイブファイルからインストールする

[yiiframework.com](http://www.yiiframework.com/download/) からダウンロードしたアーカイブファイルをウェブルートの直下、`advanced` と名付けられたディレクトリに解凍します。

その後は、次の項に記載されている指示に従ってください。


## アプリケーションを準備する

アプリケーションをインストールした後に、インストールされたアプリケーションの初期設定をするために、次の各ステップを実行しなければなりません。
これらは全体で一度だけやれば十分です。

1. コンソールターミナルを開き、`init` コマンドを実行して環境として `dev` を選択します。

   ```
   /path/to/php-bin/php /path/to/yii-application/init
   ```

   あるいは、本番サーバでは、非対話モードで `init` を実行します。

   ```
   /path/to/php-bin/php /path/to/yii-application/init --env=Production --overwrite=All
   ```

2. 新しいデータベースを作成し、それに従って `common/config/main-local.php` の `components['db']` の構成情報を修正します。

3. コンソールターミナルを開き、`/path/to/php-bin/php /path/to/yii-application/yii migrate` というコマンドでマイグレーションを適用します。

4. ウェブサーバのドキュメントルートを設定します。

   - フロントエンドのパスは `/path/to/yii-application/frontend/web/`、URL は `http://frontend/` を使用
   - バックエンドのパスは `/path/to/yii-application/backend/web/`、URL は `http://backend/` を使用

   Apache の場合は、次のように設定することが出来ます。

   ```apache
       <VirtualHost *:80>
           ServerName frontend.dev
           DocumentRoot "/path/to/yii-application/frontend/web/"
           
           <Directory "/path/to/yii-application/frontend/web/">
               # 綺麗な URL をサポートするために mod_rewrite を使用
               RewriteEngine on
               # ディレクトリまたはファイルがある場合は、リクエストを直接使用
               RewriteCond %{REQUEST_FILENAME} !-f
               RewriteCond %{REQUEST_FILENAME} !-d
               # そうでなければ、index.php にリクエストを引き渡す
               RewriteRule . index.php

               # index.php をインデックスファイルとして使用
               DirectoryIndex index.php

               # ... その他の設定 ...
           </Directory>
       </VirtualHost>

       <VirtualHost *:80>
           ServerName backend.dev
           DocumentRoot "/path/to/yii-application/backend/web/"
           
           <Directory "/path/to/yii-application/backend/web/">
               # 綺麗な URL をサポートするために mod_rewrite を使用
               RewriteEngine on
               # ディレクトリまたはファイルがある場合は、リクエストを直接使用
               RewriteCond %{REQUEST_FILENAME} !-f
               RewriteCond %{REQUEST_FILENAME} !-d
               # そうでなければ、index.php にリクエストを引き渡す
               RewriteRule . index.php

               # index.php をインデックスファイルとして使用
               DirectoryIndex index.php

               # ... その他の設定 ...
           </Directory>
       </VirtualHost>
   ```

   nginx の場合は、次のように設定することが出来ます。

   ```nginx
       server {
           charset utf-8;
           client_max_body_size 128M;

           listen 80; ## listen for ipv4
           #listen [::]:80 default_server ipv6only=on; ## listen for ipv6

           server_name frontend.dev;
           root        /path/to/yii-application/frontend/web/;
           index       index.php;

           access_log  /path/to/yii-application/log/frontend-access.log;
           error_log   /path/to/yii-application/log/frontend-error.log;

           location / {
               # 本当のファイルでないものは全て index.php にリダイレクト
               try_files $uri $uri/ /index.php$is_args$args;
           }
       
           # 存在しない静的なファイルの呼び出しを Yii が処理するのを防ぐためには、コメントをはずす
           #location ~ \.(js|css|png|jpg|gif|swf|ico|pdf|mov|fla|zip|rar)$ {
           #    try_files $uri =404;
           #}
           #error_page 404 /404.html;

           # /assets ディレクトリの php ファイルへのアクセスを拒否する
           location ~ ^/assets/.*\.php$ {
               deny all;
           }

           location ~ \.php$ {
               include fastcgi_params;
               fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
               fastcgi_pass 127.0.0.1:9000;
               #fastcgi_pass unix:/var/run/php5-fpm.sock;
               try_files $uri =404;
           }

           location ~* /\. {
               deny all;
           }
       }

       server {
           charset utf-8;
           client_max_body_size 128M;

           listen 80; ## listen for ipv4
           #listen [::]:80 default_server ipv6only=on; ## listen for ipv6

           server_name backend.dev;
           root        /path/to/yii-application/backend/web/;
           index       index.php;

           access_log  /path/to/yii-application/log/backend-access.log;
           error_log   /path/to/yii-application/log/backend-error.log;

           location / {
               # 本当のファイルでないものは全て index.php にリダイレクト
               try_files $uri $uri/ /index.php$is_args$args;
           }

           # 存在しない静的なファイルの呼び出しを Yii が処理するのを防ぐためには、コメントをはずす
           #location ~ \.(js|css|png|jpg|gif|swf|ico|pdf|mov|fla|zip|rar)$ {
           #    try_files $uri =404;
           #}
           #error_page 404 /404.html;

           # /assets ディレクトリの php ファイルへのアクセスを拒否する
           location ~ ^/assets/.*\.php$ {
               deny all;
           }

           location ~ \.php$ {
               include fastcgi_params;
               fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
               fastcgi_pass   127.0.0.1:9000;
               #fastcgi_pass unix:/var/run/php5-fpm.sock;
               try_files $uri =404;
           }

           location ~* /\. {
               deny all;
           }
       }
   ```

5. hosts ファイルを書き換えて、フロントエンドとバックエンドのドメインをあなたのサーバに向ける。

   - Windows: `c:\Windows\System32\Drivers\etc\hosts`
   - Linux: `/etc/hosts`

   次の行を追加します。

   ```
   127.0.0.1   frontend.dev
   127.0.0.1   backend.dev
   ```


アプリケーションにログインするためには、最初にユーザ登録をする必要があります。
あなたの任意のメールアドレス、ユーザ名、パスワードを指定してください。
そうすれば、同じメールアドレスとパスワードを使って何時でもアプリケーションにログインすることが出来ます。


> Note: `/` をフロントエンド、`/admin` をバックエンドにして、アドバンストテンプレートを単一のドメインで走らせたい場合は、
> [共有ホスティング環境でアドバンストプロジェクトテンプレートを使う](topic-shared-hosting.md) を参照して下さい。


## Vagrant を使ってインストールする

この方法が最も簡単ですが、時間はかかります (～20分)。

**このインストール方法では、(ウェブサーバ、PHP、MySQL 等々の) ソフトウェアを事前にインストールする必要はありません。** - 単に以下のステップを実行するだけです。 steps!

#### Linux/Unix ユーザ用マニュアル

1. [VirtualBox](https://www.virtualbox.org/wiki/Downloads) をインストールする
2. [Vagrant](https://www.vagrantup.com/downloads.html) をインストールする
3. GitHub [personal API token](https://github.com/blog/1509-personal-api-tokens) を作成する
3. プロジェクトを準備する
   
   ```bash
   git clone https://github.com/yiisoft/yii2-app-advanced.git
   cd yii2-app-advanced/vagrant/config
   cp vagrant-local.example.yml vagrant-local.yml
   ```
   
4. 作成した GitHub personal API token を `vagrant-local.yml` に置く
5. プロジェクトのルートディレクトリに移動する

   ```bash
   cd yii2-app-advanced
   ```

5. 下記のコマンドを実行する

   ```bash
   vagrant plugin install vagrant-hostmanager
   vagrant up
   ```
   
これで全部です。後はただ完了するのを待つだけです。
完了後には、次の URL でローカルのプロジェクトにアクセスすることが出来ます。
* フロントエンド: http://y2aa-frontend.dev
* バックエンド: http://y2aa-backend.dev
   
#### Windows ユーザ用マニュアル

1. [VirtualBox](https://www.virtualbox.org/wiki/Downloads) をインストールする
2. [Vagrant](https://www.vagrantup.com/downloads.html) をインストールする
3. 再起動する
4. GitHub [personal API token](https://github.com/blog/1509-personal-api-tokens) を作成する
5. プロジェクトを準備する
   * [yii2-app-advanced](https://github.com/yiisoft/yii2-app-advanced/archive/master.zip) レポジトリをダウンロードする
   * 解凍する
   * `yii2-app-advanced-master/vagrant/config` ディレクトリに入る
   * vagrant-local.example.yml` を `vagrant-local.yml` にコピーする

6. 作成した GitHub personal API token を `vagrant-local.yml` に置く
7. 次の2行を [hosts file](https://en.wikipedia.org/wiki/Hosts_(file)) に追加する
   
   ```
   192.168.83.137 y2aa-frontend.dev
   192.168.83.137 y2aa-backend.dev
   ```

8. ターミナル (`cmd.exe`) を開き、 **プロジェクトのルートディレクトリに移動して** 次のコマンドを実行する

   ```bash
   vagrant plugin install vagrant-hostmanager
   vagrant up
   ```
   
   (コマンドプロンプトでディレクトリを移動する方法については、[ここ](http://www.wikihow.com/Change-Directories-in-Command-Prompt) を読んでください) 

これで全部です。後はただ完了するのを待つだけです。
完了後には、次の URL でローカルのプロジェクトにアクセスすることが出来ます。
* フロントエンド: http://y2aa-frontend.dev
* バックエンド: http://y2aa-backend.dev

