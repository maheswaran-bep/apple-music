
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

        stage('Inspect Workspace') {
            steps {
                sh '''
                    echo "===== Workspace ====="
                    pwd
                    ls -la

                    echo "===== Backend Files ====="
                    find backend -maxdepth 3 -type f

                    echo "===== Frontend Files ====="
                    find frontend -maxdepth 2 -type f

                    echo "===== Compose Files ====="
                    find . -maxdepth 3 \\( -iname '*compose*.yml' -o -iname '*compose*.yaml' \\)
                '''
            }
        }

        stage('Validate Docker Compose') {
            steps {
                sh '''
                    docker compose config
                '''
            }
        }

        stage('Build Images') {
            steps {
                sh '''
                    docker compose build
                '''
            }
        }

        stage('Backend Syntax Check') {
            steps {
                sh '''
                    echo "Checking backend Python syntax..."

                    docker run --rm \
                      --entrypoint python \
                      apple-music-ci-cd-backend \
                      -m compileall -q /app

                    echo "Backend Python syntax check passed."
                '''
            }
        }

        stage('Frontend Build Validation') {
            steps {
                sh '''
                    echo "Checking frontend production build..."

                    docker run --rm \
                      --entrypoint sh \
                      apple-music-ci-cd-frontend \
                      -c "test -f /usr/share/nginx/html/index.html"

                    echo "Frontend production build verified."
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

        always {
            echo 'Apple Music pipeline execution finished.'
        }
    }
}
