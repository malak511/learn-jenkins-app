
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
            set +e
            npm test -- --watchAll=false --forceExit
            echo "npm test exit code: $?"
            ls -la test-results
            exit 0
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
