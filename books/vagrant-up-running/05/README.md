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

__ping で確認__

```
# ping to web
$ ping 192.168.33.10
64 bytes from 192.168.33.10: icmp_seq=0 ttl=64 time=0.497 ms

# ping to db
$ ping 192.168.33.11
64 bytes from 192.168.33.11: icmp_seq=0 ttl=64 time=0.477 ms

# web から db に ping
$ vagrant ssh web
vagrant@precise64:~$ ping 192.168.33.11
64 bytes from 192.168.33.11: icmp_req=1 ttl=64 time=0.573 ms
```

### 5.3.2 ブリッジされたネットワーク

- `bridged` なネットワークでも複数のマシン間の通信が可能です
- 厳密には、それらのマシンはすべて同じデバイスで `bridged` されなければいけません
- DHCP で割り振られたIPを手動で確認した上で、それらを使って通信する

これは理想的な方法とは言えず、複数マシンの環境下でマシン群を通信するための方法としは、
`hostonly` のネットワークを使う方がいいでしょう


## 5.4 実際的な例: MySQL

```sh
export DEBIAN_FRONTEND=noninteractive
apt-get update
apt-get install -y mysql-server
sed -i -e 's/127.0.0.0/0.0.0.0/' /etc/mysql/my.cnf
restart mysql
mysql -uroot mysql <<< "GRANT ALL ON *.* TO 'root'@'; FLUSH PRIVILEGES;"
```

- 環境変数 `DEBIAN_FRONTEND` を `noninteractive` という値にしてエクスポートする
  こうすることで、MySQLサーバーをインストールする際に、ルートのパスワードを尋ねられる
  ことなくインストールが終了する
- `sed` : バインドするアドレスをループバックからすべてのインターフェースを意味する
  `0.0.0.0` に変更する。これは、リモートマシンがこのサーバーに接続できるようにするた
  めに必要です
- `restart mysql` : MySQL を再起動します
- MySQL に対してルートで任意のホストかrの接続を許可（これはとても安全とは言えない）

```ruby
Vagrant::Config.run do |config|
  config.vm.box = 'precise64'

  config.vm.define 'web' do |web|
    web.vm.forward_port 80, 8080
    web.vm.provision :shell, path: 'provision.sh'
    web.vm.provision :shell, inline: 'apt-get install -y mysql-client'
    web.vm.network :hostonly, '192.168.33.10'
  end

  config.vm.define 'db' do |db|
    db.vm.provision :shell, path: 'db_provision.sh'
    db.vm.network :hostonly, '192.168.33.11'
  end
end
```

```sh
$ vagrant destroy
$ vagrant up
```

マシン群が動き始めたら、web マシンにSSHログインして、MySQLのクライアントを使って、db
マシンにアクセスしてください

```sh
$ vagrant ssh web
vagrant@precise64:~$ mysql -uroot -h192.168.33.11
```
