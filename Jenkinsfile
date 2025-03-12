pipeline {
    agent any

    parameters {
        string(name: 'BUILD_TAG', defaultValue: 'latest', description: 'Tag for the Docker image')
    }

    environment {
        // ECR 환경 변수
        AWS_REGION = 'ap-northeast-2'
        ECR_REPOSITORY = '718866409497.dkr.ecr.ap-northeast-2.amazonaws.com/nginx-test'
        IMAGE_NAME = "${ECR_REPOSITORY}:${BUILD_TAG}"

        // 배포 서버 환경 변수
        DEPLOY_SERVER_USER = 'ec2-user@'
        DEPLOY_SERVER_IP = '10.0.2.245'
        DEPLOY_CREDENTIAL = 'jenkins-ssh'
        DEPLOY_AWS_KEY_PATH = '/home/ec2-user/jkdev.pem'

        // Git 환경 변수
        GIT_REPOSITORY = 'https://github.com/sangminpark9/web.git'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: GIT_REPOSITORY
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${IMAGE_NAME} ."
                }
            }
        }

        stage('Push Image to ECR') {
            steps {
                script {
                    echo 'Skipping ECR login for now, as it is not set up yet.'
                }
            }
        }

        stage('Deploy on EC2') {
            steps {
                script {
                    sshagent(credentials: [DEPLOY_CREDENTIAL]) {
                        sh "ssh -i ${DEPLOY_AWS_KEY_PATH} -o 'StrictHostKeyChecking no' ${DEPLOY_SERVER_USER}${DEPLOY_SERVER_IP} \"docker pull ${IMAGE_NAME} && docker run -d -p 80:80 ${IMAGE_NAME}\""
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment Successful!'
        }
        failure {
            echo 'Deployment Failed!'
        }
        always {
            cleanWs()
        }
    }
}
