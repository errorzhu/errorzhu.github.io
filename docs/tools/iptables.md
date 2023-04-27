# iptables 使用

```
# 禁止访问8082
iptables -A INPUT -p tcp --dport 8082 -j DROP
# 允许 127.0.0.1 访问8082
iptables -I INPUT -p tcp -s 127.0.0.1 --dport 8082 -j ACCEPT
# 显示所有
iptables -nL --line-numbers
# 删除
iptables -D {}
```
