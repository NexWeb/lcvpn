# LCVPN - Light decentralized VPN in Golang

LCVPN is used in production in many different environments.
  - Very light and easy (one similar config on all hosts)
  - Use same config for all hosts (autodetect local params) - useful with puppet etc
  - Uses AES-128, AES-192 or AES-256 encryption (note that AES-256 is **much slower** than AES-128 on most computers) + optional HMAC-SHA256 or (super secure! 😅) NONE encryption (just copy without modification)
  - Communicates via UDP directly to selected host (no central server)
  - Works only on Linux (uses TUN device)
  - Support of basic routing - can be used to connect several networks
  - Multithread send and receive - scaleable for big traffic
  - Uses so_reuseport for better results in case of a larger number of hosts
  - It's still in beta, use it at your own risk (and please use only versions marked as "release")

![alt tag](https://raw.githubusercontent.com/nexweb/lcvpn/master/topology.png)

### Install and run

You need golang (at least 1.5) installed and configured:

```sh
$ go get -u github.com/nexweb/lcvpn
```

if you have config in /etc/lcvpn.conf

```sh
$ sudo $GOPATH/bin/lcvpn
```

if you want to specify different location of config (or if you need to run several instances)

```sh
$ sudo $GOPATH/bin/lcvpn -config lcvpn.conf
```
if you host is hidden behind firewall (with udp port forward) lcvpn is unable to detect
which "remote" is localhost. In this case use next syntax:

```sh
$ sudo $GOPATH/bin/lcvpn -local berlin -config lcvpn.conf
```


### Config example

```
[main]
port = 23456
encryption = aescbc
mainkey = 4A34E352D7C32FC42F1CEB0CAA54D40E9D1EEDAF14EBCBCECA429E1B2EF72D21
laltkey = 1111111117C32FC42F1CEB0CAA54D40E9D1EEDAF14EBCBCECA429E1B2EF72D21
broadcast = 192.168.3.255
netcidr = 24
recvThreads = 4
sendThreads = 4

[remote "prague"]
ExtIP = 46.234.105.229
LocIP = 192.168.3.15
route = 192.168.10.0/24
route = 192.168.15.0/24
route = 192.168.20.0/24

[remote "berlin"]
ExtIP = 103.224.182.245
LocIP = 192.168.3.8
route = 192.168.11.0/24

[remote "kiev"]
ExtIP = 95.168.211.37
LocIP = 192.168.3.3
```

where port is UDP port for communication  
encryption is *aescbc* for AES-CBC, *aescbchmac* for AES-CBC+HMAC-SHA245 or *none* for no encryption  
for *aescbc* mainkey/altkey is hex form of 16, 24 or 32 bytes key (for AES-128, AES-192 or AES-256)  
for *aescbchmac* mainkey/altkey is 32 bytes longer
for *none* mainkey/altkey mainkey/altkey is just ignored
number of remotes is virtualy unlimited, each takes about 256 bytes in memory  

### Config reload

Config is reloaded on HUP signal. In case of invalid config just log message will appeared, previous one is used.  
P.S.: listening udp socket is not reopened for now, so on port change restart is needed

### Online key change

**altkey** configuration option allows specify alternative encryption key that will be used in case if decription with primary
one failed. This allow to use next algoritm to change keys without link going offline:
  - In normal state only **mainkey** is set (setting altkey is more cpu-consuming)
  - Set altkey to new key on all hosts and send HUP signal
  - Exchange altkey and aeskey on all hosts and send HUP signal
  - Remove altkey (with old key) from configs on all hosts and send HUP signal again
  - We are running with new key :)

### Roadmap

* 100% unit test coverage
* please let me know if you need anything more
