# gitlab做持续集成

## 一 环境搭建

1. apt 源修改

```
vi /etc/apt/sources.list

deb http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse

```

2. 安装dnsmasq

```
systemctl disable systemd-resolved.service  
systemctl stop systemd-resolved.service 
rm -f /etc/resolv.conf
vim /etc/resolv.conf
nameserver 192.168.1.1

apt-get install dnsmasq

vim /etc/dnsmasq.conf

resolv-file=/opt/dns/resolv.conf
listen-address=127.0.0.1,192.168.1.1
addn-hosts=/opt/dns/hosts

vim /opt/dns/resolv.conf
nameserver 114.114.114.114

vim /opt/dns/hosts
192.168.1.1 gitlab.test.test.com

```

3. 安装java
4. 安装docker
```
curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
mv docker-compose-Linux-x86_64 /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

5. 安装gitlab

```
docker pull gitlab/gitlab-ce:latest
```
```
vi /opt/gitlab/compose.yml

gitlab:
  image: 'gitlab/gitlab-ce:latest'
  restart: always
  hostname: '192.168.1.1'
  environment:
    GITLAB_OMNIBUS_CONFIG: |
      external_url 'http://192.168.1.1:8999'
      gitlab_rails['gitlab_shell_ssh_port'] = 2224
      gitaly['custom_hooks_dir'] = "/var/opt/gitlab/hooks"
  ports:
    - '8999:8999'
    - '2224:22'
  volumes:
    - '/opt/gitlab/config:/etc/gitlab'
    - '/opt/gitlab/logs:/var/log/gitlab'
    - '/opt/gitlab/data:/var/opt/gitlab'

```
```
docker-compose up -d
```

6. 安装gitlab runner

```
curl -LJO "https://gitlab-runner-downloads.s3.amazonaws.com/latest/deb/gitlab-runner_amd64.deb"
dpkg -i gitlab-runner_amd64.deb

修改 gitlab-runner 启动用户为root
vi /etc/systemd/system/gitlab-runner.service

#注册
gitlab-runner register

```

7. 配置gitlab 备份

```
配置到备机免密
ssh-copy-id -i ~/.ssh/id_rsa.pub root@192.168.1.2
创建备份脚本
vim /opt/gitlab/scripts/backup.sh

#!/bin/bash
root_dir=$(cd "$(dirname $0)";pwd)
root_dir=$(readlink -f "$root_dir")
work_dir=$(dirname $root_dir)

docker exec -it gitlab_gitlab_1 bash gitlab-rake gitlab:backup:create
cd /opt/gitlab/data/backups
back_file=`ls -rt | tail -n 1`
scp $back_file 192.168.1.2:/home/gitlab_backup


加入crontab

crontab -e
0 2 * * * bash /opt/gitlab/scripts/backup.sh
```

8. 配置gitlab hook校验提交信息

```
mkdir -p /opt/gitlab/data/hooks/pre-receive.d
```
```
vi /opt/gitlab/data/hooks/pre-receive.d/comment


#!/bin/bash

##脚本提供功能：Commit提交的Message和代码规范是否符合统一规范
##分三个部分：
# 1.变量定义部分
# 2.校验部分：注释校验&代码分析
# 3.初始化入口
## 校验流程：
# 1.先做提交注释校验，校验的规则：是否已${TYPE_LIST}定义的开头，且内容长度是否大于${COMMIT_MESSAGE_MIN_LENGTH}
# 2.如果是master分之，修改了pom文件还会校验是否存在snapshot版本的jar
# 3.最后代码规范校验
## (单个项目校验)文件放置目录
# 1./var/opt/gitlab/git-data/repositories/@hashed/xx/xx/xx.git或者/var/opt/gitlab/git-data/repositories/${group}/${project_name}.git/
# 2.创建custom_hooks目录
# 3.在custom_hooks目录下创建pre-receive文件，并保持776可执行权限，且保持该文件权限：chown git:git pre-receive 以及阿里云的p3c-pmd的jar包权限
# 4.给chown -R git:git custom_hooks
# 5.官方文档说明：https://docs.gitlab.com/ee/administration/custom_hooks.html#setup


####### 初始化变量部分 #########

