# Vagrant の環境変数

## A.1 VAGRANT_CWD

- Vagrant の作業ディレクトリ
- デフォルトは、ユーザーのカレントディレクトリを使用する
- 作業ディレクトリが重要なのは、Vagrant が `Vagrantfile` を探す場所だから
- `Vagrantfile` 中の相対パスにも影響する

この環境変数が最も一般的に設定されるのは、Vagrant がスクリプティング環境から実行される
場合です。

```sh
#!/usr/bin/env bash

VAGRANT_CWD=/foo vagrant up
VAGRANT_CWD=/bar vagrant up
```

## A.2 VAGRANT_HOME

- Vagrant がグローバルな状態を保存するディレクトリ
- デフォルトは、`~/.vagrant.d`
- Vagrant のホームディレクトリは、ボックスなどが保存される場所

この環境変数の一般的なユースケースは2つあります。

1. 高速化のためにSSDをメインで使っている場合、速度が問題にならないような用途では、低速で大容量のディスクも使っているような場合に `VAGRANT_HOME` を大きい方に向ける
2. テストやスクリプティング用にVagrant をインストールする場合、Vagrant のホームとプロジェクトの `.vagrant` を合わせれば、Vagrant はクリーンにインストールされた状態になります


## A.3 VAGRANT_LOG

- Vagrant が記録するログメッセージの詳細度
- ログのレベルは、`debug` > `info` > `warn` > `error`


## A.4 VAGRANT_NO_PLUGINS

- `VAGRANT_NO_PLUGINS` に何らかの値を設定すると、Vagrant は一切プラグインをロードしなくなります


## A.5 VAGRANT_VAGRANTFILE

- Vagrant が検索する `Vagrantfile` のファイル名を指定します
- デフォルトは、`Vagrantfile`
- 1つのフォルダの中に設定の異なる複数の `Vagrantfile` を置くような場合
- たとえば、一時的に `Vagrant.new` をその内容が完成するまで使いたい場合

```sh
$ VAGRANT_VAGRANTFILE=Vagrantfile.new vagrant up
```
