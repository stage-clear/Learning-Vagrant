# 2. はじめての Vagrant マシン

2.1 起動と実行
--------------

```bash
# 起動
$ vagrant init precise64 http://files.vagrantup.com/precise64.box

# 実行
$ vagrant up

# SSH接続
$ vagrant ssh
```


2.2 Vagrantfile
----------------

```ruby
Vagrant::Config.run do |config| #1
  config.vm.box = 'precise64' #2

  config.vm.share_folder 'v-root', '/vagrant', '.' #3

  config.vm.provision 'shell' do |s| #4
    s.path = 'script.sh'
  end
end #5
```

1. Vagrant の設定ブロックの始まり
2. 変数の代入
3. 設定ディレクティブの例。 `share_folder` は関数呼び出しで3つのパラメータを呼ぶ
4. `provision` も関数呼び出し。追加の設定ブロックが `do` と `end` を使って開かれている


2.3 V1 と V2 の設定
-------------------

__V1 : 1.0.x__

```ruby
Vagrant::config.run do |config|
  # V1の設定
end
```

__V2 : 2.0.x__

```ruby
Vagrant.configure('2') do |config|
  # V2の設定
end
```


2.4 ボックス
------------

```ruby
Vagrant::Config.run do |config|
  config.vm.box = 'precise64';
  config.vm.box_url = 'http://files.vagrantup.com/precise64.box';
end
# precise64 はべアボーンの64bit版 Ubuntu 12.04LTS のイメージ
```

- `config.vm.box` : ボックスの名前
- `config.vm.box_url`: 必要なボックスがなかった場合にURLから解決する

ボックスの管理は、 `vagrant box` コマンドで行います。
（プロジェクトとは関係のない範囲で動作する）

- `$ vagrant box add` : ボックスを追加する
- `$ vagrant box list` : インストールされているボックスの一覧
- `$ vagrant box remove` : ボックスの削除


2.5 起動
--------

Vagrantfile ができたら起動します。

```sh
$ vagrant up
```

__バージョン管理と .vagrant/__

`.vagrant/` ディレクトリには、ゲストマシンのID、ロック、設定などが保存されます。
これらは、`vagrant up` の呼び出しに対応するものなので、 `.vagrant/` ディレクトリは、バージョン管理システムには無視させるべきです。
これをコミットしてしまうと、Vagrant の仮想マシンが「迷子」になったり、これを誤って共有してしまうと、仮想マシンが壊れたりしてしまうことがあります。


2.6 仮想マシンでの作業
---------------------

Vagrant の環境ができたので、今度はその使い方を学んでいきましょう。
マシンの状態の検査、マシンへのアクセス、マシンとのファイル共有、ネットワーク経由でのマシンとのやりとり、マシンの自動的な停止と再構築を取り上げます。


### 2.6.1 Vagrant マシンの状態

```sh
$ vagrant status
```


### 2.6.2 SSH

```sh
$ vagrant ssh
```

* Windows で SSH を使うには、OpenSSH をインストールしておかなければいけない


### 2.6.3 共有ファイルシステム

Vagrant は、仮想マシンとホストマシン間で、ファイルやフォルダが同期できるよう、共有フォルダのセットアップをサポートしています。

共有ファイルシステムは、`vagrant destroy` によって廃棄されないファイルの保存場所も提供してくれます。
ゲストマシンが破棄された場合でも、この共有システムに保存されたファイルは削除されないので、Vagrantfile に共有ファイルシステムのマッピングが保持されているかぎり、`vagrant up` で次にマシンが生成されたときにも利用できるのです。


```sh
# デフォルトの共有フォルダは /vagrant/
vagrant@precise64 ~$ ls /vagrant/
Vagrantfile
```

デフォルトの共有フォルダの場所は、Vagrantfile で上書きできます。

```sh
Vagrant::Config.run do |config|
  # ...

  config.vm.share_folder 'v-root', '/foo', '.'
  # config.vm.share_folder 'v-root', [Guest folder], [Host folder]
end
```

