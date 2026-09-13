
pipeline {
    agent any

    environment {
        APP_NAME = 'multi-branch-app'
        REGISTRY_CREDENTIALS = 'docker'
    }

    stages {
        stage('Validate branch') {
            steps {
                script {
                    if (!(env.BRANCH_NAME in ['dev', 'stg', 'main'])) {
                        error("Unsupported branch '${env.BRANCH_NAME}'. Use dev, stg, or main.")
                    }
                }
            }
        }

        stage('Test') {
            steps {
                sh 'pwd && ls -la && git status && git ls-files'
                sh 'node --check index-1.js'
            }
        }

        stage('Build image') {
            steps {
                script {
                    env.IMAGE_TAG = "${env.BRANCH_NAME}-${env.BUILD_NUMBER}"
                    sh 'docker build --pull -t $APP_NAME:$IMAGE_TAG .'
                }
            }
        }

        stage('Push image') {
            when {
                anyOf {
                    branch 'dev'
                    branch 'stg'
                    branch 'main'
                }
            }
            steps {
                script {
                    withCredentials([usernamePassword(
                        credentialsId: env.REGISTRY_CREDENTIALS,
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )]) {
                        sh '''
                            set +x
                            echo "$DOCKER_PASSWORD" | docker login --username "$DOCKER_USERNAME" --password-stdin
                            docker tag "$APP_NAME:$IMAGE_TAG" "$DOCKER_USERNAME/$APP_NAME:$IMAGE_TAG"
                            docker push "$DOCKER_USERNAME/$APP_NAME:$IMAGE_TAG"
                            docker logout
                        '''
                    }
                }
            }
        }

        stage('Deploy') {
            when {
                branch 'main'
            }
            steps {
                echo 'main image is ready for production deployment'
            }
        }
    }

    post {
        always {
            sh 'docker image rm "$APP_NAME:$IMAGE_TAG" 2>/dev/null || true'
        }
    }
}
