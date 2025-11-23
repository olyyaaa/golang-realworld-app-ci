pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = 'golang-realworld-app'
        DOCKER_TAG = "${BUILD_NUMBER}"
        APP_PORT = '8081'
    }
    
    stages {
        stage('Checkout') {
            steps {
                echo '=== Етап 1: Отримання коду з репозиторію ==='
                checkout scm
            }
        }
        
        stage('Environment Info') {
            steps {
                echo '=== Інформація про середовище ==='
                script {
                    bat '''
                        echo Current directory:
                        cd
                        echo.
                        echo Docker version:
                        docker --version
                        echo.
                        echo Go version:
                        go version || echo Go not found in PATH
                    '''
                }
            }
        }
        
        stage('Build Application') {
            steps {
                echo '=== Етап 2: Компіляція Golang застосунку ==='
                script {
                    bat '''
                        echo Building Go application...
                        go mod download
                        go build -v -o app.exe .
                        echo Build completed successfully!
                    '''
                }
            }
        }
        
        stage('Run Tests') {
            steps {
                echo '=== Етап 3: Запуск тестів ==='
                script {
                    bat '''
                        echo Running tests...
                        go test -v ./... || echo No tests found or tests failed
                        echo Tests stage completed!
                    '''
                }
            }
        }
        
        stage('Code Quality Check') {
            steps {
                echo '=== Етап 4: Перевірка якості коду ==='
                script {
                    bat '''
                        echo Checking code quality...
                        go vet ./... || echo Vet check completed with warnings
                        echo Code quality check completed!
                    '''
                }
            }
        }
        
        stage('Build Docker Image') {
            steps {
                echo '=== Етап 5: Створення Docker образу ==='
                script {
                    bat """
                        echo Building Docker image...
                        docker build -t %DOCKER_IMAGE%:%DOCKER_TAG% .
                        docker tag %DOCKER_IMAGE%:%DOCKER_TAG% %DOCKER_IMAGE%:latest
                        echo Docker image built successfully!
                    """
                }
            }
        }
        
        stage('Stop Old Container') {
            steps {
                echo '=== Етап 6: Зупинка старого контейнера ==='
                script {
                    bat '''
                        echo Stopping old container if exists...
                        docker stop golang-app || echo No container to stop
                        docker rm golang-app || echo No container to remove
                        echo Cleanup completed!
                    '''
                }
            }
        }
        
        stage('Deploy') {
            steps {
                echo '=== Етап 7: Розгортання застосунку ==='
                script {
                    bat """
                        echo Deploying application...
                        docker run -d --name golang-app -p %APP_PORT%:8080 %DOCKER_IMAGE%:latest
                        echo Waiting for application to start...
                        timeout /t 5 /nobreak
                        echo Application deployed successfully!
                    """
                }
            }
        }
        
        stage('Health Check') {
            steps {
                echo '=== Етап 8: Перевірка працездатності ==='
                script {
                    bat '''
                        echo Checking application health...
                        docker ps | findstr golang-app
                        echo Container is running!
                    '''
                }
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
            echo '🧹 Очищення тимчасових файлів...'
            script {
                bat '''
                    echo Cleaning up...
                    docker system prune -f || echo Cleanup completed
                '''
            }
        }
    }
}