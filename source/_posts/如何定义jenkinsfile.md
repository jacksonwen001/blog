---
title: 如何定义jenkinsfile
date: 2022-03-28 17:41:51
tags: jenkins 
---
# pipeline
一个常见的 pipeline 如下所示 
```groovy
pipeline {
    agent any 
    stages {
        stage('Build') { 
            agent {
                docker {
                    image 'gradle:7.4'
                    reuseNode true
                }
            }
            steps {
                // 
            }
        }
        stage('Test') { 
            steps {
                // 
            }
        }
        stage('Deploy') { 
            steps {
                // 
            }
        }
    }
}
```
这里需要注意的是 `agent` ， 如果全局的 `agent` 声明的是一个 `Docker` 镜像的话， 那么这一整套流程就全在这个镜像里面运行， 如果是在 `stage` 中声明的 `agent` 那么就说明这个 `agent` 只在这个 `stage` 中运行，其他 `stage` 就运行在全局中

# Docker 
使用 `jenkinsfile` 绕不过去的是 `Docker` 可参考下面两个例子

# 参考
* https://prog.world/how-to-set-up-pipeline-for-jenkins-selenoid-allure/
* https://docs.cloudbees.com/docs/admin-resources/latest/plugins/docker-workflow 
