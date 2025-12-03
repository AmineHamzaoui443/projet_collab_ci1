pipeline {
    agent any

    environment {
        REGISTRY = "ghcr.io/<TON_COMPTE>"          // GitHub Container Registry
        IMAGE = "reservation-app-frontend"
        TAG = "latest"
        REGISTRY_CRED = 'github-packages-cred'     // Credentials Jenkins pour GitHub (username + PAT)
        CC_CLI_TOKEN = credentials('codeclimate-token')
        SNYK_TOKEN = credentials('snyk-token')
    }

    stages {
        stage('Checkout & Install') {
            steps {
                git branch: 'main', url: 'https://github.com/<TON_COMPTE>/<TON_REPO>.git'
                sh 'npm install'
            }
        }

        stage('Code Structure Check (Rome)') {
            steps {
                sh 'npx rome check'
            }
        }

        stage('Run Tests (Vitest)') {
            steps {
                sh 'npx vitest run --coverage'
            }
        }

        stage('Code Quality & Security') {
            steps {
                sh 'curl -L https://github.com/codeclimate/codeclimate/releases/download/0.90.0/codeclimate-0.90.0-linux-amd64.tar.bz2 | tar xjf -'
                sh './cc analyze --token=$CC_CLI_TOKEN'
                
                sh 'npm install -g snyk'
                sh 'snyk auth $SNYK_TOKEN'
                sh 'snyk test'
            }
        }

        stage('Docker Build') {
            steps {
                sh "docker build -t $REGISTRY/$IMAGE:$TAG ."
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: REGISTRY_CRED, usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh '''
                        echo $PASS | docker login ghcr.io -u $USER --password-stdin
                        docker tag $REGISTRY/$IMAGE:$TAG $REGISTRY/$IMAGE:latest
                        docker push $REGISTRY/$IMAGE:$TAG
                        docker push $REGISTRY/$IMAGE:latest
                    '''
                }
            }
        }
    }

    post {
        always {
            sh 'docker logout ghcr.io'
        }
    }
}
