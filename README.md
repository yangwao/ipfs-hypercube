# ipfs-hypercube
### WorkInProgress

## Aim of Project
*Easy to deploy IPFS (local/LAN) nodes on ARM single-board computers.*

Boards that I plan document and deploy
* Hardkernel ODROID-X2 (tested/running)
* TP-LINK TL-MR-3020 (have it, onboard wifi)
* Hardkernel ODROID-C2 ([Shipping date is March 4, 2016](http://forum.odroid.com/viewtopic.php?f=135&t=18683))


### ODROID-X2
* buy board from [hardkernel](http://www.hardkernel.com/main/products/prdt_info.php?g_code=G135235611947)
* storage [SanDisk MicroSDHC 32GB Extreme UHS-I (U3)](https://www.alza.sk/sandisk-microsdhc-32gb-extreme-uhs-i-u3-sd-adapter-gopro-edition-d2923771.htm)


#### Easy go (quick & dirty, ~2h of hacking)

##### Prepare board
[Download ubuntu 14.04lts-server from odroid.in](http://odroid.in/ubuntu_14.04lts/ubuntu-14.04lts-server-odroid-x2-20140604.img.xz)
Then extract .xz

`xz -d ubuntu-14.04lts-server-odroid-x2-20140604.img.xz`

Push image to SD card

`sudo dd if=ubuntu-14.04lts-server-odroid-x2-20140604.img of=/dev/disk2 bs=1m`

Insert SD card, boot your X2, login credentials are `root:odroid`

Download [odroid-utility](https://github.com/mdrjr/odroid-utility) and resize rootfs && update your kernel to latest

```
sudo -s

wget -O /usr/local/bin/odroid-utility.sh https://raw.githubusercontent.com/mdrjr/odroid-utility/master/odroid-utility.sh

chmod +x /usr/local/bin/odroid-utility.sh

odroid-utility.sh

reboot

```

##### IPFS part
* [**go for ipfs, yay!**](https://ipfs.io/docs/install/)
(I had go through secret level http://dist.ipfs.io/ and open very secret doors => http://dist.ipfs.io/go-ipfs/v0.4.0-dev/go-ipfs_v0.4.0-dev_linux-arm.tar.gz)
* `mv ipfs /usr/local/bin`
* `mkdir -p /mnt/ipfs/datastore`

Turn off firewall (even iptables are missing in image)
* `apt-get install ufw`
* `ufw disable`

Initialize IPFS, it will generates your privkey and other stuff
* `ipfs init`

You can run ipfs daemon from terminal (or put it in screen/tmux/byobu) or use my upstart script

* `ipfs daemon`

Upstart script for ubuntu 14.04 /etc/init/ipfs.conf


```
#!upstart
description "ipfs"

#env USER=nobody
env USER=root # need change this

start on runlevel [2345]
stop on runlevel [016]

respawn
respawn limit 2 5

exec start-stop-daemon --start --chuid $USER --exec /usr/local/bin/ipfs -- daemon >> /var/log/ipfs.log 2>&1
```

Now should IPFS daemon start on every boot, cheers

and some pointless information, yaay :D

![ODROID-X2 running IPFS](https://ipfs.pics/ipfs/QmRcA7bnBpj4E65aG9JRxGw1g2zBjAfYpA6TocT1qtsoAa)
```
root@odroid-server:~# ipfs id
{
	"ID": "QmbzAwAqLzpEkCT98n8vYZkDGbbZZEkZzumYLyyRVJJfiz",
	"PublicKey": "CAASpgIwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQC77y7FK2NHq9Ty+bOCFVhcKq6rmwQWc3pQLdeMfERzsuoEYLbZOt03nEmD0/YuvzGVdV1XVtDapdnIyXenVTrDKc8Dnig4kN6aQV4bFODx27vtB7Qw+zvHGZXDW87DAhkk3aS208D+UpvCkXBoG0sDSW5S5vMNpBXoscsEhiAGbBopxxw3Ua+/mTQjYrLq0eGUAvzvKQ1HVe2sq2arUNjvS01cIL8npzgYcBYjcIQoQgGsR1Pc4IOFehOM64bDooG2k0jTZFS63HhJxLuEXQ7soolNR+4yJcDAXHj1Wp/Lpc47EE8BsolUwcON2Od9RKQlEUBhtLHp1ibMVukzaVKTAgMBAAE=",
	"Addresses": [
		"/ip4/127.0.0.1/tcp/4001/ipfs/QmbzAwAqLzpEkCT98n8vYZkDGbbZZEkZzumYLyyRVJJfiz",
		"/ip4/192.168.13.215/tcp/4001/ipfs/QmbzAwAqLzpEkCT98n8vYZkDGbbZZEkZzumYLyyRVJJfiz",
		"/ip6/::1/tcp/4001/ipfs/QmbzAwAqLzpEkCT98n8vYZkDGbbZZEkZzumYLyyRVJJfiz"
	],
	"AgentVersion": "go-ipfs/0.4.0-dev",
	"ProtocolVersion": "ipfs/0.1.0"

root@odroid-server:~# ipfs diag sys
  {"diskinfo":{"free_space":2.500820992e+09,"fstype":"61267","total_space":2.33693184e+09},"environment":{"GOPATH":"","IPFS_PATH":""},"ipfs_commit":"","ipfs_version":"0.4.0-dev","memory":{"swap":0,"virt":8.02464e+08},"net":{"interface_addresses":["/ip4/127.0.0.1","/ip4/192.168.13.215","/ip6/::1","/ip6/fe80::34e6:6aff:fe0e:97b1"]},"runtime":{"arch":"arm","compiler":"gc","gomaxprocs":3,"numcpu":4,"numgoroutines":202,"os":"linux","version":"go1.5.3"}}

root@odroid-server:~# ipfs swarm peers|wc -l
26
```
Credits goes to guys on #ipfs @ freenode, need fix traversal/discovery of external IP

Some quick fixies 

```rm -rf $GOPATH/src/gx and try it again```
```go get -u github.com/whyrusleeping/gx and go get -u github.com/whyrusleeping/gx-go try it again```

feel free to connect to my ipfs node https://ipfs.io/ipfs/QmVsrcQiXGD1FNYeKxxVBLor1dmszvr1xnv6Jgq61jhhci/paste
