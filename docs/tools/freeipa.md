# freeipa 实战
```
vi /etc/sysctl.conf
net.ipv6.conf.lo.disable_ipv6 = 0
sysctl -p
yum update nss* -y
ipa-server-install
```