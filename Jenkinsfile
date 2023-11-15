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

        stage('Run Tests') {
            steps {
                dir('app'){
                    // sh 'npm install'
                    // sh 'npm test'
                    echo 'Tests are not implemented yet'
                }
            }
        }

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
        stage('Set AWS Credentials') {
            steps {
                script {
                    def creds = getAwsEC2creds()

                    withEnv([
                        "AWS_ACCESS_KEY_ID=${creds[0]}",
                        "AWS_SECRET_ACCESS_KEY=${creds[1]}",
                        "AWS_SESSION_TOKEN=${creds[2]}"
                    ]) {
                        echo "${AWS_ACCESS_KEY_ID}"
                        sh 'aws sts get-caller-identity'
                    }
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    def image = "${env.DOCKER_REGISTRY}/${env.IMAGE_NAME}"
                    echo "Building Docker Image: ${image}"
                    sh "docker build -t ${image} ."
                    sh "aws ecr get-login-password --region us-west-2 | docker login --username AWS --password-stdin ${env.DOCKER_REGISTRY}"
                    sh "docker push ${image}"
                }
            }
        }

        stage('Deploy to EC2 Instance') {
            steps {
                script {
                    def shellCmd = "bash ./server-cmds.sh ${image}"
                    def ec2Instance = "ec2-user@10.0.7.81"

                    sshagent(['ec2devopskey']) {
                        sh "scp -o StrictHostKeyChecking=no server-cmds.sh ${ec2Instance}:/home/ec2-user"
                        sh "scp -o StrictHostKeyChecking=no docker-compose.yaml ${ec2Instance}:/home/ec2-user"
                        sh "ssh -o StrictHostKeyChecking=no ${ec2Instance} ${shellCmd}"
                    }
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
