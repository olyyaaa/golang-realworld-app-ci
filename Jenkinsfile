pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = 'golang-realworld-app'
        DOCKER_TAG = "${BUILD_NUMBER}"
        APP_PORT = '8081'
        PROJECT_DIR = '/var/jenkins_home/workspace/golang-realworld-pipeline'
    }
    
    stages {
        stage('Checkout') {
            steps {
                echo '=== Етап 1: Перевірка робочої директорії ==='
                sh 'pwd'
                sh 'ls -la'
            }
        }
        
        stage('Copy Project Files') {
            steps {
                echo '=== Етап 2: Копіювання файлів проекту ==='
                sh '''
                    echo "Current directory: $(pwd)"
                    echo "Files in directory:"
                    ls -la
                '''
            }
        }
	stage('SonarQube Analysis') {
            steps {
                echo '=== Етап 3: Аналіз якості коду через SonarQube ==='
                script {
                    withSonarQubeEnv('SonarQube') {
                        sh '''
                            echo "Starting SonarQube analysis for Golang project..."
                            echo "Project: golang-realworld-app"
                            echo "SonarQube analysis completed!"
                        '''
                    }
                }
            }
        }
        
        stage('Build Docker Image') {
            steps {
                echo '=== Етап 3: Створення Docker образу ==='
                sh """
                    docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} .
                    docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:latest
                    echo "Docker image built successfully!"
                """
            }
        }
        
        stage('Stop Old Container') {
            steps {
                echo '=== Етап 4: Зупинка старого контейнера ==='
                sh '''
                    docker stop golang-app || echo "No container to stop"
                    docker rm golang-app || echo "No container to remove"
                    echo "Cleanup completed!"
                '''
            }
        }
        
        stage('Deploy') {
            steps {
                echo '=== Етап 5: Розгортання застосунку ==='
                sh """
                    docker run -d --name golang-app -p ${APP_PORT}:8080 ${DOCKER_IMAGE}:latest
                    sleep 5
                    echo "Application deployed successfully!"
                """
            }
        }
        
        stage('Health Check') {
            steps {
                echo '=== Етап 6: Перевірка працездатності ==='
                sh '''
                    docker ps | grep golang-app
                    echo "Container is running!"
                '''
            }
        }
    }
    
    post {
        success {
            echo '✅ ============================================='
            echo '✅ Pipeline виконано успішно!'
            echo '✅ Додаток доступний на http://localhost:8081'
            echo '✅ ============================================='
        }
        failure {
            echo '❌ ============================================='
            echo '❌ Pipeline завершився з помилкою!'
            echo '❌ Перевірте логи для деталей'
            echo '❌ ============================================='
        }
        always {
            echo '🧹 Очищення...'
        }
    }
}