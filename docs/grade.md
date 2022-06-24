# gradle常用

```
allprojects {
  repositories {
    maven {
      url 'https://maven.aliyun.com/repository/public/'
    }
    maven {
      url 'https://maven.aliyun.com/repository/jcenter'
    }
    mavenLocal()
    mavenCentral()
  }
}
```