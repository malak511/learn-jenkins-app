pipeline {
    agent any

    stages {

        stage('Build') {
            agent {
                docker {
                    image 'node:20-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    node --version
                    npm --version
                    npm ci
                    npm run build
                '''
            }
        }

        stage('Tests') {
            parallel {

                stage('Unit tests') {
                    agent {
                        docker {
                            image 'node:20-alpine'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            mkdir -p test-results
                            CI=true npx react-scripts test --watchAll=false --forceExit --reporters=default --reporters=jest-junit
                        '''
                    }
                    post {
                        always {
                            // jest-junit config in package.json writes to test-results/junit.xml
                            junit allowEmptyResults: true, testResults: 'test-results/junit.xml'
                        }
                    }
                }

                stage('E2E') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            # install serve outside the workspace so it can't disturb the unit tests running in parallel
                            npm install --prefix /tmp/serve serve
                            /tmp/serve/node_modules/.bin/serve -s build &
                            sleep 10
                            PLAYWRIGHT_HTML_OPEN=never npx playwright test --reporter=html
                        '''
                    }
                    post {
                        always {
                            // requires the "HTML Publisher" plugin
                            publishHTML([
                                allowMissing: true,
                                alwaysLinkToLastBuild: false,
                                keepAll: false,
                                reportDir: 'playwright-report',
                                reportFiles: 'index.html',
                                reportName: 'Playwright HTML Report',
                                reportTitles: '',
                                useWrapperFileDirectly: true
                            ])
                        }
                    }
                }
            }
        }

        stage('Deploy') {
            agent {
                docker {
                    image 'node:20-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm install netlify-cli@20.1.1
                    node_modules/.bin/netlify --version
                '''
            }
        }
    }
}