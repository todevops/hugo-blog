---
title: Jenkins使用Maven构建Java应用程序
date: 2019-12-16
categories: ["tools"]
tags:
  - jenkins
  - docker
---

Jenkins使用Maven构建简单的Java应用程序
<!--more-->

## 安装 Jenkins

> 参考文档 [Jenkins 的安装与使用](https://todevops.github.io/posts/jenkins/)

## 安装 Docker

> 参考文档 [Docker 的安装与使用](https://todevops.github.io/posts/docker/)

## 在Jenkins中创建流水线项目
1. 创建目录，克隆项目；

```bash
[root@jenkins ~]# mkdir /home/github
[root@jenkins ~]# git clone https://github.com/jenkins-docs/simple-java-maven-app.git /home/github/simple-java-maven-app
```

2. 为新的流水线项目指定名称（例如 simple-java-maven-app）；

![](/resources/jenkins/jenkins_07.png)

3. 在 Definition 域中，选择 Pipeline script from SCM 选项。此选项指示Jenkins从源代码管理（SCM）仓库获取你的流水线， 这里的仓库就是你clone到本地的Git仓库；
4. 在 SCM 域中，选择 Git ，在 Repository URL 域中，填写你本地仓库的目录路径，这是从你主机上的用户账户home目录映射到Jenkins容器的 /home 目录：`/home/github/simple-java-maven-app`；
5. 点击 Save 保存你的流水线项目。你现在可以开始创建你的 Jenkinsfile，这些文件会被添加到你的本地仓库

![](/resources/jenkins/jenkins_08.png)

## 将初始流水线创建为Jenkinsfile
> 创建一个初始流水线来下载 Maven Docker 镜像，并将其作为 Docker 容器运行（这将构建你的简单Java应用）。同时添加一个“构建”阶段到流水线中，用于协调整个过程。
1. 在你本地的 simple-java-maven-app Git仓库的根目录创建并保存一个名为 Jenkinsfile 的文本文件。
2. 复制以下声明式流水线代码并粘贴到 Jenkinsfile 文件中：

```bash
[root@jenkins ~]# cd /home/github/simple-java-maven-app
[root@jenkins simple-java-maven-app]# vim Jenkinsfile
```

```shell
pipeline {
    agent {
        docker {
            image 'maven:3-alpine'
            args '-v /root/.m2:/root/.m2'
        }
    }
    stages {
        stage('Build') {
            steps {
                sh 'mvn -B -DskipTests clean package'
            }
        }
    }
}
```

3. 保存对 Jenkinsfile 的修改并且将其提交到你本地的 simple-java-maven-app Git仓库
```shell
[root@jenkins simple-java-maven-app]# git stage .
[root@jenkins simple-java-maven-app]# git commit -m "Add initial Jenkinsfile"
```
4. 再次回到 Jenkins，点击 Build Now。

![](/resources/jenkins/jenkins_09.png)

## 为流水线增加test阶段
1. 打开你的 Jenkinsfile
2. 复制以下声明式流水线代码，并粘贴到 Jenkinsfile 中 Build 阶段的下方：
```shell
        stage('Test') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }
```
最终的代码为：
```shell
pipeline {
    agent {
        docker {
            image 'maven:3-alpine'
            args '-v /root/.m2:/root/.m2'
        }
    }
    stages {
        stage('Build') {
            steps {
                sh 'mvn -B -DskipTests clean package'
            }
        }
        stage('Test') { 
            steps {
                sh 'mvn test' 
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }
    }
}
```
3. 保存对 Jenkinsfile 的修改并将其提交到你的本地 simple-java-maven-app Git仓库。
```shell
git stage .
git commit -m "Add 'Test' stage"
```
4. 运行构建

## 为你的流水线增加deliver阶段
1. 打开你的 Jenkinsfile。
2. 复制以下声明式流水线代码，并粘贴到 Jenkinsfile 中 Test 阶段的下方：
```shell
        stage('Deliver') {
            steps {
                sh './jenkins/scripts/deliver.sh'
            }
        }
```
最终的代码为：
```shell
pipeline {
    agent {
        docker {
            image 'maven:3-alpine'
            args '-v /root/.m2:/root/.m2'
        }
    }
    stages {
        stage('Build') {
            steps {
                sh 'mvn -B -DskipTests clean package'
            }
        }
        stage('Test') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }
        stage('Deliver') { 
            steps {
                sh './jenkins/scripts/deliver.sh' 
            }
        }
    }
}
```
3. 保存对 Jenkinsfile 的修改并将其提交到你的本地 simple-java-maven-app Git仓库。
```shell
git stage .
git commit -m "Add 'Deliver' stage"
```
4. 运行构建