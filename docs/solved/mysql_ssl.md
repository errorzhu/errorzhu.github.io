# java 使用jdbc 连接mysql 报ssl的问题
1. 现象
```
Caused by: java.io.EOFException: SSL peer shut down incorrectly
	at sun.security.ssl.SSLSocketInputRecord.read(SSLSocketInputRecord.java:480)
	at sun.security.ssl.SSLSocketInputRecord.readFully(SSLSocketInputRecord.java:458)
	at sun.security.ssl.SSLSocketInputRecord.decodeInputRecord(SSLSocketInputRecord.java:242)
	at sun.security.ssl.SSLSocketInputRecord.decode(SSLSocketInputRecord.java:180)
	at sun.security.ssl.SSLTransport.decode(SSLTransport.java:110)
	at sun.security.ssl.SSLSocketImpl.decode(SSLSocketImpl.java:1290)

```
2. 分析
涉及三方面的因素
a. java
某些高版本的java仅支持tls1.2版本
```
import javax.net.ssl.SSLContext;
import java.security.NoSuchAlgorithmException;

public class TlsTest {
    public static void main(String[] args) throws NoSuchAlgorithmException {
        String[] supportedProtocols = SSLContext.getDefault().createSSLEngine().getEnabledProtocols();
        System.out.println("Java supports the following TLS protocols:");
        for (String protocol : supportedProtocols) {
            System.out.println(protocol);
        }
    }
}

```
b. mysql jdbc driver
TLSv1 and TLSv1.1 在 Connector/J 8.0.26废弃 并且在 8.0.28已经移除

c. mysql 的版本
SHOW VARIABLES LIKE 'tls_version';
来查看

# 总结
以上问题经常出现在三者不兼容的情况下,可以在url中强制指定tls版本 enabledTLSProtocols=TLSv1.2,TLSv1.3 ,这个参数在Connector/J 8.0.28版本被重命名为enabledTLSProtocols 
可以使用-Djavax.net.debug=all增加调试信息



# 参考
https://dev.mysql.com/doc/connector-j/8.0/en/connector-j-versions.html
https://dev.mysql.com/doc/connector-j/8.0/en/connector-j-reference-using-ssl.html