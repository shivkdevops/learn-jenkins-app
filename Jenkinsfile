pipeline {
    agent any

    environment {
        HOME = "${WORKSPACE}"
        NPM_CONFIG_CACHE = "${WORKSPACE}/.npm"
    }
    
    stages {
        /*
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    ls -la
                    node -v
                    npm -v
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }
        */
        stage('Tests') {
            parallel {
                stage('Unit Tests') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    steps {
                        echo "Running tests..."
                        sh '''
                            test -f build/index.html
                            npm test
                        '''
                    }
                    post {
                        always {
                            junit 'test-results/junit.xml'
                        }
                    }
                }
                stage('E2E Tests') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.63.0-noble'
                            reuseNode true
                        }
                    }
                    steps {
                        echo "Running tests..."
                        sh '''
                            npm install serve
                            node_modules/.bin/serve -s build &
                            sleep 10
                            npx playwright test --reporter=html --output=e2e-results
                        '''
                    }
                    post {
                        always {
                            publishHTML(target: [
                                allowMissing: false,
                                alwaysLinkToLastBuild: true,
                                keepAll: true,
                                reportDir: 'e2e-results/html-report',
                                reportFiles: 'index.html',
                                reportName: 'E2E Test Report'
                            ]
                       )
                    }
                }
            }
        }
    }
}