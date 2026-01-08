pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
        buildDiscarder(logRotator(numToKeepStr: '10'))
        timeout(time: 30, unit: 'MINUTES')
    }

    environment {
        APP_NAME = 'my-app'
        DOCKER_IMAGE = 'my-org/my-app'
        REGISTRY_CREDENTIALS = 'dockerhub-creds'
        GIT_BRANCH = "${env.BRANCH_NAME}"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                  echo "Installing dependencies..."
                  npm install
                '''
            }
        }

        stage('Lint') {
            steps {
                sh '''
                  echo "Running lint..."
                  npm run lint
                '''
            }
        }

        stage('Test') {
            steps {
                sh '''
                  echo "Running tests..."
                  npm test
                '''
            }
        }

        stage('Build') {
            steps {
                sh '''
                  echo "Building application..."
                  npm run build
                '''
            }
        }

        stage('Docker Build') {
            when {
                branch 'main'
            }
            steps {
                sh """
                  docker build -t ${DOCKER_IMAGE}:${BUILD_NUMBER} .
                """
            }
        }

        stage('Docker Push') {
            when {
                branch 'main'
            }
            steps {
                withCredentials([usernamePassword(
                    credentialsId: REGISTRY_CREDENTIALS,
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh """
                      echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                      docker tag ${DOCKER_IMAGE}:${BUILD_NUMBER} ${DOCKER_IMAGE}:latest
                      docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}
                      docker push ${DOCKER_IMAGE}:latest
                    """
                }
            }
        }

        stage('Deploy') {
            when {
                branch 'main'
            }
            steps {
                sh '''
                  echo "Deploying application..."
                  # kubectl apply -f k8s/
                  # helm upgrade --install
                '''
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline succeeded'
        }
        failure {
            echo '❌ Pipeline failed'
        }
        always {
            cleanWs()
        }
    }
}
