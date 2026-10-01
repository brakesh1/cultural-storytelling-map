pipeline {

    agent any

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }


        stage('Verify Files') {
            steps {
                sh '''
                    set -eu

                    echo "Commit:"
                    git rev-parse --short HEAD

                    test -f frontend/Dockerfile
                    test -f frontend/nginx.conf
                    test -f backend/Dockerfile
                    test -f docker-compose.yml

                    echo "Required deployment files found."
                '''
            }
        }


        stage('Docker Check') {
            steps {
                sh '''
                    docker --version
                    docker compose version
                    docker info >/dev/null
                '''
            }
        }


        stage('Compose Validation') {
            steps {
                sh '''
                    docker compose \
                      --env-file /home/ubuntu/cultural-storytelling-map/.env \
                      config -q

                    echo "Compose configuration is valid."
                '''
            }
        }


        stage('Build') {
            steps {
                sh '''
                    docker compose \
                      --env-file /home/ubuntu/cultural-storytelling-map/.env \
                      build
                '''
            }
        }


        stage('Deploy') {
            steps {
                sh '''
                    docker compose \
                      --env-file /home/ubuntu/cultural-storytelling-map/.env \
                      up -d
                '''
            }
        }


        stage('Status') {
            steps {
                sh '''
                    docker compose \
                      --env-file /home/ubuntu/cultural-storytelling-map/.env \
                      ps
                '''
            }
        }


        stage('Health Check') {
            steps {
                sh '''
                    sleep 10

                    curl --fail \
                         --retry 10 \
                         --retry-delay 3 \
                         --retry-connrefused \
                         http://localhost/

                    echo
                    echo "Narrify backend is healthy."
                '''
            }
        }
    }


    post {

        success {
            echo 'Narrify deployed successfully.'
        }


        failure {

            echo 'Deployment failed. Showing diagnostics.'

            sh '''
                docker compose \
                  --env-file /home/ubuntu/cultural-storytelling-map/.env \
                  ps || true

                echo "===== BACKEND LOGS ====="

                docker compose \
                  --env-file /home/ubuntu/cultural-storytelling-map/.env \
                  logs --tail=100 backend || true

                echo "===== FRONTEND LOGS ====="

                docker compose \
                  --env-file /home/ubuntu/cultural-storytelling-map/.env \
                  logs --tail=100 frontend || true
            '''
        }
    }
}
