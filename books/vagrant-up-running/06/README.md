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
