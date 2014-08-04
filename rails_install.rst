====================================================
ローカルPC上にRuby on Railsの開発環境を構築してみる
====================================================

ローカルPCでの環境構築を行った際のメモ書きです。

.. warning:: 
   | Windows環境を前提とした説明になっています。
   | 他のOSを使用している場合は、本ページを参考に自分でアレンジして進めてください。
   
.. contents:: 目次
    :local:

-----------------------------
各種ツールのバージョン
-----------------------------
以下の構成でやってみます。

* Ruby
    Version 2.0.0pxxx

* Ruby on Rails
    Version 4.0.2
 
* IDE
    Aptana3を使ってみます。

    .. note:: 
        `RubyMine <http://www.jetbrains.com/ruby/>`_  最強説があるようなので使ってみたいところですが、

            * 有料(しかもぼちぼちお高い$99)
            * 比較対象があったほうが採用検討には有効

         ということで、今回はAptanaを使ってみることにしました。

-----------------------------
Rubyのインストール
-----------------------------
| Windows用のインストーラが有志によって作成されているので、それをDLして使用します。

        * ActiveScriptRuby ： http://www.artonx.org/data/asr/
        * Ruby Installer : http://rubyinstaller.org/

| 今回はRuby Installerから「rubyinstaller-2.0.0-p353.exe」をDLしてインストールしました。

-----------------------------
Devkitのインストール
-----------------------------
| Rails用のGemをインストールするにはDevkitというツールが必要らしいです。
| なのでインストールしましょう。

    .. note:: 
        | Gemの中にはCで書かれているものもあるらしく、そのまま引っ張ってきてもWindows環境でコンパイルできなくて使えない、
        | なのでコンパイルするためのツールを作りましょう、というのがDevkitのざっくりとした概要らしいです。
        |
        | 参考URL：http://supportdoc.net/support-ror3/devkit.html

#. | Ruby InstallerにDevkitのDLリンクもあるので、そこからDLします。
   | 今回は32bitのWindows7でRuby2.0を使用するので、以下の赤丸のリンクですね。
   
    .. image:: ./img/devkit_dl.jpg
        :width: 600px

#. | インストーラをダブルクリックするとインストール先を聞かれます。
   | 今回は「C:\devkit」にインストールすることにしました。
   
    .. image:: ./img/devkit_dir.jpg
        :width: 250px

#. 「Extract」を押下するとDevkitのインストールが開始され、しばらく待つと完了します。

