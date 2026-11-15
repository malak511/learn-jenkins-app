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
                            SERVE_PID=$!
                            sleep 10
                            PLAYWRIGHT_HTML_OPEN=never npx playwright test --reporter=html
                            RESULT=$?
                            kill $SERVE_PID || true
                            exit $RESULT
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
                    // Debian-based image: avoids the musl-specific sharp/libvips download
                    image 'node:20'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm config set fetch-retries 5
                    npm config set fetch-retry-mintimeout 20000
                    npm config set fetch-timeout 180000

                    installed=false

                    # Attempt 1-2: default sharp download (GitHub)
                    for i in 1 2; do
                        echo "Install attempt $i (default host)"
                        if npm install netlify-cli@20.1.1; then
                            installed=true
                            break
                        fi
                        sleep 5
                    done

                    # Attempt 3-4: fall back to a mirror for the sharp-libvips binary
                    if [ "$installed" = "false" ]; then
                        export npm_config_sharp_libvips_binary_host="https://npmmirror.com/mirrors/sharp-libvips"
                        for i in 3 4; do
                            echo "Install attempt $i (mirror host)"
                            if npm install netlify-cli@20.1.1; then
                                installed=true
                                break
                            fi
                            sleep 5
                        done
                    fi

                    if [ "$installed" = "false" ]; then
                        echo "netlify-cli installation failed after all attempts"
                        exit 1
                    fi

                    node_modules/.bin/netlify --version
                '''
            }
        }
    }
}