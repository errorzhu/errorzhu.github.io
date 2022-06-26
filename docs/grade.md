# gradle常用

```
vi $GRADLE_HOME/init.d/init.gradle
```

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

settingsEvaluated { settings ->
    settings.pluginManagement {
        repositories {
            maven { url "https://maven.aliyun.com/repository/gradle-plugin" }
            mavenLocal()
            mavenCentral()
            gradlePluginPortal()
        }
    }
}
```