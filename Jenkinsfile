pipeline {
    agent any
    stages {
        stage('Clone Repository') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh """
                    docker login -u AWS -p \$(aws ecr get-login-password --region ap-northeast-1) 262429377270.dkr.ecr.ap-northeast-1.amazonaws.com
                    docker build -t ${REPOSITORY_URI}:${GIT_HASH} .
                """
            }
        }

        stage('ImagePush') {
            steps {
                sh """
                    docker push ${REPOSITORY_URI}:${GIT_HASH}
                    docker rmi ${REPOSITORY_URI}:${GIT_HASH}
                """
            }
        }

    environment {
        REPOSITORY_URI = 'public.ecr.aws/s8h8u3c8/fastapi-demoecs'
        GIT_HASH = "${GIT_COMMIT[0..6]}"
        PROJECT_NAME = "fastapi-DemoECS"
        DEPLOY_ENV = "main"
    }
}
}