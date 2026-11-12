
pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Validate Docker Compose') {
            steps {
                sh 'docker compose config'
            }
        }

        stage('Build Images') {
            steps {
                sh 'docker compose build'
            }
        }

        stage('Backend Syntax Check') {
            steps {
                sh '''
                    docker run --rm \
                      -v "$WORKSPACE/backend:/app" \
                      -w /app \
                      python:3.12-slim \
                      python -m compileall -q app
                '''
            }
        }

        stage('Frontend Build Validation') {
            steps {
                sh '''
                    docker run --rm \
                      -v "$WORKSPACE/frontend:/app" \
                      -w /app \
                      node:22-alpine \
                      sh -c "npm ci && npm run build"
                '''
            }
        }
    }

    post {
        success {
            echo 'Apple Music CI completed successfully!'
        }
        failure {
            echo 'Apple Music CI failed. Check the console output.'
        }
    }
}
