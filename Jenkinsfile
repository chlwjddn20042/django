pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = 'dockerhub-cred'   // Docker Hub 크리덴셜 ID
        IMAGE_NAME            = 'chlwjddn/django'  // Docker Hub 리포지토리 이름
    }

    // Git 변경 감지 트리거
    triggers {
        // 1분마다 Git 레포 변경 여부 체크
        pollSCM('H/1 * * * *')
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Jenkins 빌드 번호를 태그로 사용
                    def imageTag = "${env.BUILD_NUMBER}"
                    bat """
                    docker build -t ${IMAGE_NAME}:${imageTag} -t ${IMAGE_NAME}:latest .
                    """
                    env.IMAGE_TAG = imageTag
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    withCredentials([
                        usernamePassword(
                            credentialsId: DOCKERHUB_CREDENTIALS,
                            usernameVariable: 'DOCKERHUB_USER',
                            passwordVariable: 'DOCKERHUB_PASS'
                        )
                    ]) {
                        bat """
                        docker login -u %DOCKERHUB_USER% -p %DOCKERHUB_PASS%
                        docker push ${IMAGE_NAME}:${IMAGE_TAG}
                        docker push ${IMAGE_NAME}:latest
                        docker logout
                        """
                    }
                }
            }
        }
    }

    post {
        always {
            bat 'docker image prune -f || ver>nul'
        }
    }
}

