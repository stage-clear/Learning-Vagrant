2. はじめての Vagrant マシン
========================

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
  config.vm.box_url = 'http://files.vagrantup.com/precise64.com';
end
```
