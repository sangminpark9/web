pipeline {
    agent any

    parameters {
        string(name: 'BUILD_TAG', defaultValue: 'latest', description: 'Tag for the Docker image')
    }

    environment {
        // ECR 환경 변수 - 올바른 계정 ID로 수정
        AWS_REGION = 'ap-northeast-2'
        ECR_REPOSITORY = '476114142897.dkr.ecr.ap-northeast-2.amazonaws.com/nginx-test'
        IMAGE_NAME = "${ECR_REPOSITORY}:${BUILD_TAG}"

        // 배포 서버 환경 변수 (Private Subnet - web)
        DEPLOY_SERVER_USER = 'ec2-user'
        DEPLOY_SERVER_IP = '10.0.1.134' // Private Subnet의 Web 서버
        DEPLOY_CREDENTIAL = 'jenkins-ssh' // Jenkins에 등록된 SSH 자격 증명 ID

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
                    sh '''
                    aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REPOSITORY}
                    docker push ${IMAGE_NAME}
                    '''
                }
            }
        }

        stage('Deploy on Private Subnet Web Server') {
            steps {
                script {
                    sshagent(credentials: [DEPLOY_CREDENTIAL]) {
                        // StrictHostKeyChecking 옵션 추가
                        sh """
                        ssh -o StrictHostKeyChecking=no ${DEPLOY_SERVER_USER}@${DEPLOY_SERVER_IP} '
                            aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REPOSITORY} && 
                            docker pull ${IMAGE_NAME} && 
                            docker stop \$(docker ps -q --filter ancestor=${IMAGE_NAME} 2>/dev/null) 2>/dev/null || true && 
                            docker run -d -p 80:80 --name nginx-app-\$(date +%s) ${IMAGE_NAME}
                        '
                        """
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
            sh "docker rmi ${IMAGE_NAME} || true"
        }
    }
}
