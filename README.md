# Cloud Foundry PHP Buildpack

注意：この翻訳は更新しません。 最新情報は英語 https://github.com/cloudfoundry/php-buildpack を参照してください。


## CF Slack 
[![CF Slack](https://www.google.com/s2/favicons?domain=www.slack.com) CF Slack Slackに参加しよう](https://cloudfoundry.slack.com/messages/buildpacks/)

クラウド・プロバイダーや独自のインスタンスなどのCloud FoundryベースのシステムにPHPアプリケーションをデプロイするためのビルドパックです。


## ビルドパックのユーザーマニュアル

公式ビルドパックのドキュメントは、[php buildpack docs]（http://docs.cloudfoundry.org/buildpacks/php/index.html）にあります。


## ビルパックの構築

GitHub からサブモジュールを取得したことを確認するには次のコマンドを実行する

~~~
git submodule update --init
~~~

最新のビルドパックの依存パッケージを取得する

~~~
BUNDLE_GEMFILE=cf.Gemfile bundle
~~~


ビルドパックをビルドする

~~~
BUNDLE_GEMFILE=cf.Gemfile bundle exec buildpack-packager [ --uncached | --cached ]
~~~
                    

Cloud Foundryの中で実行する

buildpackをCloud Foundryインスタンスにアップロードし、オプションで名前を指定します。 **Bluemixでは利用できません**

~~~
cf create-buildpack custom_php_buildpack php_buildpack-cached-custom.zip 1
cf push my_app -b custom_php_buildpack
~~~
          

## 寄贈

ガイドラインを[ここ](https://github.com/cloudfoundry/php-buildpack/blob/develop/CONTRIBUTING.md)から探してください。


## 統合テスト

ビルドパックは統合テストを実行するために[Machete](https://github.com/cloudfoundry/machete)フレームワークを使用します。

ビルドパックをテストするには、ビルドパックのディレクトリから次のコマンドを実行します。 **CloudFoundry CLIが必要です**

~~~
BUNDLE_GEMFILE=cf.Gemfile bundle exec buildpack-build
~~~


## ユニットテスト

~~~
./run_test.sh
~~~



## 要件

[PyEnv] - Python 2.6.6を簡単にインストールできます。Python 2.6.6は、CloudFoundryのステージング環境で使用できるものと同じバージョンです。

[virtualenv]＆[pip] - buildpackは virtualenv と pip を使って必要なパッケージをセットアップします。 これらはユニットテストで使用され、ビルドパック自体では必要ありません。


## セットアップ

~~~
git clone https://github.com/cloudfoundry/php-buildpack
cd php-buildpack
python -V  # should report 2.6.6, if not fix PyEnv before creating the virtualenv
virtualenv `pwd`/env
. ./env/bin/activate
pip install -r requirements.txt
~~~




## プロジェクトの構成

プロジェクトは以下のディレクトリに分かれています：

* bin：compile, release, detect のスクリプトが含まれています。
* defaults：デフォルト設定
* docs：プロジェクトのドキュメント
* extensions：非コア拡張
* env：virtualenv環境
* lib：コア拡張、ヘルパーコード、buildpackユーティリティを含む
* scripts：compile, release, detect に必要なPythonスクリプトが含まれています。
* test：テスト・スクリプトとテスト・データ
* run_tests.sh：完全な一連のテストを実行するための便利なスクリプト



## ビルドパックの理解

ビルドパックを理解する最も簡単な方法は、スクリプトの流れをトレースすることです。 buildpackシステムは、buildpackによって提供されたcompile, release そして、detect スクリプトを呼び出します。 これらは、一般的にbinディレクトリの下にあります。 スクリプト・ディレクトリの対応するPythonスクリプトにリダイレクトされます。

これらのうち、detectスクリプトと releaseスクリプトは、簡単でビルドパックで必要とされる最小の機能を提供します。 compileスクリプトはもっと複雑ですが、このように動作します。

* 負荷設定
* WEBDIRディレクトリを設定
* buildpack utilsとコア拡張（HTTPD、Nginx＆PHP）のインストール
* 他の拡張機能をインストール
* 書き換えスクリプト、および、インストールスクリプト のインストール
* 実行時環境とプロセスマネージャをセットアップ
* startup.shスクリプトを生成

一般的に、ビルドパック自体を変更する必要はありません。 代わりに拡張機能を作成が必要です。


## 拡張機能

ビルドパックは、拡張機能に大きく依存しています。 拡張機能はPythonメソッドのセットで、単にステージングプロセス中にさまざまな時に呼び出されます。


## 作成方法

拡張機能を作成するには、単にフォルダを作成します。 フォルダの名前が拡張子になります。 そのフォルダの中に、extension.pyという名前のファイルを作成します。 そのファイルにはコードが含まれます。 そのファイルの中に、拡張メソッドと必要なコードを追加します。

### configure(ctx):

~~~
def configure(ctx):
    pass
~~~


configureメソッドは、拡張機能の作成者に、拡張機能が実行される前にビルドパックの設定を調整する機会を与えます。 このメソッドは、ビルドパックのライフサイクルの早い段階で呼び出されます。このメソッドを使用する場合は、この点に注意してください。 このメソッドの目的は、拡張機能の作成者がPHP、Webサーバー、またはそれらのコンポーネントがインストールされる前の別の拡張機能の設定を変更できるようにすることです。

このメソッドを使用する例は、インストールされるPHP拡張モジュールのリストを調整することです。

このメソッドはbuildpackコンテキストである1つの引数をとります。 コンテキストを編集してビルドパックの状態を更新することができます。 戻り値は無視/不要です。

### preprocess_commands(ctx):

~~~
def preprocess_commands(ctx):
    return ()
~~~


preprocess_commandsメソッドは、拡張機能の作成者に、サービスに先立って実行すべきコマンドのリストを提供する機能を提供します。 これらのコマンドは、ステージング環境ではなく実行環境で実行されるため、すばやく実行して完了する必要があります。 これらのコマンドの目的は、拡張者の作者に環境に合わせるための最後のコードを実行する機会を与えることです。

例として、これは、コア拡張機能によって、構成ファイルを実行時環境固有の情報で書き換えることによって使用されます。

このメソッドはコンテキストを引数として取り、タプルのタプル（つまり、実行するコマンドのリスト）を返す必要があります。


### service_commands(ctx):

~~~
def service_commands(ctx):
    return {}
~~~

service_commandsメソッドは、拡張の作成者に、実行する必要のある一連のサービスを提供します。 これらのコマンドは実行され、実行を続行する必要があります。 サービスが終了すると、プロセスマネージャーは他のすべてのサービスを停止し、Cloud Foundryによってアプリケーションが再開されます。

このメソッドはコンテキストを引数として取り、実行するサービスの辞書を返す必要があります。 キーはサービス名でなければならず、値はコマンドと引数であるタプルでなければなりません。


### service_environment(ctx):

~~~
def service_environment(ctx):
    return {}
~~~


service_environmentメソッドは、拡張機能の作成者に、サービスで設定および使用できる環境変数を提供する機能を提供します。

このメソッドは、引数としてbuildpackコンテキストをとり、サービス（service_commandsを参照）が実行される環境に追加される環境変数の辞書を返す必要があります。

キーは変数名でなければならず、値は値でなければなりません。値は文字列にすることができます。この場合、環境変数には文字列の値を設定するか、リストにすることができます。

リストの場合、内容は文字列に結合され、パス分離文字（Unix / Linuxでは '：'、Windowsでは ';'）で区切られます。同一または異なる拡張機能によって複数回設定されたキーは、同じパス区切り文字を使用して1つの環境変数に自動的に結合されます。これは、2つの拡張が両方とも同じ変数に寄与したい場合（LD_LIBRARY_PATHなど）に役立ちます。

環境変数は、設定されると評価されないことに注意してください。これは、実行環境とは異なるステージング環境に設定されているため動作しません。つまり、PATH = $ PATH：/ new / pathやNEWPATH = $ HOME / some / pathのようなことはできません。この問題を回避するために、ビルドパックは処理される前に環境変数ファイルを書き換えます。このプロセスは、@ <env-var>マーカーを実行環境の環境変数の値に置き換えます。したがって、PATH = @ PATH：/ new / pathまたはNEWPATH = @ HOME / some / pathを実行すると、サービスは正しく設定されたPATHまたはNEWPATHで終了します。


### compile(install):

~~~
def compile(install):
    return 0
~~~


compileメソッドは主な方法であり、拡張作成者はそのロジックの大部分を実行する必要があります。 このメソッドは、拡張機能をインストールする際にbuildpackから呼び出されます。

このメソッドにはインストーラビルダオブジェクトである1つの引数が与えられます。 このオブジェクトは、パッケージ、設定ファイルのインストール、コンテキストへのアクセス（HTTPD、Nginx、PHP、Dynatrace、NewRelicなどのコア拡張を参照してください）に使用できます。 成功すると0を返し、失敗した場合はそれを返します。 必要に応じて、拡張機能は例外を発生させることができます。 これはまた、失敗を通知し、何かが失敗した理由の詳細を提供することができます。



## メソッドの順序

ビルドパックがエクステンション内のメソッドを呼び出すためにどのような順序で使用するかを知ることは時々役に立ちます。 次の順序で呼び出されます。

1. configure
2. compile
3. service_environment
4. service_commands
5. preprocess_commands
    
      
### メソッドのコール例

ここに拡張の例を示します。 技術的には正しいものの、実際には何もしません。

次が、そのディレクトリです。

~~~
$ ls -lRh
total 0
drwxr-xr-x  3 daniel  staff   102B Mar  3 10:57 testextn

./testextn:
total 8
-rw-r--r--  1 daniel  staff   321B Mar  3 11:03 extension.py
~~~


次が、そのコードです。

~~~
import logging

_log = logging.getLogger('textextn')

# Extension Methods
def configure(ctx):
    pass

def preprocess_commands(ctx):
    return ()

def service_commands(ctx):
    return {}

def service_environment(ctx):
    return {}

def compile(install):
    return 0
~~~


## Tips

1. ビルドパックの他の部分と一貫性を持たせるために、拡張モジュールは標準ロギングモジュールをインポートして使用する必要があります。これにより、ビルドパックの残りの部分の出力に拡張出力を組み込むことができます。

2. buildpackは、buildpackとアプリケーションに含まれるすべての拡張機能を実行します。特定の拡張機能を無効にするメカニズムはありません。したがって、拡張機能を記述するときは、その機能を有効/無効にするための方法をいくつか作成する必要があります。この例については、NewRelic拡張モジュールを参照してください。

3. 拡張機能に設定が必要な場合は、拡張機能に含める必要があります。 defaults / options.jsonファイルはビルドパックとそのコア拡張です。これの例は、NewRelicビルドパックを参照してください。

4. 拡張モジュールには独自のテストモジュールが必要です。これは一般的に、tests / test_ <extension_name> .pyという形式です。

5. bosh-liteを実行します。テストを高速化し、必要に応じて手動で環境を検査できるようにします。

6. バイナリ用のローカルWebサーバーを実行します。ダウンロード時間が大幅に短縮されます。

7. テスト、テスト、テストをもう一度行います。コードと拡張機能のユニットテストと統合テストを作成します。これにより、コードに対して迅速かつ正確なフィードバックが得られます。また、将来的に変更を加えることが容易になり、物事を破壊していないと確信することができます。

8. flake8でコードをチェックします。このリンティングツールは、問題を迅速に検出するのに役立ちます。


[PyEnv]:https://github.com/yyuu/pyenv
[virtualenv]:http://www.virtualenv.org/en/latest/
[pip]:http://www.pip-installer.org/en/latest/
[required packages]:https://github.com/cloudfoundry/php-buildpack/blob/master/requirements.txt
[bosh-lite]:https://github.com/cloudfoundry/bosh-lite
[HTTPD]:https://github.com/cloudfoundry/php-buildpack/tree/master/lib/httpd
[Nginx]:https://github.com/cloudfoundry/php-buildpack/tree/master/lib/nginx
[PHP]:https://github.com/cloudfoundry/php-buildpack/tree/master/lib/php
[Dynatrace]:https://github.com/cloudfoundry/php-buildpack/tree/master/extensions/dynatrace
[NewRelic]:https://github.com/cloudfoundry/php-buildpack/tree/master/extensions/newrelic
[unit tests]:https://github.com/cloudfoundry/php-buildpack/blob/master/docs/development.md#testing


## ヘルプとサポート

Slackコミュニティの#buildpacksチャンネルに参加する [Slack community](http://slack.cloudfoundry.org/) 



## 報告の問題

このプロジェクトはGitHubを通じて管理されています。 何か問題が発生した場合、バグやビルドに関する問題が発生した場合は、問題をオープンしてください。


## アクティブな開発

プロジェクトのバックログは[Pivotal Tracker](https://www.pivotaltracker.com/projects/1042066)にあります

[Configuration Options]:https://github.com/cloudfoundry/php-buildpack/blob/master/docs/config.md
[Development]:https://github.com/cloudfoundry/php-buildpack/blob/master/docs/development.md
[Troubleshooting]:https://github.com/cloudfoundry/php-buildpack/blob/master/docs/troubleshooting.md
[Usage]:https://github.com/cloudfoundry/php-buildpack/blob/master/docs/usage.md
[Binaries]:https://github.com/cloudfoundry/php-buildpack/blob/master/docs/binaries.md
[php-info]:https://github.com/dmikusa-pivotal/cf-ex-php-info
[PHPMyAdmin]:https://github.com/dmikusa-pivotal/cf-ex-phpmyadmin
[PHPPgAdmin]:https://github.com/dmikusa-pivotal/cf-ex-phppgadmin
[Wordpress]:https://github.com/dmikusa-pivotal/cf-ex-worpress
[Drupal]:https://github.com/dmikusa-pivotal/cf-ex-drupal
[CodeIgniter]:https://github.com/dmikusa-pivotal/cf-ex-code-igniter
[Stand Alone]:https://github.com/dmikusa-pivotal/cf-ex-stand-alone
[pgbouncer]:https://github.com/dmikusa-pivotal/cf-ex-pgbouncer
[Apache License]:http://www.apache.org/licenses/LICENSE-2.0
[vcap-dev]:https://groups.google.com/a/cloudfoundry.org/forum/#!forum/vcap-dev
[support forums]:http://support.run.pivotal.io/home
[Composer support]:https://github.com/cloudfoundry/php-buildpack/blob/master/docs/composer.md
["offline" mode]:https://github.com/cloudfoundry/php-buildpack/blob/master/docs/binaries.md#bundling-binaries-with-the-build-pack
[phalcon]:https://github.com/dmikusa-pivotal/cf-ex-phalcon
[Phalcon]:http://phalconphp.com/en/
[composer]:https://github.com/dmikusa-pivotal/cf-ex-composer
[Proxy Support]:http://docs.cloudfoundry.org/buildpacks/proxy-usage.html





