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

        stage('prepareToDeploy') {
            steps {
                sh """
                    sed -e 's;%GIT_HASH%;${GIT_HASH};g' taskdef-${DEPLOY_ENV}.json > \
                    taskdef-${GIT_HASH}.json

                    aws ecs register-task-definition --family ${PROJECT_NAME}-${DEPLOY_ENV} --cli-input-json file://taskdef-${GIT_HASH}.json
                """
            }
        }

        stage('deployToEcs') {
            steps {
                def TASK_REVISION = sh(returnStdout: true, script: "aws ecs describe-task-definition --task-definition ${PROJECT_NAME}-${DEPLOY_ENV} | egrep 'revision' | awk '{print \$2}'").trim().replace(",","")
                sh "aws ecs update-service --cluster ${PROJECT_NAME} --service ${DEPLOY_ENV} --task-definition ${PROJECT_NAME}-${DEPLOY_ENV}:${TASK_REVISION}"
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