## 定义java_home变量 需要修改你配置的java_home
JAVA_HOME=/usr/local/jdk1.8.0_141
## 是否开启commit message的校验：0是，1否
CHECK_COMMIT_MESSAGE_ON=0
## 是否开启代码检查：0是，1否
CHECK_CODE_RULE_ON=1
## 是否校验master上的pom文件是否包含snapshot：0是，1否
CHECK_MASTER_POM_SNAPSHOT_ON=1
## 注释内容最小长度，默认20
COMMIT_MESSAGE_MIN_LENGTH=20
### 代码校验规则：0使用阿里云P3C规则，1使用checkStyle
CODE_RULE_TYPE=0

## 定义提交开头类型字符规则
## e.g: fix:测试提交bug修复，Bug编号#12
TYPE_LIST=(
         'feature:'   #新功能feature
         'update:' #在feature内修改
         'fix:'  #修补bug
         'docs:'  #文档
         'style:' #格式化，不影响代码运行的变动
         'refactor:' #重构
         'pref:'  #性能优化
         'test:'  #增加测试
         'chore:'  #构建过程或辅助工具的变动
         #'[ci skip]'  #忽略校验
)

## 获取当前路径
BASE_PATH=$(cd `dirname $0`; pwd)
echo 'BASE_PATH: '$BASE_PATH

