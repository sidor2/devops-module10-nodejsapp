#!/usr/bin/env groovy

library identifier: 'jenkins-shared-library@main', retriever: modernSCM(
        [$class: 'GitSCMSource',
         remote: 'git@github.com:sidor2/devops-module8-jenkins-shared-lib.git',
         credentialsId: '2c40c606-3564-4fc4-8fc2-3a89a016f089',
        ]
)

pipeline {
    agent any

    tools {
            nodejs 'nodejs-20'
        }

    environment {
        DOCKER_IMAGE = 'devops-module10'
        DOCKER_REGISTRY = '253021321210.dkr.ecr.us-west-2.amazonaws.com'
        GIT_CREDENTIALS = '2c40c606-3564-4fc4-8fc2-3a89a016f089'
    }

    stages {
        
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        // stage('Run Tests') {
        //     steps {
        //         dir('app'){
        //             sh 'npm install'
        //             sh 'npm test'
        //         }
        //     }
        // }

        stage('Increment Version') {
            steps {
                script {
                    dir('app'){
                        sh 'npm version patch'

                        def packageJson = readJSON file: 'package.json'
                        def version = packageJson.version

                        // set the new version as part of IMAGE_NAME
                        env.IMAGE_NAME = "$DOCKER_IMAGE:$version"

                    }
                }
            }
        }
        stage('Fetch AWS Credentials from IMDSv2') {
            steps {
                script {
                    sh "TOKEN=`curl -X PUT 'http://169.254.169.254/latest/api/token' -H 'X-aws-ec2-metadata-token-ttl-seconds: 21600'`"
                    sh "curl -H 'X-aws-ec2-metadata-token: $TOKEN' http://169.254.169.254/latest/meta-data/iam/security-credentials/JenkinsInstanceProfile > creds.json"
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    def image = "${env.DOCKER_REGISTRY}/${env.IMAGE_NAME}"
                    echo "Building Docker Image: ${image}"
                    sh "which docker"
                    sh "docker build -t ${image} ."
                }
            }
        }

        stage('commit to git'){
            steps{
                script{
                    commitToGithub("${GIT_CREDENTIALS}","devops-module10-nodejsapp","master")
                }
            }
        }
    }
}
