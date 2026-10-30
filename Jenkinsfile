#!/usr/bin/env groovy
library identifier: 'jenkins-shared-library@master', retriever: modernSCM(
        [$class: 'GitSCMSource',
        remote: 'https://gitlab.com/twn-devops-bootcamp/latest/08-jenkins/jenkins-shared-library.git',
        credentialsId: 'gitlab-credentials'])

def gv

pipeline {   
    agent any
    tools {
        maven 'maven-3.10'
    }
    environment {
        IMAGE_NAME = 'olushad/demo-app:java-maven-1.0'
    }
    stages {
        stage("init") {
            steps {
                script {
                    gv = load "script.groovy"
                }
            }
        }

        stage("build jar") {
            steps {
                script {
                    buildJar()
                }
            }
        }

        stage("build and push image") {
            steps {
                script {
                    buildImage(env.IMAGE_NAME)
                    dockerLogin()
                    dockerPush(env.IMAGE_NAME)
                }
            }
        }
        
        stage("deploy") {
            steps {
                script {
                    echo 'deploying docker image to EC2...'
                    def dockerCmd = "docker run -d -p 8080:8080 ${IMAGE_NAME}"
                    sshagent(credentials: ['ec2-server-key'], executable: '') {
                        sh "ssh -o StrictHostKeyChecking=no ec2-user@3.148.201.188 ${dockerCmd}"
                    }
                }
            }
        }               
    }
}
