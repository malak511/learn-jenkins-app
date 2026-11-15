
pipeline {
    agent {
        docker {
            image 'node:20-alpine'
        }
    }

    stages {
        stage('Build') {
            steps {
                sh '''
                    npm ci
                    npm run build
                '''
            }
        }

        stage('Test') {
            steps {
                sh '''
                    mkdir -p test-results
                    CI=true JEST_JUNIT_OUTPUT_DIR=./test-results JEST_JUNIT_OUTPUT_NAME=junit.xml npm test -- --watchAll=false
                '''
            }
        }
    }

    post {
        always {
            junit 'test-results/junit.xml'
        }
    }
}
