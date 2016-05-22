# VagrantでWebアプリケーション構築入門

矢吹太朗『Webアプリケーション構築入門』（森北出版, 第2版, 2011）のための開発環境を**Vagrant**で構築する。

この書籍は，サーバ構築技術とアプリケーション構築技術を学べるものになっている。ここでは，書籍の方法を少し改良して，これらをなるべく別々に学ぶ方法を紹介する。書籍どおりの方法を試したい場合は，[書籍のサポートサイト](https://github.com/taroyabuki/webbook2)の[Ubuntuでの環境構築のまとめ](https://github.com/taroyabuki/webbook2/blob/master/ubuntu.md)を参照してほしい。

サーバ構築技術は，Vagrantで管理する仮想マシンのプロビジョニングによって学ぶ（Javaのサーバは例外で，ホスト側で動作する）。詳しくは，Vagrantの概要を別に学んだ後で，以下の第2章についてのメモと，`Vagrantfile`と`provisioning.sh`を読んでほしい。仮想マシンはUbuntu 14.04（32-bit）を使う。Ubuntu 16.04でないのは、2016年5月時点ではUbuntu 16.04の動作に不明な部分があるため，32-bit版を使うのは，多くの環境で使えるようにしたいためである。

アプリケーション構築技術は，本書の3章以降で学ぶ。以下に記述する2章の準備を済ませた後で，3章以降を実践してほしい。

## 第2章

書籍のシステムの構成図は次のようなものであった。

![](https://raw.githubusercontent.com/taroyabuki/webbook2/master/update/system-textbook.png)

ここで構築するシステムは上を改良したもので，その構成図は次のようなものである。

![](https://raw.githubusercontent.com/taroyabuki/webbook2/master/update/system-vagrant.png)

この構成を使う利点は次のとおり。

* GUIが必要なウェブブラウザとNetBeansがホスト側で動くため，仮想マシンにはGUIが不要になる。その結果，動作が全体的に軽くなる。
* 仮想マシンの構築を完全に自動化できる。その結果，仮想マシンの作成や破棄が気軽にできるようになる。

### 2.2の代わり（その1）：Vagrantの準備

#### Windows 10

1. [Chocolatey](https://chocolatey.org/)をインストールする。
1. コマンドプロンプト（管理者）を再起動する。（スタートボタンを右クリック）
1. VagrantとVirtualBox，JDKをインストールする（すでにインストールされている場合は，以下のコマンドからそれを除くこと。一度アンインストールしてもよい）。（`rsync`は`ssh`クライアントのため）

```
cinst -y git rsync vagrant virtualbox jdk
```

参考：[Chocolateyを使った環境構築の時のメモ](http://qiita.com/konta220/items/95b40b4647a737cb51aa)

（管理者でない）コマンドプロンプトを起動する。コマンドプロンプトのタイトルバーを右クリックし，プロパティの設定で「簡易編集モード」を有効にしておく（範囲選択し`Enter`キーでコピー，右クリックで貼り付けができるようになる）。

Vagrantのためのフォルダ`c:\vagrant`を作る。（別の場所でもよいが，以下の説明はこれを前提にしている。）

```
c:
cd \
mkdir vagrant
```

#### Mac OS X

1. 端末を起動する。
1. [Homebrew](http://brew.sh/index_ja.html)をインストールする。
1. [Homebrew Cask](https://caskroom.github.io/)をインストールする。
1. JDK，VirtualBox，Vagrantをインストールする。

```bash
brew cask install java virtualbox vagrant
```

参考：[Homebrew CaskでVagrantの環境構築メモ](http://qiita.com/moomoo-ya/items/30f9ffdb7dd34caf9ec0)

Vagrantのためのフォルダ`~/vagrant`を作る。（別の場所でもよいが，以下の説明はこれを前提にしている。）

```bash
cd
mkdir vagrant
```

### 2.2の代わり（その2）：仮想マシン

#### 準備

仮想マシンの再構築を高速にするために，キャッシュを導入する。

```
vagrant plugin install vagrant-cachier
```

**（Macのみ）**起動の前後でコマンドを実行するためのプラグインを導入する。

```bash
vagrant plugin install vagrant-triggers
```

Vagrantのためのフォルダに移動し，VagrantのBoxを用意する。

Windowsなら次のとおり。

```
c:
cd \vagrant
git clone https://github.com/taroyabuki/webbook2server.git
```

Macなら次のとおり。

```bash
cd ~/vagrant
git clone https://github.com/taroyabuki/webbook2server.git
```

#### 起動

仮想マシンを起動する。（初期設定を行うため，一回目は時間がかかる。）

```
cd webbook2server
vagrant up
```

#### 再起動と停止，削除

* 再起動：vagrant reload
* 停止：vagrant halt
* 削除：vagrant destroy

すべてやり直したいときは，`vagrant destroy`の後でフォルダ`webbook2server`を削除する。

#### 接続と切断

Windowsでは，ホストにいるときのプロンプトは`C:\vagrant\webbook2server>`。そこで`vagrant ssh`として，ゲストに接続する。

```
C:\vagrant\webbook2server>vagrant ssh
```

Macでは，ホストにいるときのプロンプトは`マシン名:vagrant ユーザ名$ `。そこで`vagrant ssh`として，ゲストに接続する。

```
マシン名:vagrant ユーザ名$ vagrant ssh
```

ゲストにいるときのプロンプトは`vagrant@vagrant-ubuntu-trusty-32:~$ `。そこで`exit`として，切断する。

```
vagrant@vagrant-ubuntu-trusty-32:~$ exit
```

#### 動作確認

[http://localhost/info.php](http://localhost/info.php)や[http://localhost/phpmyadmin/](http://localhost/phpmyadmin/)（ユーザ名：`root`，パスワード：`pass`）にアクセスできることを確認する。

#### 2.3.1の代わり：ドキュメントルート

Apache HTTP Serverのドキュメントルートは以下のとおり。

* ホスト側：Windowsなら`c:/vagrant/webbook2server/html`，Mac OS Xなら`~/webbook2server/html`
* ゲスト側：`/var/www/html`

この他に，次のフォルダもホストとゲストで共有している。

* ホスト側：Windowsなら`c:/vagrant/webbook2server`，Mac OS Xなら`~/webbook2server`
* ゲスト側：`/vagrant`

PHPのエラーに関する`php.ini`の設定は変更済み。

### 2.4の代わり：ウェブブラウザと統合開発環境

#### ウェブブラウザ

ブラウザは，FirefoxやChrome，Internet Explorer，Edge，Safariなど，何でもよい。

#### 統合開発環境（NetBeans）

[https://netbeans.org/downloads/?pagelang=ja](https://netbeans.org/downloads/?pagelang=ja)から「PHP」または「すべて」をダウンロードし，インストールする。

### 2.5

#### 2.5.1

Javaのプロジェクトは「Maven」の「Webアプリケーション」を使う（プロジェクト名を`javaweb`にすれば，あとはデフォルトでよい）。

![](https://raw.githubusercontent.com/taroyabuki/webbook2/master/update/maven.png)

以下のライブラリをここで用意する。

* Apache Commons Lang（書籍では6.3.1項でインストールする）
* MySQL Connector/J（書籍では8.2.2項でインストールする）

そのために，「プロジェクト・ファイル」の中にある`pom.xml`の`dependencies`要素の中に以下を追記する。（よくわからなければ，ここで配布している`pom.xml`で全体を置き換えてもかまわない。ちなみに，これらの記述は[http://mvnrepository.com/](http://mvnrepository.com/)で見つけられる。）

```xml
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>6.0.2</version>
</dependency>
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.4</version>
</dependency>
```

#### 2.5.2

プロジェクトのソースフォルダは，Windowsなら`C:\vagrant\webbook2server\html\phpweb`，Mac OS Xなら`~/webbook2server/html/phpweb`とする。

![](https://raw.githubusercontent.com/taroyabuki/webbook2/master/update/phpweb.png)

## 第3章

HTML Validatorのインストールは不要。

参照：[サポートサイトの第3章部分](https://github.com/taroyabuki/webbook2/tree/master/src/03)

## 第4章

Firebugのインストールは不要。

参照：[サポートサイトの第4章部分](https://github.com/taroyabuki/webbook2/tree/master/src/04)

### 4.1

NetBeansのプロジェクト`phpweb`内に`alert.html`を作り，ブラウザで[http://localhost/phpweb/alert.html](http://localhost/phpweb/alert.html)を開いて確認する。

ホストとゲストのフォルダは同期しているから，ゲスト側で次のようにしてファイルを作ることもできる。

```bash
cd /var/www/html/phpweb
wget https://raw.githubusercontent.com/taroyabuki/webbook2/master/src/04/alert.html
```

## 第5章

* Firebugのインストールは不要。
* PEAR HTTP_Requestはインストール済み。

参照：[サポートサイトの第5章部分](https://github.com/taroyabuki/webbook2/tree/master/src/05)

## 第6章

Apache Commons Langのインストールは不要（上述のとおり`pom.xml`を修正していれば）。

参照：[サポートサイトの第6章部分](https://github.com/taroyabuki/webbook2/tree/master/src/06)

## 第7章

* MySQLはインストール済み。（ユーザ`root`のパスワードは`pass`）
* phpMyAdminはインストール済みであり，[http://localhost/phpmyadmin/](http://localhost/phpmyadmin/)で利用できる。

参照：[サポートサイトの第7章部分](https://github.com/taroyabuki/webbook2/tree/master/src/07)

## 第8章

MySQL Connector/Jのインストールは不要（上述のとおり`pom.xml`を修正していれば）

参照：[サポートサイトの第8章部分](https://github.com/taroyabuki/webbook2/tree/master/src/08)

データベースへのアクセス権は，データベースのサーバとクライアントが別々のマシンになることを考慮し，次のようにする。

実際に，別のホストマシン等から仮想マシン内のデータベースにアクセスするためには，`my.cnf`において`bind-address`の値を`0.0.0.0`にしておく必要があるが，その設定は`provision.sh`内で済ませてある。

## 第9章

* lhaのインストールは不要。
* nkfはインストール済み。

参照：[サポートサイトの第9章部分](https://github.com/taroyabuki/webbook2/tree/master/src/09)
