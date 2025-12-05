pipeline {
    agent any

    stages {

        stage('Checkout & Install Dependencies') {
            steps {
                echo "🔄 Checkout du code"
                checkout scm

                echo "📦 Installation des dépendances npm"
                sh 'npm install'
            }
        }

        stage('Linting') {
            steps {
                echo "📝 Lint du code"
                sh 'npx eslint . --ext .ts,.tsx,.js'
            }
        }

        stage('Unit Tests') {
            steps {
                echo "🧪 Tests unitaires"
                sh 'npm test'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                echo "🔍 Analyse SonarQube"

                withSonarQubeEnv('SonarQubeServer') {
                    sh '''
                        sonar-scanner \
                        -Dsonar.projectKey=reservation_front \
                        -Dsonar.sources=./src \
                        -Dsonar.host.url=http://localhost:9000 \
                        -Dsonar.login=squ_c1bde0c3e8e92658fb22f543c11e7d1412ee24e1
                    '''
                }
            }
        }

        stage('Docker Build') {
            steps {
                echo "🐳 Build de l’image Docker"
                sh '''
                    docker build -t ghcr.io/AmineHamzaoui443/reservation-frontend:latest .
                '''
            }
        }

        stage('Trivy Security Scan') {
            steps {
                echo "🔒 Scan Trivy"
                sh '''
                    trivy image --exit-code 1 --severity HIGH,CRITICAL ghcr.io/AmineHamzaoui443/reservation-frontend:latest
                '''
            }
        }

        stage('Push Docker Image') {
            steps {
                echo "🚀 Push vers GitHub Container Registry"

                sh '''
                    echo "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" | docker login ghcr.io -u AmineHamzaoui443 --password-stdin
                    docker push ghcr.io/AmineHamzaoui443/reservation-frontend:latest
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline complète : Lint + Tests + Sonar + Docker + Trivy + Push !"
        }
        failure {
            echo "❌ Pipeline échouée. Vérifie les logs."
        }
    }
}