#定义和组装校验规则
declare -a regex_list
arrLen=${#TYPE_LIST[@]}
for ((i=0;i<$arrLen;i++)) do
  regex_list[i]='^'${TYPE_LIST[i]}
done
regex_list[$arrLen+1]='^[ci skip]:'
#echo "reg_list=== "${regex_list[@]}
separator="|"
## 合并成一个完整的正则表达式
regex="$( printf "${separator}%s" "${regex_list[@]}" )"
#echo "type regex: "$regex
## 去除头部的 |
regex=${regex:${#separator}}
#echo "regex: "$regex

## 定义注释出错提示信息
tips_msg="$( printf "${separator}%s" "${TYPE_LIST[@]}" )"
tips_msg=${tips_msg:${#separator}}
####### 初始化变量部分 #########

####### 校验部分：注释校验&代码分析###########
## 校验commit message
validate_commit_message()
{
   oldrev=$(git rev-parse $1)
   newrev=$(git rev-parse $2)
   refname="$3"
   #echo 'Old version: '$oldrev
   #echo 'New version: '$newrev
   #echo 'Branch: '$refname

   ## git 命令
   #GITCMD="git"
   ## 按时间倒序列出 commit  找出两个版本之间差异的版本号集合  oldrev~newrev
   commitList=`git rev-list $oldrev..$newrev`
   #echo 'commitList: '$commitList

   split=($commitList)
   #echo 'split: '$split

   # 遍历数组
   for s in ${split[@]}
   do
      #echo “$s”
      #通过版本号获取仓库中对象实体的类型、大小和内容的信息
      #比如提交人、作者、邮件、提交时间、提交内容等
      currentContent=`git cat-file commit $s`
      #echo 'Commit obj: '$currentContent
      #获取提交内容
      msg=`git cat-file commit $s | sed '1,/^$/d'`
      #echo 'msg: '$msg

	    ## merge合并分之直接放行
	    if [[ $msg == *"Merge branch"* ]]; then
        echo "Merge branch...skip the checking"
	    else
		    ## 做内容校验
		    match=`echo $msg | grep -nE "(${regex})"`
		    #echo 'Match result: '$match

		    ## 找到匹配说明是符合规范的
		    if [ "${match}" != "" ]; then
          ## 校验注释长度
          msg_length=${#msg}
          #echo "Msg length: ${msg_length}"
          if [[ ${msg_length} -lt ${COMMIT_MESSAGE_MIN_LENGTH} ]]; then
            echo -e "Error: 提交信息的长度需要长于 ${COMMIT_MESSAGE_MIN_LENGTH} and current commit message length: ${msg_length}"
            exit 1
          fi

          ### 找到匹配内容做相应处理，如fix ,校验pom文件等
          #if [[ "${match}" =~ "fix:" ]]; then
            ## 如果是修补bug，规范有点获取到fix中的ID，然后调用禅道对外的API关闭，其他场景类似
          #fi

          # 是否开启校验和master分之
          isMaster=$(echo $refname | grep "master$")
          if [ $CHECK_MASTER_POM_SNAPSHOT_ON == 0 ] && [ -n "$isMaster" ]; then
            # 如果是master分之，并且pom文件发生了变更，判断pom文件是否含有sonapshot的引用
            pomfile=`git diff --name-only ${oldrev} ${newrev} | grep -e "pom\.xml"`
            if [[ "${pomfile}" != "" ]]; then
              #echo $pomfile
              ## 获取pom文件更新的内容
              pomcontent=`git show $newrev:$pomfile`
              #echo $pomcontent
              ## 校验pom文件是否包含snapshot版本
              if [[ $pomcontent =~ 'SNAPSHOT' ]]; then
                echo -e "Error: Snapshot 版本不能存在master分支"
                exit 1
              fi
            fi
          fi

          ## 其他操作
          echo "Commit Success!"
        else
          echo -e "Error: 提交信息需要以以下内容开头 [${tips_msg}]..."
          exit 1
        fi
		  fi
   done
}


####### 校验部分：注释校验&代码分析###########

####### 执行入口###########
pre_receive()
{
    #commit message 校验
	if [[ $CHECK_COMMIT_MESSAGE_ON == 0 ]]; then
	   validate_commit_message $1 $2 $3
	fi

}

# update hook触发会带参数执行if逻辑
# hooks脚本触发无参数执行else逻辑
if [ -n "$1" -a -n "$2" -a -n "$3" ]; then
    # Output to the terminal in command line mode - if someone wanted to
    # resend an email; they could redirect the output to sendmail
    # themselves
    pre_receive $2 $3 $1
    #echo $1'+'$2'+'$3
else
    while read oldrev newrev refname
    do
       pre_receive $oldrev $newrev $refname
       #echo $oldrev' '$newrev' '$refname
    done
fi
####### 执行入口###########
exit 0

```

9. 配置cicd流水线

![](images/wiki/ci.png)
```
stages:          # List of stages for jobs, and their order of execution
  - prepare
  - build
  - test
  - deploy

prepare-job:
  stage: prepare
  script:
    - bash /opt/cicd/scripts/prepare.sh

build-job:       # This job runs in the build stage, which runs first.
  stage: build
  script:
    - cd /opt/cicd/dds_adaptor
    - mvn clean package

unit-test-job:   # This job runs in the test stage.
  stage: test    # It only starts when the job in the build stage completes successfully.
  script:
    - echo "Running unit tests... This will take about 60 seconds."
    - sleep 5
    - echo "Code coverage is 90%"

lint-test-job:   # This job also runs in the test stage.
  stage: test    # It can run at the same time as unit-test-job (in parallel).
  script:
    - echo "Linting code... This will take about 10 seconds."
    - sleep 5
    - echo "No lint issues found."

deploy-job:      # This job runs in the deploy stage.
  stage: deploy  # It only runs when *both* jobs in the test stage complete successfully.
  script:
    - echo "Deploying application..."
    - echo "Application successfully deployed."

```

10. 安装nexus

修改maven setting
```
修改maven setting,增加发布仓库信息
<?xml version="1.0" encoding="utf-8"?>

<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">  
  <localRepository>E:\repository</localRepository>  
  <pluginGroups></pluginGroups>  
  <proxies></proxies>  
  <servers> 
    <server> 
      <id>developer</id>  
      <username>developer</username>  
      <password>123456</password> 
    </server> 
  </servers>  
  <mirrors> 
    <mirror> 
      <id>test</id>  
      <mirrorOf>central</mirrorOf>  
      <name>test仓库</name>  
      <url>http://192.168.1.1:8081/repository/test/</url> 
    </mirror> 
  </mirrors>  
  <profiles></profiles> 
</settings>

```
```
项目的pom中增加发布地址
<distributionManagement>
        <repository>
            <id>developer</id>
            <name>test-release</name>
            <url>http://192.168.1.1:8081/repository/test-release/</url>
        </repository>
        <snapshotRepository>
            <id>developer</id>
            <name>test-snapshot</name>
            <url>http://192.168.1.1:8081/repository/test-snapshot/</url>
        </snapshotRepository>
    </distributionManagement>
    <dependencyManagement>
```

11. 安装 sonarqube

```
mkdir -p  /opt/sonarqube
cd /opt/sonarqube
vi compose.yml
version: '2'
services:
  # database service: postgres
  postgres:
    image: postgres:9.6
    ports:
    - 5432:5432
    networks:
      - sonarnet
    volumes:
      - /opt/sonarqube/postgres/postgresql/:/var/lib/postgresql
      - /opt/sonarqube/postgres/data/:/var/lib/postgresql/data
    environment:
        POSTGRES_USER: sonar
        POSTGRES_PASSWORD: sonar
        POSTGRESQL_DATABASE: sonar
    restart: "no"
  # Security service: sonarqube
  sonarqube:
    image: sonarqube:8.9.2-community
    ports:
      - "9000:9000"
    networks:
      - sonarnet
    volumes:
      - sonar_data:/opt/sonarqube/data
      - sonar_log:/opt/sonarqube/log
      - sonar_extensions:/opt/sonarqube/extensions
      - sonar_conf:/opt/sonarqube/conf
    environment:
      - sonar.jdbc.username=sonar
      - sonar.jdbc.password=sonar
      - sonar.jdbc.url=jdbc:postgresql://postgres:5432/sonar
      - SONAR_ES_BOOTSTRAP_CHECKS_DISABLE=true  
    restart: "no"
networks:
  sonarnet:
    driver: bridge
volumes:
  sonar_data:
  sonar_conf:
  sonar_log:
  sonar_extensions:
```
```
docker-compose up -d 
```

12. 安装sonar-scanner

```
wget https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-4.6.2.2472-linux.zip
-o /opt/sonarqube

unzip sonar-scanner-cli-4.6.2.2472-linux.zip

ln -s  /opt/sonarqube/sonar-scanner-4.6.2.2472-linux/bin/sonar-scanner /usr/bin/sonar-scanner
```

13. 搭建私有docker registry

```
1 vi /etc/docker/daemon.json 

"insecure-registries": [
    "harbor.test.test.com:8888"
  ]


2 docker login harbor.test.test.com:8888

3 docker tag f8f4ffc8092c harbor.test.test.com:8888/test/nginx:1.21.3

4 docker push harbor.test.test.com:8888/test/nginx:1.21.3

5 docker pull harbor.test.test.com:8888/test/nginx:1.21.3
```

14. 使用docker作为gitlab runner
```
vi /etc/gitlab-runner/config.toml
volumes = ["/cache","/var/run/docker.sock:/var/run/docker.sock"]
pull_policy = "if-not-present"
```
```
stages:
  - prepare
  - test
  - check
  - build
  - deploy

prepare:
  image: harbor.test.test.com:8888/test/docker:20.10.9-git
  stage: prepare
  script:
    - git clone http://test:12345678@192.168.1.1:8999/rd/test.git target
  cache:
    key: ${CI_PIPELINE_ID}
    paths: 
      - target/

tests:
  image: harbor.test.test.com:8888/test/python:3.8-slim-gcc
  stage: test
  only:
    changes:
      - tests/*
  script:
    - cd target
    - pip install --no-cache-dir  -r requirements.txt -i http://192.168.1.1:8081/repository/test-pypi/simple --trusted-host 192.168.1.1 
    - pytest
  cache:
    key: ${CI_PIPELINE_ID}
    paths: 
      - target/


check:
  image: harbor.test.test.com:8888/test/docker:20.10.9-git
  stage: check
  only:
    changes:
      - tests/*
  script:
    - cd target
    - docker run --rm  harbor.test.test.com:8888/test/sonar-scanner-cli:4.6  sonar-scanner -Dsonar.projectKey=test -Dsonar.sources=.  -Dsonar.host.url=http://192.168.1.1:9000 -Dsonar.login=9152432e16396f6d8da1661d0699ac3c0325d21d 
  cache:
    key: ${CI_PIPELINE_ID}
    paths: 
      - target/  

    
build:
  image: harbor.test.test.com:8888/test/docker:20.10.9-git
  stage: build
  rules:
    - if: $CI_PIPELINE_SOURCE == "schedule"
  script:
    - cd target/build
    - docker build --network=host . -t harbor.test.test.com:8888/test/test:0.0.1
    - docker login -u admin -p Harbor12345 harbor.test.test.com:8888
    - docker push harbor.test.test.com:8888/test/test:0.0.1
  cache:
    key: ${CI_PIPELINE_ID}
    paths: 
      - target/


deploy:
  image: harbor.test.test.com:8888/test/docker:20.10.9-git
  rules:
    - if: $CI_PIPELINE_SOURCE == "schedule"
  stage: deploy
  script:
    - docker stop test && docker rm test
    - docker run --rm  -d  -p 5000:5000 --name test harbor.test.test.com:8888/test/test:0.0.1
    
```

15. 未完待续。。。。
