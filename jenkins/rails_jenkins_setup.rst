====================================================
CentOS上にRuby on Rails用のJenkins環境を構築してみる
====================================================

Ruby、railsのインストール、Jenkinsのインストールとセットアップをやってみます。

.. warning:: 
   | 用意されていたマシンが32bitだったため、とりあえず32bitで進めてみます。
   | そのうち64bitに変えます。

.. contents:: 目次
    :local:

-----------------------------
マシン
-----------------------------
| いつものようにVM切りました。

``uname -a`` の情報はこれ。

::

    Linux api-build-v01 2.6.32-358.el6.i686 #1 SMP Thu Feb 21 21:50:49 UTC 2013 i686 i686 i386 GNU/Linux


``/etc/redhat-release`` の内容はこれ。

::

    CentOS release 6.4 (Final)

作業ユーザは「recoroot」です。

-----------------------------
Rubyのインストール
-----------------------------

| `Ruby on Rails環境構築ガイド <http://www.amazon.co.jp/Ruby-Rails%E7%92%B0%E5%A2%83%E6%A7%8B%E7%AF%89%E3%82%AC%E3%82%A4%E3%83%89-%E9%BB%92%E7%94%B0-%E5%8A%AA/dp/4844333755>`_ のChapter7(P.160～)を参考に進めます。
| 随時、近藤さんメモも参照していきます。

-----------------------------
EPELリポジトリの追加
-----------------------------
| libyaml-develというパッケージが必要になるらしいが、EPELにしかいないそうなのでリポジトリを追加します。
| 本の通りにやります。

#. http://ftp.riken.jp/Linux/fedora/epel/RPM-GPG-KEY-EPEL-6 をrpmコマンドで登録

    ::

        sudo rpm --import http://ftp.riken.jp/Linux/fedora/epel/RPM-GPG-KEY-EPEL-6

#. http://ftp.riken.jp/Linux/fedora/epel/6/i386/epel-release-6-8.noarch.rpm をrpmコマンドで登録

    ::

        sudo rpm --import http://ftp.riken.jp/Linux/fedora/epel/6/i386/epel-release-6-8.noarch.rpm
        
とやったらエラーになりました。

    ::

        http://ftp.riken.jp/Linux/fedora/epel/6/i386/epel-release-6-8.noarch.rpm: key 1 not an armored public key.

なので、wgetでローカルに取得してから追加を行うこととしました

    ::

        wget http://ftp.riken.jp/Linux/fedora/epel/6/i386/epel-release-6-8.noarch.rpm
        sudo rpm -ivh epel-release-6-8.noarch.rpm 

#. ``yum update`` を実行すると、EPELの設定が取り込まれます。

    .. image:: ../img/jenkins/epel_repo_add.jpg
        :width: 800px

-----------------------------
Rubyのインストール
-----------------------------
| 依存パッケージをインストール後に、Ruby本体をインストールします。
| これもまた本の通りにやります。

#. | 依存パッケージをyumで入れます。
   | gccとかwgetとかmakeとか元々入ってますけど、とりあえず本の通りに全て指定します。
   | (mysqlとかpostgersqlとかは使う予定は無いけど、、、いつか使うかもしれないので)
   
    ::
   
        sudo yum -y install gcc gcc-c++ make autoconf openssl-devel readline-devel libyaml-devel mysql-devel postgresql19.1-devel wget git

#. | Rubyをmake-installします。
   | `Ruby公式サイト <https://www.ruby-lang.org/ja/downloads/>`_ では、2.0系の安定板が「Ruby 2.0.0-p451」となっていたので(2014/04/03現在)、こちらを使用します。
   
        * wgetで「http://cache.ruby-lang.org/pub/ruby/2.0/ruby-2.0.0-p451.tar.gz」を取得
        * ``tar -xvf ruby-2.0.0-p451.tar.gz`` で解凍
        * 解凍されたディレクトリに移動し、``./configure`` 実行してMakefile作成
        * ``make`` 実行してビルド
        * ``make install`` 実行してインストール

        こんな感じでインストールが終了します。
        
            .. image:: ../img/jenkins/ruby_install_complete.jpg
                :width: 800px
        
        
        ``ruby -v`` でバージョンを、``which ruby`` でパスが通っている先を確認しておきましょう。

            .. image:: ../img/jenkins/ruby_version_which.jpg
                :width: 800px
    
        .. note::

            展開したソースはここに置きっぱなしになっています。
            
            ::
                
                /home/recoroot/ruby-2.0.0-p451
            
            | アンインストールする場合は「.installed.list」とか必要になりそうなので、このままにしておきます。
            | 参考URL：http://www.mk-mode.com/octopress/2013/03/02/ruby-2-0-0-install-by-src/
            |
            | ソースインストールする時のソース置き場は統一した方がいいですね。。。

