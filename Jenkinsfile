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
                    // Fetch the session token
                    env.TOKEN = sh(script: "curl -X PUT 'http://169.254.169.254/latest/api/token' -H 'X-aws-ec2-metadata-token-ttl-seconds: 21600'", returnStdout: true).trim()

                    // Fetch the IAM role name
                    env.ROLE_NAME = sh(script: "curl -H 'X-aws-ec2-metadata-token: ${env.TOKEN}' http://169.254.169.254/latest/meta-data/iam/security-credentials/", returnStdout: true).trim()

                    // Fetch the IAM role credentials and set environment variables
                    withCredentials([string(credentialsId: 'AWS_ACCESS_KEY_ID', variable: 'AWS_ACCESS_KEY_ID'),
                                     string(credentialsId: 'AWS_SECRET_ACCESS_KEY', variable: 'AWS_SECRET_ACCESS_KEY'),
                                     string(credentialsId: 'AWS_SESSION_TOKEN', variable: 'AWS_SESSION_TOKEN')]) {
                        sh '''
                            curl -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/meta-data/iam/security-credentials/$ROLE_NAME > temp_credentials.json
                            export AWS_ACCESS_KEY_ID=$(jq -r '.AccessKeyId' temp_credentials.json)
                            export AWS_SECRET_ACCESS_KEY=$(jq -r '.SecretAccessKey' temp_credentials.json)
                            export AWS_SESSION_TOKEN=$(jq -r '.Token' temp_credentials.json)
                            rm -f temp_credentials.json
                        '''
                    }
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
