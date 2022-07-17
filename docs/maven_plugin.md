# 常用maven插件

# maven-antrun-plugin
运行自定义ant任务,适合一些灵活的场景,比如拷贝release包到一个自定义目录
```
 <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-antrun-plugin</artifactId>
        <version>3.0.0</version>
        <executions>
          <execution>
            <id>compile</id>
            <phase>compile</phase>
            <configuration>
              <target>
                <property name="compile_classpath" refid="maven.compile.classpath"/>
                <property name="runtime_classpath" refid="maven.runtime.classpath"/>
                <property name="test_classpath" refid="maven.test.classpath"/>
                <property name="plugin_classpath" refid="maven.plugin.classpath"/>

                <echo message="compile classpath: ${compile_classpath}"/>
                <echo message="runtime classpath: ${runtime_classpath}"/>
                <echo message="test classpath:    ${test_classpath}"/>
                <echo message="plugin classpath:  ${plugin_classpath}"/>
              </target>
            </configuration>
            <goals>
              <goal>run</goal>
            </goals>
          </execution>
        </executions>
      </plugin>
```

maven-archetype-plugin
生成项目骨架
```
mvn archetype:generate -DgroupId=com.mycompany.app -DartifactId=myapp -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
```

maven-assembly-plugin
制作项目分发包,制作结构化的部署包
```
      <plugin>
        <artifactId>maven-assembly-plugin</artifactId>
        <version>3.3.0</version>
        <configuration>
          <filters>
            <filter>src/assembly/filter.properties</filter>
          </filters>
          <descriptors>
            <descriptor>src/assembly/distribution.xml</descriptor>
          </descriptors>
        </configuration>
      </plugin>
```
distribution.xml
```
<assembly xmlns="http://maven.apache.org/ASSEMBLY/2.1.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/ASSEMBLY/2.1.0 http://maven.apache.org/xsd/assembly-2.1.0.xsd">
  <id>distribution</id>
  <formats>
    <format>jar</format>
  </formats>
  <fileSets>
    <fileSet>
      <directory>${basedir}</directory>
      <includes>
        <include>*.txt</include>
      </includes>
      <excludes>
        <exclude>README.txt</exclude>
        <exclude>NOTICE.txt</exclude>
      </excludes>
    </fileSet>
  </fileSets>
  <files>
    <file>
      <source>README.txt</source>
      <outputDirectory></outputDirectory>
      <filtered>true</filtered>
    </file>
    <file>
      <source>NOTICE.txt</source>
      <outputDirectory></outputDirectory>
      <filtered>true</filtered>
    </file>
  </files>
</assembly>
```
maven-dependency-plugin
解析项目依赖，获取所有依赖包
```
    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-dependency-plugin</artifactId>
        <version>3.1.2</version>
        <executions>
          <execution>
            <id>copy-dependencies</id>
            <phase>package</phase>
            <goals>
              <goal>copy-dependencies</goal>
            </goals>
            <configuration>
              <outputDirectory>${project.build.directory}/alternateLocation</outputDirectory>
              <overWriteReleases>false</overWriteReleases>
              <overWriteSnapshots>false</overWriteSnapshots>
              <overWriteIfNewer>true</overWriteIfNewer>
            </configuration>
          </execution>
        </executions>
      </plugin>
```
maven-enforcer-plugin
配置规则检查
```
     <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-enforcer-plugin</artifactId>
        <version>3.0.0-M3</version>
        <executions>
          <execution>
            <id>enforce-versions</id>
            <goals>
              <goal>enforce</goal>
            </goals>
            <configuration>
              <rules>
                <bannedPlugins>
                  <!-- will only display a warning but does not fail the build. -->
                  <level>WARN</level>
                  <excludes>
                    <exclude>org.apache.maven.plugins:maven-verifier-plugin</exclude>
                  </excludes>
                  <message>Please consider using the maven-invoker-plugin (http://maven.apache.org/plugins/maven-invoker-plugin/)!</message>
                </bannedPlugins>
                <requireMavenVersion>
                  <version>2.0.6</version>
                </requireMavenVersion>
                <requireJavaVersion>
                  <version>1.5</version>
                </requireJavaVersion>
                <requireOS>
                  <family>unix</family>
                </requireOS>
              </rules>
            </configuration>
          </execution>
        </executions>
      </plugin>
```

maven-resources-plugin
处理resources目录下资源文件
```
 <resources>
      <resource>
        <directory>[your directory]</directory>
        <includes>
          <include>[resource file #1]</include>
          <include>[resource file #2]</include>
          <include>[resource file #3]</include>
          ...
          <include>[resource file #n]</include>
        </includes>
      </resource>
      ...
    </resources>
```
versions-maven-plugin
统一修改版本
```
mvn versions:set -DnewVersion=1.2.0-SNAPSHOT
```