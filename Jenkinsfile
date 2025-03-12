pipeline {
    agent any

    parameters {
        string(name: 'BUILD_TAG', defaultValue: 'latest', description: 'Tag for the Docker image')
    }

    environment {
        // ECR 환경 변수
        AWS_REGION = 'ap-northeast-2'
        AWS_CREDENTIAL_ID = 'jenkins-ecr'
        ECR_REPOSITORY = '718866409497.dkr.ecr.ap-northeast-2.amazonaws.com/nginx-test'
        IMAGE_NAME = "${ECR_REPOSITORY}:${BUILD_TAG}"

        // 배포 서버 환경 변수
        DEPLOY_SERVER_USER = 'ec2-user@'
        DEPLOY_SERVER_IP = '10.0.2.245'
        DEPLOY_CREDENTIAL = 'jenkins-ssh'
        DEPLOY_AWS_KEY_PATH = '/home/ec2-user/jkdev.pem'

        // Git 환경 변수
        GIT_REPOSITORY = 'https://github.com/your-repo/nginx-deploy.git'
        GIT_CREDENTIAL_ID = 'your-git-credentials'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', credentialsId: GIT_CREDENTIAL_ID, url: GIT_REPOSITORY
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${IMAGE_NAME} web/"
                }
            }
        }

        stage('Push Image to ECR') {
            steps {
                script {
                    sh 'aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_REPOSITORY'
                    sh "docker push ${IMAGE_NAME}"
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
