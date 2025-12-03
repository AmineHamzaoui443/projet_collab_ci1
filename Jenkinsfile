pipeline {
    agent any

    environment {
        REGISTRY = "icr.io/<TON_NAMESPACE>"           // Remplace par ton registry OpenShift/IBM
        IMAGE = "reservation-app-frontend"           // Nom de l'image Docker
        TAG = "latest"
        OPENSHIFT_TOKEN = credentials('openshift-token') // token OpenShift (non utilisé ici)
        REGISTRY_CRED = 'registry-cred'              // credentials Jenkins pour registry
        CC_CLI_TOKEN = credentials('codeclimate-token') // token CodeClimate
        SNYK_TOKEN = credentials('snyk-token')      // token Snyk
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
                // CodeClimate
                sh 'curl -L https://github.com/codeclimate/codeclimate/releases/download/0.90.0/codeclimate-0.90.0-linux-amd64.tar.bz2 | tar xjf -'
                sh './cc analyze --token=$CC_CLI_TOKEN'

                // Snyk
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
                        echo $PASS | docker login $REGISTRY -u $USER --password-stdin
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
            sh 'docker logout $REGISTRY'
        }
    }
}