-----------------------------
RubyGemsのアップデート
-----------------------------
| これで。
|
| ``sudo gem update --system``
|
| やってみたら、コマンドが見つからないと出た。。
|
| よく分からなかったので、suでrootになって
|
| ``gem update --system``
|
| でアップデートしました。
| アップデート後のバージョンは「2.2.2」となりました。

.. note:: 
    
    | 後でググってみたら、コマンドが見つからない件について情報がありました。
    |
    | http://blog.bungu-do.jp/archives/3525
    |
    | CentOS6.4からの事象ということらしいです。
    | リンク先の情報を参考に、visudoでsudoersを開いてsecure_pathに「/usr/local/bin」を追加することで、sudoからのgem呼び出しもできるようになりました。
    |
    | ``Defaults    secure_path = /sbin:/bin:/usr/sbin:/usr/bin:/usr/local/bin/``
    

-----------------------------
Bundlerのインストール
-----------------------------
| これで。
|
| ``sudo gem install bundler``
|
| インストールされたバージョンは「1.6.1」でした。

-----------------------------
railsのインストール
-----------------------------
| これで。
|
| ``sudo gem install rails --version "=4.0.2" --no-rdoc --no-ri``
|

-----------------------------------------
ローカルでテスト用プロジェクト作る
-----------------------------------------
| 参考にしているページ(http://www.ruby.or.jp/ja/tech/development_tool/ci/)の通りにやりました。

#. rails new でプロジェクト作る。
#. GemfileにRSpecとsimplecov用のgemを追加する。

    ::

        group :development, :test do
          gem 'rspec'
          gem 'rspec-rails'
          gem 'simplecov', :require => false
          gem 'simplecov-rcov', :require => false
        end

#. ``bundle install`` する。
#. ``rails g rspec::install`` で、RSpecの設定ファイルを生成する。
#. | simplecovのモジュールが使用できるように、生成されたspec/spec_helper.rbを編集する。
   | 「require 'rspec/rails'」の行の前に以下を追加。
   
    ::

        require 'simplecov'
        require 'simplecov-rcov'
        SimpleCov.formatter = SimpleCov::Formatter::RcovFormatter
        SimpleCov.start 'rails'

#. ``rails g scaffold Article title:string`` を実行して、scaffoldを作成する。
#. ``rake db:migrate`` を実行して、DBのマイグレーションを行う。
    * 以下のエラーが出た。
    
        ::
            
            rake aborted!
            You have already activated rake 10.1.1, but your Gemfile requires rake 10.2.2. Prepending `bundle exec` to your command may solve this.
            c:/rails_app/jenkins-test/config/boot.rb:4:in `<top (required)>'
            c:/rails_app/jenkins-test/config/application.rb:1:in `<top (required)>'
            c:/rails_app/jenkins-test/Rakefile:4:in `<top (required)>'
            (See full trace by running task with --trace)
   
        | ``bundle exec rake db:migrate`` で実行した。
        | 参考にしたのは、基礎Ruby on RailsのP.164

-----------------------------------------
Jenkinsジョブ流してみる
-----------------------------------------
SVNからのチェックアウト後の処理として、シェルの実行で以下を指定して実行してみました。

    ::

        cd ./jenkins-test
        /usr/local/bin/bundle install

実行してみると以下のようなエラーが出ました。

    ::

        ~ 中略 ~

        Gem::Ext::BuildError: ERROR: Failed to build gem native extension.

        /usr/local/bin/ruby extconf.rb 
        /usr/local/bin/ruby: invalid option -/  (-h will show valid options) (RuntimeError)

        extconf failed, exit code 1

        Gem files will remain installed in /home/tomcat/.jenkins/jobs/jenkins_test/workspace/jenkins-test/vendor\bundle
        /ruby/2.0.0/gems/atomic-1.1.16 for inspection.
        Results logged to /home/tomcat/.jenkins/jobs/jenkins_test/workspace/jenkins-test/vendor\bundle
                /ruby/2.0.0/extensions/x86-linux/2.0.0-static/atomic-1.1.16/gem_make.out
        An error occurred while installing atomic (1.1.16), and Bundler cannot continue.
        Make sure that `gem install atomic -v '1.1.16'` succeeds before bundling.


お告げの通りに ``gem install atomic -v '1.1.16'`` とやってみます。

    ::
    
        [sudo] password for recoroot: 
        Building native extensions.  This could take a while...
        Successfully installed atomic-1.1.16
        Parsing documentation for atomic-1.1.16
        unable to convert "\x84" from ASCII-8BIT to UTF-8 for ../../extensions/x86-linux/2.0.0-static/atomic-1.1.16/atomic_reference.so, skipping
        unable to convert "\x84" from ASCII-8BIT to UTF-8 for lib/atomic_reference.so, skipping
        1 gem installed

| あんまり関係無さそう。

| エラーメッセージを見ていると ``vendor\bundle`` の部分がバックスラッシュになっているのが怪しい。
| jenkins-test/.bundle/configファイルに以下の記載があるため。

    ::

        ---
        BUNDLE_PATH: vendor\bundle
        BUNDLE_DISABLE_SHARED_GEMS: '1'

| ``vendor/bundle`` に変更してみます。
| あと、改行がCRLFになっていたので、LFに変更しました。

| エラーが変わりまして、以下の通りとなりました。

    ::

        ~ 中略 ~

        Gem::Ext::BuildError: ERROR: Failed to build gem native extension.

        /usr/local/bin/ruby extconf.rb 
        checking for sqlite3.h... no
        sqlite3.h is missing. Try 'port install sqlite3 +universal',
        'yum install sqlite-devel' or 'apt-get install libsqlite3-dev'
        and check your shared library search path (the
        location where your sqlite3 shared library is located).

| sqlite-develを入れればいいようです。(サンプルアプリのGemfileに ``gem 'sqlite3'`` を書いてるからですね。)
| 今後sqliteを使う予定はないですが、ライブラリを入れても害はないので、
| ``sudo yum -y install sqlite-devel``
| で入れておきます。
|
| bundle installはこれで通るようになりました。
| 次はこのエラーが出ます。

    ::

        java.io.IOException: Cannot run program "rake" (in directory "/home/tomcat/.jenkins/jobs/jenkins_test/workspace/jenkins-test"): error=2, そのようなファイルやディレクトリはありません

| Jenkins Rakeプラグインを使ってspec実行しようとしていたのですが、rakeが認識できていないようなので、コマンドラインからの実行に変更してみます。

    ::

        /usr/local/bin/bundle install
        /usr/local/bin/bundle exec /usr/local/bin/rake spec

| これで実行すると、今度は以下のエラーになりました。

    ::

        ExecJS::RuntimeUnavailable: Could not find a JavaScript runtime. See https://github.com/sstephenson/execjs for a list of available runtimes.

| node.js入れましょうか。
| nvmとか入れた方がいいの？とか悩みますが、とりあえず素のままyumで入れることにしました。
| EPELのリポジトリ登録してあれば、yumでインストールできるそうです。
| 参考URL: http://qiita.com/you21979@github/items/4efd9fc4363573191b5c

    ::

        yum install nodejs npm --enablerepo=epel

バージョンは以下のようになりました。

    ::

        [recoroot@api-build-v01 ~]$ node -v
        v0.10.26
        [recoroot@api-build-v01 ~]$ npm -v
        1.3.6

この状態で実行すると、ジョブが正常に動きました！

