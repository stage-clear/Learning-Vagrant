# Vagrant 学習ログ

- [実践 Vagrant を読んでのまとめ](books/vagrant-up-running)
- [(Gist) 以前のメモ](https://gist.github.com/kesuiket/e2797b2e87ddb776ab07)



## コマンドについて

* 大文字はパラメータ
* `[]` はオプション

### Vagrant コマンド

- 各プロジェクト内に影響する

|Command|Summary|format|
|:--|:--|:--|
|`vagrant init`|初期化|`vagrant init [BOX_NAME]`|
|`vagrant up`|起動||
|`vagrant reload`|再起動||
|`vagrant ssh`|SSH接続|`vagrant ssh [TAEGET]`|
|`vagrant suspend`|現状を保存して停止||
|`vagrant resume`|サスペンドを引き継いで起動||
|`vagrant halt`|停止||
|`vagrant destroy`|破棄||
|`vagrant status`|現在の状態を表示||


### ボックスコマンド

- Vagrant 全体（グローバル）に影響。

|Command|Summary|format|
|:--|:--|:--|
|`vagrant box add`|ボックスの追加|`vagrant box add [BOX_NEW_NAME] BOX_URL`|
|`vagrant box list`|ボックス一覧||
|`vagrant box remove`|ボックスの削除|`vagrant box remove BOX_NAME`|
|`vagrant box package`|ボックスのパッケージ化||
|`vagrant box repackage`|`box add` 済みのボックスの再パッケージ化||


## リンク

- [VAGRANT](https://www.vagrantup.com/)
- [VAGRANT DOCUMENTATION](https://docs.vagrantup.com/v2/)


