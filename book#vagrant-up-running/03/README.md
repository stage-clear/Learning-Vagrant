# 3. Vagrant 仮想マシンのプロビジョニング

開発中のすべてのソフトウェアがゲストにインストールされていなければならない。
これを実現する方法は2つあります。

1. ボックス自体にソフトウェアを入れ込んでおく
2. 開発環境の生成プロセスの一部として、自動的にソフトウェアをインストールする

後者がプロビジョニングです。

ブートされたシステムにソフトウェアをインストールすることは、プロビジョニングと呼ばれ、
多くの場合は、シェルスクリプト、設定管理システム、あるいはコマンドラインを主導で使って
行われます。

Vagrant は自動的なプロビジョニングをサポートしている。
インストール直後から、シェルスクリプト、Chef、Puppet によるプロビジョニングをサポート
しています。


Apache をセットアップして、性的なファイルを Vagrant の共有フォルダから配信できるよう
にします。
Apache が自動的に起動され、共有フォルダのファイルを提供してくれるように設定します。
この基本的な例における開発者は、ゲストマシンにSSHする必要すらなく、完全にホストの環境
内だけで作業できるようになります。


## 3.1 自動プロビジョニングを行う理由

自動プロビジョニングには、主に3つのメリットを提供します。

1. 利用の容易さ
2. 再現性
3. 開発環境と実働環境の同一性の向上


### 3.2 サポートされているプロビジョナ

- シェルスクリプト
- Chef
- Puppet


### 3.3 手動での Apache のセットアップ

Apache を手動でセットアップしてみることにしましょう。
これは通常、自動化を行う前に必要になる、最初のステップです。
自動化できるようになるためには、やらなければならないことを知っておく必要があります。

```ruby
Vagrant::Config.run do |config|
  config.vm.box = 'precise64'
  config.vm.forward_port 80, 8080
end
```

Vagrantfile ができたら、`vagrant up` !

```sh
$ vagrant up
```

マシンが立ち上がって動作し始めたら、SSH。

```sh
$ vagrant ssh
```

使っているベースボックスは Ubuntu のマシンなので、ソフトウェアをインストールするのに使う
パッケージマネージャは Apt です。
まず、`apt-get update` を実行しなければなりません。

```sh
vagrant@precise64:~$ sudo apt-get update
```

パッケージのインデックスが更新できたら、Apache をインストールします。
これは `apt-get install` で行えます。

```sh
vagrant@precise64:~$ sudo apt-get install apache2
```

デフォルトでは、Ubuntu はシステムの起動時に Apache が立ち上がるように設定し、
インストールしたままの Apache は、`/var/www/` にあるファイルを配信します。
設定を簡単にするために、`/var/www` を、デフォルトの共有ディレクトリである `/vagrant` への
シンボリックリンクに変更しましょう。

```sh
vagrant@precise64:~$ sudo rm -rf /var/www
vagrant@precise64:~$ sudo ln -fs /vagrant /var/www
```

これで、`http://localhost:8080` でホストマシンのブラウザからアクセスできる。
SSH を終了して、`index.html` を作成する

```sh
vagrant@precise64:~$ logout
Connection to 127.0.0.1 closed.
$ echo '<strong>Hello</strong>' > index.html
```

仮想マシンから配信されました。

この手作業のプロセスを、`vagrant up` を実行するたびにやらなければならないとしたら、それは大変苦痛なことです。
しかし、自動化プロビジョニングを行えば、この苦痛を味わずに済みます。


### 3.4 自動化プロビジョニング

Apache の手動セットアップのプロセスは一通りやってみたので、そのプロセス全体を自動化
するのに必要なステップはもうわかっていて、その内容も理解できています。
Chef や　Puppet を使ったことがなく、いずれかを使おうと計画しているのであれば、
簡単なことはシェルでまずやってみて、それを Chef や Puppet に変換してみるのは、いい練習になります。


### 3.4.1 シェルスクリプト

シェルスクリプトは、最も基本的な形であれば、実行するコマンドのリストに過ぎません.


```sh
#!/usr/bin/env bash

echo 'Installing Apache and setting it up...'
apt-get update >/dev/null 2>&1
apt-get install -y apache2 >/dev/null 2>&1
rm -rf /var/www
ln -fs /vagrant /var/www
```

