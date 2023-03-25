# 使用javassist运行时修改第三方库

## 背景
调试flink程序时,debug到一个依赖的第三方库,代码如下
```
if (this.pingOutstanding == 0 && time - this.lastOutboundActivity >= 2L * this.keepAliveNanos) 

```
想分析一下其中几个变量的值,为何会进入这个分支(当时不知道idea的evalue and log功能,汗),每次手动watch这几个变量,感觉很麻烦。突发奇想一个方法,使用字节码编辑技术,人工添加自己的log进入目标方法体,这样即使不是debug模式,直接就可在控制台看到连续的变量变化。


## 实践

```
    private void decorateMqtt() throws NotFoundException, CannotCompileException, IOException {
        ClassPool classPool = ClassPool.getDefault();
        String clsName = "org.eclipse.paho.client.mqttv3.internal.ClientState";
        CtClass ctClass = classPool.get(clsName);
        CtMethod ctMethod = ctClass.getDeclaredMethod("checkForActivity");
        ctMethod.insertBefore("System.out.println(\"############\");");
        ctClass.toClass();
    }
```
使用javassist装饰checkForActivity方法,使得checkForActivity被调用时会打印我们的log


## 问题
```
class "org.eclipse.paho.client.mqttv3.internal.HighResolutionTimer"'s signer information does not match signer information of other classes in the same package
```
是classloader在加载类时,发现签名和同一个包下的其他类签名不同
检查依赖的jar包,在META-INF中发现使用了rsa加密签名

## 解决
使用了偷懒的方法,直接忽略了jar包的签名校验
```
Security.setProperty("jdk.jar.disabledAlgorithms", "MD2, MD5, RSA");
```

## 附录
完整代码如下
```
import javassist.*;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;

import java.io.IOException;
import java.security.Security;


public class ParallelTest {


    private void decorateMqtt() throws NotFoundException, CannotCompileException, IOException {
        ClassPool classPool = ClassPool.getDefault();
        String clsName = "org.eclipse.paho.client.mqttv3.internal.ClientState";
        CtClass ctClass = classPool.get(clsName);
        CtMethod ctMethod = ctClass.getDeclaredMethod("checkForActivity");
        ctMethod.insertBefore("System.out.println(\"############\");");
        ctClass.toClass();
    }



    public static void main(String[] args) throws Exception {

        Security.setProperty("jdk.jar.disabledAlgorithms", "MD2, MD5, RSA");
        
        ParallelTest test = new ParallelTest();
        test.decorateMqtt();
        
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);
        env.addSource(new PSource()).addSink(new PSink());
        env.execute();
        
    }



}
```
