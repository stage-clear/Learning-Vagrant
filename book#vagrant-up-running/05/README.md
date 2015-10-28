# 複数マシン構成のクラスタのモデリング

- 現代的なWebアプリケーションは複数の要素で構成されている
- それらの構成要素はサービスと呼ばれることもある
  - 例) Web、データベース、キャッシュサービス、ワーカーキュー
- 複雑なWebサイトは、数百のサービスから生み出されることもある
- このような設計を、サービス指向アーキテクチャと呼ばれることがる

__Vagrant では__

- Vagrant ではマルチマシン環境と呼ばれる機能がある
- マルチマシン環境では、複数雨の仮想マシンが1つのVagrantfile から構築される


## 5.1 複数の仮想マシンの実行

```ruby
Vagrant::Config.run do |config|
  config.vm.box = 'precise64'

  config.vm.define 'web' do |web|
    web.vm.forward_port 80, 8080
    web.vm.provision :shell, path: 'provision.sh'
  end

  config.vm.define 'db' do |db|
    # ここはこのあと作成します
  end
end
```

- `config.vm.define` : 1つの Vagrantfile 内で新しいマシンを定義するディレクティブ
  このディレクティブは、マシンの名前を指定するパラメータだけを取ります

```sh
$ vagrant up
```

ここで Vagrant が立ち上げるマシンは2台です。出力に `web:` と `db:` があることがわか
るでしょう


## 5.2 複数のマシンの制御

- Vagrant の環境に複数マシンを導入すると, `vagrant` コマンドの振る舞いが変わります
- `up` `destory` `reload` といった多くのコマンドは、対象マシンの名前を引数として取ります
- 引数を指定しない場合はすべてのマシンが対象となる

```sh
$ vagrant reload web
```

- コマンドにより、すべてのマシンを対象にすることが意味を成さないものがある
- `vagrant ssh` は、1つのターミナルからすべてのマシンにSSH接続するわけにはいきません
- コマンドによっては、対象マシンの指定が必須となるものもあります

__確認する__

```sh
$ vagrant status
web                       running (virtualbox)
db                        running (virtualbox)
```

`web:` のみを確認する

```sh
$ vagrant status web
web                       running (virtualbox)
```

正規表現を使うこともでできる

```sh
$ vagrant status /node\d/
```


## 5.3 マシン間の通信

- サービス指向アーキテクチャをモデル化するためにマシン同士での通信する方法が必要
- デフォルトでは、単純に定義しただけでは、マシン間で通信することはできません


### 5.3.1 ホストのみのネットワーク

- 同一サブネット上のマシン群に対して、`hostonly` を定義するとマシン同士は通信できます
- 同一サブネット上とは、IPアドレスの最初の3つの部分（オクテット）が同じであること
- Vagrant ではデフォルトで `255.255.255.0` というサブネットマスクを使用します

```ruby
Vagrant::Config.run do |config|
  config.vm.box = 'precise64';

  config.vm.define 'web' do |web|
    web.vm.forward_port 80, 8080
    web.vm.provision :shell, path: 'provision.sh'
    web.vm.network :hostonly, '192.168.33.10' # <-
  end

  config.vm.define 'db' do |db|
    db.vm.network :hostonly, '192.168.33.11' # <-
  end
end
```
