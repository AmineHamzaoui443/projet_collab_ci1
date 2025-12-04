pipeline {
    agent any

    environment {
        // Nom de l'image Docker et registry (à adapter)
        DOCKER_IMAGE = "mon-utilisateur/mon-projet"
        REGISTRY = "docker.io" // ou ton registry
        VERSION = "${env.BUILD_NUMBER}"
    }

    stages {

        stage('Checkout & Install Dependencies') {
            steps {
                echo "🔄 Checkout du code"
                checkout scm

                echo "📦 Installation des dépendances npm"
                sh 'npm install'
            }
        }

        stage('Linting / Vérification code') {
            steps {
                echo "📝 Vérification de la structure du code avec ESLint"
                // Recherche automatique des fichiers à linter
                sh 'npx eslint . --ext .js,.ts,.tsx'
            }
        }

        stage('Unit Tests') {
            steps {
                echo "🧪 Lancement des tests unitaires"
                sh 'npm test'
            }
        }

        stage('Code Quality') {
            steps {
                echo "🔍 Analyse qualité du code avec SonarQube"
                withSonarQubeEnv('SonarQubeServer') {
                    sh 'sonar-scanner -Dsonar.projectKey=mon-projet -Dsonar.sources=. -Dsonar.host.url=$SONAR_HOST_URL -Dsonar.login=$SONAR_AUTH_TOKEN'
                }
            }
        }

        stage('Docker Build') {
            steps {
                echo "🐳 Construction de l'image Docker"
                sh "docker build -t ${DOCKER_IMAGE}:${VERSION} -f Dockerfile ."
                sh "docker tag ${DOCKER_IMAGE}:${VERSION} ${DOCKER_IMAGE}:latest"
            }
        }

        stage('Docker Image Scan') {
            steps {
                echo "🔒 Scan de l'image Docker avec Trivy"
                sh "trivy image --exit-code 1 --severity HIGH,CRITICAL ${DOCKER_IMAGE}:${VERSION}"
            }
        }

        stage('Push Docker Image') {
            steps {
                echo "🚀 Push de l'image Docker vers le registry"
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin ${REGISTRY}"
                    sh "docker push ${DOCKER_IMAGE}:${VERSION}"
                    sh "docker push ${DOCKER_IMAGE}:latest"
                }
            }
        }

    }

    post {
        success {
            echo "✅ Pipeline terminée avec succès. L'image Docker est disponible dans le registry."
        }
        failure {
            echo "❌ Pipeline échouée. Vérifier les logs pour plus de détails."
        }
    }
}
