
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
                    if [ -f docker-compose.yml ]; then
                        docker compose -f docker-compose.yml config
                    elif [ -f compose.yaml ]; then
                        docker compose -f compose.yaml config
                    elif [ -f compose.yml ]; then
                        docker compose -f compose.yml config
                    else
                        echo "ERROR: Docker Compose file not found"
                        exit 1
                    fi
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
                    echo "Checking Python files..."

                    docker run --rm \
                      -v "$WORKSPACE/backend:/app" \
                      -w /app \
                      python:3.12-slim \
                      sh -c '
                        find . -type f -name "*.py" \
                          -not -path "./.venv/*" \
                          | grep -q . || {
                            echo "ERROR: No Python files found"
                            exit 1
                          }

                        python -m compileall -q .
                      '
                '''
            }
        }

        stage('Frontend Build Validation') {
            steps {
                sh '''
                    echo "Checking package-lock.json..."

                    test -f frontend/package-lock.json || {
                        echo "ERROR: package-lock.json missing"
                        exit 1
                    }

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
    }

    post {
        failure {
            echo 'Apple Music CI failed. Check the console output.'
        }
    }
}
