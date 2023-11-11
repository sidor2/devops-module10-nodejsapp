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

        // stage('Increment Version') {
        //     steps {
        //         script {
        //             dir('app'){
        //                 sh 'npm version patch'

        //                 def packageJson = readJSON file: 'package.json'
        //                 def version = packageJson.version

        //                 // set the new version as part of IMAGE_NAME
        //                 env.IMAGE_NAME = "$DOCKER_IMAGE:$version"

        //             }
        //         }
        //     }
        // }
        stage('Fetch AWS Credentials from IMDSv2') {
            steps {
                // script {
                //     echo "Fetching AWS Credentials from IMDSv2"
                //     sh "pwd"
                //     sh 'export TOKEN=`curl -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600"`'
                //     sh 'curl -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/meta-data/iam/security-credentials/JenkinsInstanceRole > creds.json'
                // }
                script {
                    // Define metadata URL and headers for IMDSv2
                    def IMDSv2_URL = 'http://169.254.169.254/latest/meta-data/iam/security-credentials/'
                    def IMDSv2_TOKEN = sh (
                        script: "curl -X PUT \"http://169.254.169.254/latest/api/token\" -H \"X-aws-ec2-metadata-token-ttl-seconds: 21600\"",
                        returnStdout: true
                    ).trim()

                    // Retrieve the role name
                    def ROLE_NAME = sh (
                        script: "curl -H \"X-aws-ec2-metadata-token: ${IMDSv2_TOKEN}\" ${IMDSv2_URL}",
                        returnStdout: true
                    ).trim()

                    // Retrieve the credentials
                    def CREDENTIALS = sh (
                        script: "curl -H \"X-aws-ec2-metadata-token: ${IMDSv2_TOKEN}\" ${IMDSv2_URL}${ROLE_NAME}",
                        returnStdout: true
                    ).trim()
                    def TEMP_ROLE = readJSON text: "${CREDENTIALS}"

                    // Extract credentials
                    AWS_ACCESS_KEY_ID = TEMP_ROLE.AccessKeyId
                    AWS_SECRET_ACCESS_KEY = TEMP_ROLE.SecretAccessKey
                    AWS_SESSION_TOKEN = TEMP_ROLE.Token

                    // Mask passwords and set environment variables
                    wrap([$class: 'MaskPasswordsBuildWrapper', varPasswordPairs: [[password: AWS_ACCESS_KEY_ID], [password: AWS_SECRET_ACCESS_KEY], [password: AWS_SESSION_TOKEN]]]) {
                        withEnv(['AWS_SESSION_TOKEN=${AWS_SESSION_TOKEN}', 'AWS_SECRET_ACCESS_KEY=${AWS_SECRET_ACCESS_KEY}', 'AWS_ACCESS_KEY_ID=${AWS_ACCESS_KEY_ID}']) {
                            echo "TOKEN: ${AWS_SESSION_TOKEN}"
                            echo "KEY: ${AWS_SECRET_ACCESS_KEY}"
                            echo "ID: ${AWS_ACCESS_KEY_ID}"
                        }
                    }
}

                
            }
        }
        // stage('Build Docker Image') {
        //     steps {
        //         script {
        //             def image = "${env.DOCKER_REGISTRY}/${env.IMAGE_NAME}"
        //             echo "Building Docker Image: ${image}"
        //             sh "which docker"
        //             sh "docker build -t ${image} ."
        //         }
        //     }
        // }

        // stage('commit to git'){
        //     steps{
        //         script{
        //             commitToGithub("${GIT_CREDENTIALS}","devops-module10-nodejsapp","master")
        //         }
        //     }
        // }
    }
}
