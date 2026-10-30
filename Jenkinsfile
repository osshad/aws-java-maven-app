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
                    def shellCmd = "bash ./server-cmds.sh"
                    // def dockerComposeCmd = "docker-compose -f docker-compose.yaml up -d"
                    sshagent(credentials: ['ec2-server-key'], executable: '') {
                        sh "scp server-cmds.sh ec2-user@3.148.201.188:/home/ec2-user"
                        sh "scp docker-compose.yaml ec2-user@3.148.201.188:/home/ec2-user"
                        sh "ssh -o StrictHostKeyChecking=no ec2-user@3.148.201.188 ${shellCmd}"
                    }
                }
            }
        }               
    }
}