#. devkitのコンパイラを使用できるようにするため、コマンドプロンプトから「C:\\devkit\\devkitvars.bat」を実行しておきます。

    .. image:: ./img/devkit_vars.jpg
        :width: 500px

    .. warning:: この手順、もしかしたら必要ないかも。。。(古いやり方かも。。。 ）

#. devkitの初期化を行うため、コマンドプロンプトから以下の2つを実行します。

    :: 
    
        ruby c:\devkit\dk.rb init

    :: 
    
        ruby c:\devkit\dk.rb install

-----------------------------
Ruby on Railsのインストール
-----------------------------
コマンドプロンプトから、以下のコマンドを実行しましょう。

::
    
    gem install rails --version "=4.0.2" --no-rdoc --no-ri

オプションについては、以下の通りです。
    
    *  「--version "=4.0.2"」
        | Railsのバージョンを指定しています。
        | この場合は「4.0.2」決め打ちとなります。
        | 「"~>4.0.0"」と指定すると、「4.0.0以上4.1未満のバージョン番号を持つものを選択してください」ということになるそうです。

    * 「--no-rdoc --no-ri」
        | インストールされるGemのrdocとriをインストールしない、という指定になります。
        | インストールするとかなり時間がかかるため、インストールしないのが一般的？らしいです。
        
        .. warning:: 
            | とは書いたものの、rdocとriが何者なのかがまだ分かっていません。。。
            | とりあえずは、このまま進めてみます。
    
        .. note::
            Gemで何かをインストールする際に毎回このオプションを指定するのは面倒なため、以下の手順でデフォルト化している人も多いようです。
            
                #. 以下の内容を記載した「.gemrc」ファイルを作成する。
                    gem: --no-ri --no-rdoc
                #. 以下に配置する。
                    C:\Users\ユーザー名\.gemrc

-----------------------------
Aptana3のインストール(1)
-----------------------------
インストール、ついでに日本語化までしてみましょう。

    .. note:: 
        | Aptanaはeclipseベースの開発ツールのため、JDK(Runtime)が無いと動きません。
        | インストールされていない人(いないとは思いますが。。。)は、別途インストールしてpathを通しておきましょう。


#. `ここ <http://www.aptana.com/>`_ にダウンロードボタンがあるので押しましょう。

#. eclipseプラグインでもインストールできますが、色々重くなりそうなのでStandalone版を使用します。
    
    .. image:: ./img/aptana_dl.jpg
        :width: 600px

    | exeファイルのDLが開始されると、以下のページが表示されました。
    | よく見ると、Gitが必要だと書いてあります。
   
    .. image:: ./img/aptana_dl_start.jpg
        :width: 600px
    
    なので、exeの実行はいったん止めて、こちらを先にインストールしましょう。

-----------------------------
msysgitのインストール
-----------------------------

.. note:: msysgitとは、Windows環境上でGitを使用できるようにするソフトウェアです。

#. 先程のページにあったリンク(http://code.google.com/p/msysgit/)をクリックすると、ページが移動してました。

    .. image:: ./img/msysgit_move.jpg
        :width: 500px

#. | 移動先ページからDLボタンを押すと、ファイルの一覧が表示されました。
   | どれがいいかよく分かりませんが、最新かつ安定してそうな「Git-1.8.4-preview20130916.exe 」を使ってみることにしました。

    .. image:: ./img/msysgit_dl.jpg
        :width: 500px

#. | exeを実行するとインストールが始まります。
   | インストール先はデフォルトにしました。
   | 基本設定は以下のように変更しています。
        
        * Windows Explorer integration
            → 右クリックで表示されるメニューらしい。そんなに必要ないので、simple構成にした。
        
        * 「Associate .git* ...」「Associate .sh ...」
            → ファイル関連付けの設定みたいなので、外した。 

        .. image:: ./img/msysgit_setup_1.jpg
            :width: 300px
            
   以降の項目は、全てデフォルトとしました。

#. | インストール先のbinディレクトリにpathを通しておきます。
   | デフォルトでインストールした場合、以下になるはずです。
   
    ::
    
        C:\Program Files\Git\bin

-----------------------------
Aptana3のインストール(2)
-----------------------------
Aptanaのインストールに戻ります。

#. | DLしたexeを実行します。
   | 以降は、特にこだわりが無ければデフォルト設定でインストールしてしまえばよろしいかと思います。
   | 自分は、インストール先ディレクトリを変えたのと、ファイル関連付け(.css、.js)を外しました。 ※Sublime Text使いたいので。

    .. image:: ./img/aptana_setup_1.jpg
        :width: 450px

#. | 日本語化します。
   | `Pleiades <http://mergedoc.sourceforge.jp/#pleiades.html>`_ のページからプラグインをDLしてきて、

    .. image:: ./img/pleiades.jpg
        :width: 450px

   | ファイルを解凍するとfeturesとpluginsディレクトリがいるので、これをAptanaインストールディレクトリ直下に上書きしてください。
   | **一般的な、手動でeclipseにプラグインを追加する方法と同じです。**

#. Aptanaインストールディレクトリ直下に「AptanaStudio3.ini」ファイルがいるので、これを開いて一番最後に以下の行を足します。
    
    ::
    
        -javaagent:plugins/jp.sourceforge.mergedoc.pleiades/pleiades.jar

   これで起動すれば、日本語化されています。

----------------------------------------
AptanaからRailsプロジェクトを作ってみる
----------------------------------------
Aptanaを起動してプロジェクトを作ってみましょう

#. Aptanaを起動。プロジェクトエクスプローラーで右クリックして「新規」→「Railsプロジェクト」を押下します。

    .. image:: ./img/make_rails_pj_1.jpg
        :width: 450px

#. プロジェクト名を入力します。他は特に変更しなくてよいと思います。

    .. image:: ./img/make_rails_pj_2.jpg
        :width: 450px

#. プロジェクト用のコンソールが起動し、rails newコマンドが発行されてプロジェクトが作成されます。

    .. image:: ./img/make_rails_pj_3.jpg
        :width: 600px

#. railsコマンドの実行が完了してコンソールが入力待ちになったら、

    ::
    
        rails server
   
   とコマンドを打ち込んで、WEBrickを起動してみましょう。
   

    .. image:: ./img/make_rails_pj_4.jpg
        :width: 600px

#. ブラウザから「http://localhost:3000」にアクセスし、以下の画面が表示されたら完了です。

    .. image:: ./img/make_rails_pj_5.jpg
        :width: 300px

----------------------------------------
Ruby開発ツールの追加
----------------------------------------
| 「ウィンドウ」→「設定」でRubyについての設定をしたいのですが、Aptanaのデフォルトだと
| 必要なツールがインストールされていないようです。
|
| 参考URL:http://sakamoto-san.com/ruby_aptana_install.html
|
| なので、追加します。

#. 「ヘルプ」→「新規ソフトウェアのインストール」と進みます。

#. 作業対象「すべての使用可能なサイト」、フィルタに「Ruby」と入れると、以下のように開発ツールが表示されます。

    .. image:: ./img/ruby_plugin_1.jpg
        :width: 600px

    .. note:: 画像のように選択できない場合は、「使用可能なソフトウェア・サイト」リンクから「Eclipse Indigo Update Site」のチェックを付けてみてください。

   これにチェックを付けて、「次へ」でインストールを進めてください。

#. | インストールが終わるとAptanaの再起動を求められるので、再起動します。
   | 「ウィンドウ」→「設定」を開いてみると、Rubyの設定ができるようになっています。

    .. image:: ./img/ruby_plugin_2.jpg
        :width: 450px

----------------------------------------
Aptanaでデバッグ環境を整える
----------------------------------------
特に何も設定をせずにRubyプログラムに対してデバッグ実行をかけると、以下のエラーが発生しました。
    .. image:: ./img/aptana_debug_error.jpg
        :width: 450px
        
「ruby-debug-ide」が無いと言われているので、Gemでインストールします。

    .. image:: ./img/aptana_debug_debugide.jpg
        :width: 450px

通常のRubyスクリプトは、ファイルを右クリックして「デバッグ」→「Rubyアプリケーション」で

    .. image:: ./img/aptana_debug_exe_1.jpg
        :width: 450px

デバッグができるようになりました。

    .. image:: ./img/aptana_debug_exe_2.jpg
        :width: 450px

| Railsプロジェクトについては、プロジェクト右クリックで「サーバーのデバッグ」を選択すれば起動するはずですが、
| やってみたところ特に何も起きません。。
| workspace下の.logを見てもエラーが出ていません。。
| WEBrickが起動している形跡もないし(localhost:3000で接続できなかったので)、、困った。。
|
| ちょっと悩んで、「サーバービューからデバッグボタンで起動」というのを思い出して試してみたら、やっと原因ぽいエラーが表示されました。

    .. image:: ./img/aptana_debug_rails_error.jpg
        :width: 600px

| ググってみると、以下のソースが見つかりました。
|
| http://stackoverflow.com/questions/19704541/rails-4-0-project-not-starting-through-aptana-studio-3
| http://beautifulajax.dip.jp/?p=23
|
| Rails4からフォルダ構成が変わったことに、Aptanaが対応できていないようです。
| AptantaのJIRAにも `issue <https://jira.appcelerator.org/browse/APSTUD-8062?page=com.xiplink.jira.git.jira_git_plugin:git-commits-tabpanel>`_ がアップされていますが、
| 2014/2/21現在では優先度LowのOpen状態なので、すぐに対応されるというのは期待できそうにないです。
| 前述のソースも、解決策としては
| ``「とりあえずscriptディレクトリ作って、binフォルダの下のファイルを置けばいいんじゃない？」``
| 的な内容だったので、そうすることにします。


ファイルが2箇所に分散するのはイヤなので、ディレクトリを作成するときにbinへのリンクを張ることにします。

    .. image:: ./img/aptana_script_dir_link.jpg
        :width: 600px


| フォルダ作成後に、プロジェクト右クリックで「サーバーのデバッグ」を選択するとWEBrickが起動しました。
| 適当なControllerにデバッグポイント張ってブラウザからアクセスしてみると、、、

    .. image:: ./img/aptana_debug_rails_exe.jpg
        :width: 600px

デバッグできました！