`config.vm.share_folder` はいくつかのパラメータをとります。

- 共有フォルダの識別子。ここでは `v-root` を指定することで、Vagrant がセットアップするデフォルtの共有フォルダが上書きされています。
- `/foo` はゲストマシン内にフォルダが置かれるパスです。
- ホストマシンから共有されるフォルダのパス。この指定は相対パスでも絶対パスでも構いません。相対パスの場合、プロジェクトのルートに対する相対となる。

* __現在は、`config.vm.synced_folder` を使う__


### 2.6.4 基本的なネットワーキング

Vagrant が自動的に様々ネットワーキングのオプションを仮想マシンに対して設定してくれる。
これにより、Vagrant を使って仮想マシンと通信できるようになる。


__ポート 80 を公開してWebサービスにアクセスできるようにする__

```ruby
Vagrant::Config.run do |config|
  # ...

  config.vm.forward_port 80, 8080
end
```

このフォワードされたポートが動作していることを示すために、仮想マシン内で Web サーバーを起動して、ホストマシンのブラウザからアクセスする。

```sh
$ vagrant reload
$ vagrant ssh
vagrant@precise64:~$ cd /vagrant
vagrant@precise64:/vagrant sudo python -m SimpleHTTPServer 80
Serving HTTP on 0.0.0.0 port 80 ...
# /vagrant にあたるフォルダは任意のフォルダ
```


### 2.6.5 ティアダウン

Vagrant のティアダウンは非常にクールなものです。
Web の開発者誰でも、Webサーバー、データベースなどを止めるのを忘れて、家に帰ってしまうという失敗をしたことがあります。
Vagrant があれば、これは単に起こりえないことです。開発環境に関連するすべてのソフトウェアは、
1つの仮想マシンの中に隔離されているので、1つのコマンドを実行することで、適切にホストマシンをクリーンアップしてくれます。


#### サスペンド

ゲストマシンをサスペンドすると、現在の動作状況を保存した後、マシンは停止します。
そしてこのマシンは、この停止したときの状態から、後にリジュームすることができます。

サスペンドの主なメリットは、仕事の再開が非常に早くできること。
また、中断したところからそのまま再開することです。これは、Web サービス、データベース、インメモリキャッシュなどが、
すべてそのまま残されているということです。

サスペンドの欠点は、ゲストマシンが残るためにハードディスクの領域が取られてしまうこと。

```sh
# suspend
$ vagrant suspend

# リジューム
$ vagrant up
# あるいは vagrant resume
```

__vagrant up と vagrant resume の違い__

`vagrant resume` はサスペンドされていなかった場合はエラーを表示するが、`vagrant up` は状態に応じて適切なブートシーケンスを実行する


#### 停止

停止を使うと、ゲストマシンは通常のコンピュータと同様、シャットダウンされます。
その後は、電源ボタンを押すのと同様に、マシンは通常のブートアップのプロセスを踏んで再起動させることができます。

RAMは保存されていないので、作業を再開する場合は、Web サーバーやデータベースといった必要なプロセスは起動しなおさなければなりません。

```sh
$ vagrant halt

# マシンが正常に終了しなくてもかまわいない場合は
# --force フラグをわたしてやれば強制的にマシンを停止する
$ vagrant halt --force
```


#### 破棄

ゲストマシンを破棄すると、そのマシンはシャットダウンされ、ハードディスクや状態の管理ファイルを削除することによって、
その痕跡はすべてなくなります。ゲストマシンを破棄すれば、ホストマシンは `vagrant up` が呼ばれていなかったかのように、元の状態に戻ります。
共有フォルダにおいていなかったデータはすべて失われます。

```sh
$ vagrant destroy

# 破棄を実行する前に、確認を求めてきます
# 確認をスキップするには --force フラグを使います
vagrant destroy --force
```