コマンドラインに入力した内容とは少し異なっている部分があります。
最初の行はシェバンとよばれるもので、ファイルのこれ以降の内容を実行するのに使うシェル
を指定するものです。ここで使うのは `bash` です

`-y` フラグは、`apt-get` に対し、すべてのプロンプトに自動的に `yes` を返すように
指示しています。自動化プロビジョニングにおいては人が介入することがないので、もし
`apt-get` が確認を求めてきたらな、このスクリプトはいつまで経っても完了しないだけに
なるかあるいはクラッシュすることになるでしょう。

シェルスクリプトができたら、以下の行を、Vagrantfile のどこかに追加してください。

```ruby
config.vm.provision 'shell', path: 'provision.sh'
```

クリーンな状態から確認するため、マシンを破棄します。

```sh
$ vagrant destroy
```

最後に `vagrant up` を実行して、自動化の魅力を味わいましょう。

```sh
$ vagrant up
```


### 3.4.2 Chef

Vagrantfile に Chef のプロビジョナを設定する

```ruby
config.vm.provision 'chef_solo', run_list: ['vagrant_book']
```

Vagrant のデフォルトで見に行くクックブックは、プロジェクトのディレクトリに対する
相対パスの `./cookbooks` ディレクトリである

```ruby
Vagrant::Config.run do |config|
  config.vm.box = 'precise64'
  config.vm.forward_port 80, 8080
  config.vm.provision 'chef_solo', run_list: ['vagrant_book']
end
```

実際のクックブックとレシピを作成してみましょう。

```ruby
# default.rb
execute 'apt-get update'
package 'apache2'
execute 'rm -rf /var/www'
link '/var/www' do
  to '/vagrant'
end
```

プロジェクトディレクトリのレイアウト。

```txt
$ tree
.
├── Vagrantfile
└── cookbooks
    └──vagrant_book
        └── recipes
             └── default.rb

3 directries, 2 files
```

```sh
$ vagrant destroy
$ vagrant up
```


### 3.4.3 Puppet

Vagrantfile に Puppet のプロビジョナを設定する

```sh
config.vm.provision 'puppet'
```

```
# manifests/default.pp
exec { 'apt-get update':
  command => '/usr/bin/apt-get update',
}
package { 'apache2'
  require => Exec['apt-get update'],
}

file { '/var/www':
  ensure => link,
  target => '/vagrant',
  force => true,
}
```

プロジェクトディレクトリ。

```txt
$ tree
.
├── Vagrantfile
└── manifests
    └── default.pp

1 directries, 2 files
```

```sh
$ vagrant up
```


## 3.5 複数のプロビジョナ

プロビジョナは、1つしか使えないわけではありません。複数の `config.vm.provision`
ディレクティブを Vagrantfile で指定すれば、指定された順序でプロビジョニングします。

```ruby
Vagrant::Config.run do |config|
  config.vm.box = 'precise64'

  config.vm.provision 'shell', install: 'apt-get update'
  config.vm.provision 'puppet'
  # ... and so on
end
```

マシンのブートストラップ時にはシェルスクリプトを利用し、Chefのクックブックをテストする
のに Chef のプロビジョナを使い、リロードの時にだけその Chef のプロビジョナを実行したい、
といった場合に便利です。`up` と `reload` のどちらでも以下のような指定できます

```ruby
$ vagrant up --provision-with=chef
```

もちろん、この場合のパラメータには、使っているプロビジョナの名前をどれでも使うことがで
きます。


## 3.6 「プロビジョニングしない」モード

```sh
$ vagrant up --no-provision
```


### 3.7 詳細なプロビジョナの利用方法

自動化プロビジョナの基本的な使い方を見たので、今度はそれぞれのプロビジョナをどのように
設定し、チューニングすれば、やりたいことをぴったりこなしてくれるのか、その複雑な詳細に
入っていきましょう。


### 3.7.1 シェルスクリプト

シェルスクリプトのユニークさは、非常にシンプルなものから、信じられないほど複雑なものに
まで及ぶ、その柔軟性にあります。
