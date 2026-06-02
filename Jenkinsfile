pipeline {
    agent { label 'redhat' } 
    
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-creds')
        DOCKER_USER = "doniasobhy" 
        IMAGE_NAME = "${DOCKER_USER}/simple-java-app"
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {
        stage('Build App') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Test App') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Build Image') {
            steps {
                sh 'docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .'
                sh 'docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${IMAGE_NAME}:latest'
            }
        }

        stage('Push DockerHub') {
            steps {
                // جينكينز هيحاول 3 مرات لو حصل أي إيرور في الشبكة
                retry(3) { 
                    sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
                    sh 'docker push ${IMAGE_NAME}:${IMAGE_TAG}'
                    sh 'docker push ${IMAGE_NAME}:latest'
                }
            }
        }
    }
    
    post {
        always {
            // تنظيف وتشغيل الكونتينر الجديد
            sh 'docker logout'
            sh 'docker stop java-app || true'
            sh 'docker rm java-app || true'
            sh 'docker run -d --name java-app -p 8081:8080 ${IMAGE_NAME}:latest'
        }
    }
}
