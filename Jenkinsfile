pipeline {
    agent any

    stages {
        // This is a comment
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                    //arg '-u root:root'   //this is how we can specify different user, but don not user root user, that is not not a good practice
                }
            }
            environment {
                NPM_CONFIG_CACHE = "${WORKSPACE}/.npm-cache"
            }
            steps {
                sh '''
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }

        stage('Tests') {
            parallel {
                stage('Unit test') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    //environment {
                        //NPM_CONFIG_CACHE = "${WORKSPACE}/.npm-cache"
                    //}
                    // To comment out sh command # should be in front of a command like this #test -f build/index.html
                    steps {
                        sh '''
                            test -f build/index.html
                            npm test
                        '''
                    }
                    post {
                        always {
                            junit 'jest-results/junit.xml'
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
                     environment {
                        NPM_CONFIG_CACHE = "${WORKSPACE}/.npm-cache"
                    }
                    //originally (npm install -g serve, serve -s build) we try to install serve as global dependency, but instead of global we just use local dependency
                    steps {
                        sh '''
                            npm install serve
                            node_modules/.bin/serve -s build &
                            sleep 10
                            npx playwright test --reporter=html
                        '''
                    }
                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }

        stage('Deploy') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            environment {
                NPM_CONFIG_CACHE = "${WORKSPACE}/.npm-cache"
            }
            steps {
                sh '''
                    npm install netlify-cli@22.1.3
                    node_modules/.bin/netlify --version
                '''
            }
        }
    }
}
