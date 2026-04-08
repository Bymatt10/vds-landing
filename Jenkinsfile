pipeline {
    agent any

    environment {
        IMAGE_NAME     = 'vds-landing'
        CONTAINER_NAME = 'vds-landing'
        HOST_PORT      = '3000'
        COMPOSE_FILE   = 'docker-compose.yml'
    }

    options {
        buildDiscarder(logRotator(numToKeepStr: '10'))
        timeout(time: 15, unit: 'MINUTES')
        disableConcurrentBuilds()
        timestamps()
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
                echo "Branch : ${env.GIT_BRANCH}"
                echo "Commit : ${env.GIT_COMMIT}"
            }
        }

        stage('Build') {
            steps {
                echo "Building Docker image: ${IMAGE_NAME}"
                sh 'docker compose -f ${COMPOSE_FILE} build --no-cache --pull'
            }
        }

        stage('Deploy') {
            steps {
                echo "Stopping existing container (if any)"
                sh 'docker compose -f ${COMPOSE_FILE} down --remove-orphans || true'
                echo "Starting new container"
                sh 'docker compose -f ${COMPOSE_FILE} up -d'
                sh 'sleep 5'
            }
        }

        stage('Health Check') {
            steps {
                echo "Verifying service on localhost:${HOST_PORT}"
                sh '''
                    for i in 1 2 3 4 5; do
                        echo "Attempt $i / 5 ..."
                        if curl --silent --fail --max-time 10 http://localhost:${HOST_PORT}; then
                            echo ""
                            echo "Health check PASSED."
                            exit 0
                        fi
                        sleep 6
                    done
                    echo "Health check FAILED after 5 attempts."
                    exit 1
                '''
            }
        }
    }

    post {
        always {
            sh 'docker image prune -f || true'
        }
        success {
            echo """
============================================================
  DEPLOYMENT SUCCEEDED
  Image    : ${IMAGE_NAME}
  Container: ${CONTAINER_NAME}
  Port     : ${HOST_PORT}
  Commit   : ${env.GIT_COMMIT}
  Build #  : ${env.BUILD_NUMBER}
  URL      : https://vdstecnologia.com
============================================================
"""
        }
        failure {
            echo """
============================================================
  DEPLOYMENT FAILED — Build #${env.BUILD_NUMBER}
  Check the logs above for the failing stage.
  Previous container (if any) may still be running.
============================================================
"""
        }
    }
}
