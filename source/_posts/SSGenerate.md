---

title: SSGenerate
urlPath: SSGenerate
date: 2019-06-06
updated: 2019-06-06
tag: Shadowsocks

---

### Mac finalShell
* execute`ssh-keygen -t rsa -f rsaSavePath -C whoiam`
* add .pub(full txt) to GCP sshkey
* finalShell add rsa path

### Mac terminal
* `ssh -i rsaSavePath whoiam@ip`

<!-- more -->

### SSGenerate

* `wget--no-check-certificate https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocks-libev-debian.sh`
* `chmod +x shadowsocks-libev-debian.sh`
* `./shadowsocks-libev-debian.sh 2>&1 | tee shadowsocks-libev-debian.log`

### SSStatus

* `/etc/init.d/shadowsocks start` 
* `/etc/init.d/shadowsocks stop` 
* `/etc/init.d/shadowsocks restart` 
* `/etc/init.d/shadowsocks status`



 
