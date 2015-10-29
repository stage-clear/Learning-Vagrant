# Vagrant の設定リファレンス

(V1のみのリファレンス)

|Name|Summary|Default|
|:--|:--|:--|
|`config.nfs.map_gid`||`:auto`|
|`config.nfs.map_uid`||`:auto`|
|`config.package.name`||`package.box`|
|`config.ssh.forward_agent`||`false`|
|`config.ssh.forward_x11`||`false`|
|`config.ssh.guest_port`||`22`|
|`config.ssh.host`||`nil`|
|`config.ssh.max_retries`||`30`|
|`config.ssh.port`||`nil`|
|`config.ssh.private_key_path`|||
|`config.ssh.shell`||`bash -l`|
|`config.ssh.timeout`||`10`|
|`config.vagrant.dotfile_name`||`.vagrant`|
|`config.vagrant.host`||自動検出|
|`config.vm.auto_port_range`||`2200..2250`|
|`config.vm.base_mac`|||
|`config.vm.boot_mode`||`:headless`|
|`config.vm.box`|使用するボックスの名前||
|`config.vm.box_url`|ボックスのURL||
|`config.vm.customize`|||
|`config.vm.define`|マルチマシン環境でのマシン定義。`:primary`||
|`config.vm.forward_port`|`:adapter` `:auto` `:id` `protocol`||
|`config.vm.guest`|ゲストマシンで動作しているOS|`:linux`|
|`config.vm.host_name`|ゲストマシンのホスト名|`nil`|
|`config.vm.network`|ゲストマシンのネットワーク設定||
|`config.vm.provision`|プロビジョナの設定||
|`config.vm.share_folder`|ゲストマシンの共有フォルダ||


- [プライベートのIPv4アドレス空間](https://en.wikipedia.org/wiki/Private_network#/Private_IPv4_address_spaces)
