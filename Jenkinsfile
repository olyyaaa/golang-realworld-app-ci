pipeline {
    agent any

    environment {
        DOCKER_IMAGE   = 'golang-realworld-app'
        DOCKER_TAG     = "${BUILD_NUMBER}"
        APP_PORT       = '8082'           // 8081 зайнятий Jenkins
        PROJECT_DIR    = "${WORKSPACE}"

        SONAR_URL      = 'http://host.docker.internal:9000'
        PROM_URL       = 'http://host.docker.internal:9090'
        GRAFANA_URL    = 'http://host.docker.internal:3000'
    }

    stages {
        stage('Checkout') {
            steps {
                echo '=== Етап 1: Перевірка робочої директорії ==='
                sh 'pwd'
                sh 'ls -la'
            }
        }

        stage('SonarQube Health Check') {
            steps {
                echo '=== Етап 2: Перевірка доступності SonarQube ==='
                sh """
                    echo "Перевіряємо SonarQube: ${SONAR_URL}"
                    STATUS=\$(curl -s -o /dev/null -w "%{http_code}" "${SONAR_URL}/api/system/health" || echo 000)
                    echo "HTTP статус SonarQube: \$STATUS"

                    if [ "\$STATUS" != "200" ] && [ "\$STATUS" != "403" ]; then
                      echo "❌ SonarQube недоступний або не відповідає очікувано!"
                      exit 1
                    fi

                    echo "✅ SonarQube працює (отримали статус \$STATUS)."
                """
            }
        }

        stage('Build Docker Image (demo)') {
            steps {
                echo '=== Етап 3: Створення Docker образу (ДЕМО) ==='
                sh """
                    echo "Тут у реальному середовищі виконується:"
                    echo "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ."
                    echo "docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:latest"
                """
            }
        }

        stage('Deploy (demo)') {
            steps {
                echo '=== Етап 4: Розгортання застосунку (ДЕМО) ==='
                sh """
                    echo "Тут у реальному середовищі виконується:"
                    echo "docker stop golang-app || true"
                    echo "docker rm golang-app || true"
                    echo "docker run -d --name golang-app -p ${APP_PORT}:8080 ${DOCKER_IMAGE}:latest"
                """
            }
        }

        stage('Monitoring Check (Prometheus & Grafana)') {
            steps {
                echo '=== Етап 5: Перевірка моніторингу (Prometheus, Grafana) ==='
                sh """
                    echo "Перевіряємо Prometheus: ${PROM_URL}"
                    P_STATUS=\$(curl -s -o /dev/null -w "%{http_code}" "${PROM_URL}/-/healthy" || echo 000)
                    echo "HTTP статус Prometheus: \$P_STATUS"

                    echo "Перевіряємо Grafana: ${GRAFANA_URL}"
                    G_STATUS=\$(curl -s -o /dev/null -w "%{http_code}" "${GRAFANA_URL}/login" || echo 000)
                    echo "HTTP статус Grafana: \$G_STATUS"

                    if [ "\$P_STATUS" != "200" ]; then
                      echo "❌ Prometheus недоступний!"
                      exit 1
                    fi

                    if [ "\$G_STATUS" != "200" ]; then
                      echo "❌ Grafana недоступна!"
                      exit 1
                    fi

                    echo "✅ Prometheus і Grafana доступні з Jenkins."
                """
            }
        }
    }

    post {
        success {
            echo '✅ ============================================='
            echo '✅ Pipeline виконано успішно!'
            echo "✅ SonarQube: ${SONAR_URL}"
            echo "✅ Prometheus: ${PROM_URL}"
            echo "✅ Grafana: ${GRAFANA_URL}"
            echo '✅ ============================================='
        }
        failure {
            echo '❌ ============================================='
            echo '❌ Pipeline завершився з помилкою!'
            echo '❌ Перевірте логи для деталей'
            echo '❌ ============================================='
        }
        always {
            echo '🧹 Завершення роботи pipeline'
        }
    }
}
