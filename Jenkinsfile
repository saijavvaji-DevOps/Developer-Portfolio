pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    environment {
        APP_DIR = '/opt/developer-portfolio/Developer-Portfolio'
        COMPOSE_PROJECT_NAME = 'developer-portfolio'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Validate Files') {
            steps {
                sh '''
                    test -f docker-compose.yml
                    test -f App_Server/Dockerfile
                    test -f Portfolio/Dockerfile
                    test -f Portfolio/nginx.conf
                    docker compose version
                '''
            }
        }

        stage('Sync Source') {
            steps {
                sh '''
                    rsync -a --delete \
                      --exclude '.git' \
                      --exclude '.env' \
                      ./ "$APP_DIR"/
                '''
            }
        }

        stage('Build Images') {
            steps {
                sh '''
                    cd "$APP_DIR"
                    docker compose build --pull
                '''
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    cd "$APP_DIR"
                    docker compose up -d --remove-orphans
                '''
            }
        }

        stage('Health Check') {
            steps {
                sh '''
                    cd "$APP_DIR"
                    sleep 15
                    docker compose ps
                    curl --fail --retry 5 --retry-delay 5 http://localhost/
                '''
            }
        }
    }

    post {
        success {
            echo 'Deployment completed successfully.'
        }

        failure {
            echo 'Deployment failed.'
            sh '''
                cd "$APP_DIR"
                docker compose ps || true
                docker compose logs --tail=100 || true
            '''
        }
    }
}
