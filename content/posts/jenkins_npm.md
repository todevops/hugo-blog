---
title: Jenkins使用npm构建Node.js应用
date: 2019-12-16
categories: ["tools"]
tags:
  - jenkins
  - docker
---

Jenkins使用npm构建Node.js应用
<!--more-->

## 在Jenkins中创建流水线项目
```shell
git clone https://github.com/jenkins-docs/building-a-multibranch-pipeline-project.git
```

```shell
pipeline {
    agent {
        docker {
            image 'node:6-alpine' 
            args '-p 3000:3000' 
        }
    }
    stages {
        stage('Build') { 
            steps {
                sh 'npm install' 
            }
        }
    }
}
```

```shell
git add .
git commit -m "Add initial Jenkinsfile"
```


simple-node-js-react-npm-app

An entry-level Pipeline demonstrating how to use Jenkins to build a simple Node.js and React application with npm.

### 向流水线添加Test阶段

```shell
pipeline {
    agent {
        docker {
            image 'node:6-alpine'
            args '-p 3000:3000'
        }
    }
    environment {
        CI = 'true' 
    }
    stages {
        stage('Build') {
            steps {
                sh 'npm install'
            }
        }
        stage('Test') { 
            steps {
                sh './jenkins/scripts/test.sh' 
            }
        }
    }
}
```

```shell
git stage .
git commit -m "Add 'Test' stage"
```

### 给流水线添加最终交付阶段

```shell
pipeline {
    agent {
        docker {
            image 'node:6-alpine'
            args '-p 3000:3000'
        }
    }
    environment { 
        CI = 'true'
    }
    stages {
        stage('Build') {
            steps {
                sh 'npm install'
            }
        }
        stage('Test') {
            steps {
                sh './jenkins/scripts/test.sh'
            }
        }
        stage('Deliver') { 
            steps {
                sh './jenkins/scripts/deliver.sh' 
                input message: 'Finished using the web site? (Click "Proceed" to continue)' 
                sh './jenkins/scripts/kill.sh' 
            }
        }
    }
}
```

```shell
git stage .
git commit -m "Add 'Deliver' stage"
```
