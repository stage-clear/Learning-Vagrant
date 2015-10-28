# 6. ボックス

ボックスは、Vagrantが作成する環境のベースとなるイメージです。

## 6.1 ボックスを使う理由

- ボックスは、`vagrant up` のたびにOSをインストールしないで済むようにするための仕組み
- `vagrant destroy` と `vagrant up` で新しい環境をてにするのに要する時間は数分程度
- ボックスはポータブルで、MacOSX で作成されたボックスは Win でも Linux でも正しく動作するでしょう

## 6.2 ボックスのフォーマット

- ボックスファイルのフォーマットは、単なる `tar` ファイル
- gzip で圧縮されていることもある
- `.box` という拡張子、Vagrant で使用されることを明確にするため

VirtualBox 用のボックスファイルの内容は、VirtualBox の仮想マシンのエクスポートの出力
に過ぎません。precise64 のボックスファイルを展開してみれば、以下のような内容が出てくる
でしょう。

```
$ tree
.
├── Vagrantfile
├── box-disk1.vmdk
├── box.ovf
└── metadata.json

0 directories, 4 files
```

`VMDK` と `OVF` ファイルは、VirtualBox のエクスポートで得られるファイル。です。

- `VMDK` : 圧縮されたハードディスク
- `OVF` : マシンを動作させる仮想ハードウェアを記述するファイル
- `Vagrantfile` : 通常の Vagrantfile です
- `metadata.json` : Vagrant に対してこのボックスが使うシステム(この場合はVirtualBox)を知らせる


## 6.3 Vagrant を使用しない基本的なボックスの管理

- Vagrant は、ボックスをユーザーごとにグローバルに管理します
- プロジェクト用の Vagrantfile がプロジェクトごとに管理されるのとは対照的
- `vagrant up` `vagrant ssh` などのコマンド群とは異なり、ボックスを管理するコマンドは、すべての Vagrant の環境に影響します
- ボックスは Vagrant の中で（ユーザー任意の）論理名にマップされる

* * *

__どれだけグローバルか__

Vagrant は、デフォルトですべてのグローバルな状態を `~/.vagrant.d` フォルダに格納し
ます。これにはボックス群も含まれます。
これはすなわち、Vagrant がボックス群を「グローバル」に管理すると言う場合、これは実際に
は、デフォルトではユーザーごとに管理するということです。
環境変数 `VAGRANT_HOME` で変更可能。

* * *

すべてのボックスの管理は、`vagrant box` コマンドを通じて行います。

```sh
$ vagrant box
Usage: vagrant box <subcommand> [<args>]
Available subcommands:
     add
     list
     outdated
     remove
     repackage
     update
For help on any individual subcommand run `vagrant box <subcommand> -h`
```

- `vagrant box add` : ボックスの追加。　
  `config.vm.box_url` は環境内に指定したボックスがなければ `add` を使ってダウンロードする
- `vagrant box list` : インストールしたボックスの一覧を表示
- `vagrant box remove` : ボックスの削除
- `vagrant box repackage` : カレントディレクトリに `package.box` を生成する
  このファイルはダウンロードしたオリジナルのボックスファイルと同じものです


## 6.4 既存環境からの新しいボックスの生成

- 新しいボックスを生成する最も簡単な方法は、既存の Vagrant の環境を出発点とすること

既存の状態からボックスを構築するには、まずは既存の環境がひつようになるので `vagrant up`
しましょう

```sh
$ vagrant up
```

デフォルトでインストールされていない htop をインストールする

```sh
$ vagrant ssh
vagrant@precise64:~$ sudo apt-get install -y htop
# ... Done
vagrant@precise64:~$ logout
```

次に、`vagrant package` を実行します

```sh
$ vagrant package
```

作成される `package.box` は、動作中の既存の Vagrant 環境をベースとする新しいボックス
です。このボックスを `add` して、それを元に新しい Vagrant の環境を動作させれば、htop
はインストール済みのはずです。

* * *

__package と repackage__

Vagrant には `vagrant package` と `vagrant repackage` というコマンドがあります。

- `vagrant package` :　その時点で動作している Vagrant の環境を、再利用可能なボックス
  にパッケージ化するもの
- `vagrant repackage` : 以前に `add` されたボックスをボックスファイルへとパッケージ
  し直して、配布できるようにします

* * *

## 6.5 スクラッチから新しいボックスを生成


====== スクラッチから作成することはないのでメモは省略 ======
