pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID='fb6cc5e6-da5c-4e0d-89fb-56790b350d32'
        NETLIFY_AUTH_TOKEN=credentials('netlify-token')
    }

    stages {
        // This is a comment from Jenkinsfile
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
                NETLIFY_HOME = "${WORKSPACE}/.netlify"
            }
            steps {
                sh '''
                    echo 'Small change'
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
                            sleep 7
                            npx playwright test --reporter=html
                        '''
                    }
                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Local Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }

        stage('Deploy staging') {
            agent {
                docker {
                    image 'node:20-alpine'
                    args "-v /etc/passwd:/etc/passwd"
                    reuseNode true
                }
            }
            environment {
                HOME = "${WORKSPACE}"
                NPM_CONFIG_CACHE = "${WORKSPACE}/.npm-cache"
                NETLIFY_HOME = "${WORKSPACE}/.netlify"
            }
            steps {
                sh '''
                    npm install netlify-cli@23.9.1 node-jq
                    node_modules/.bin/netlify --version
                    echo "Deploying to staging. Site ID: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --no-build --dir=build --json > deploy-output.json
                '''
                script {
                    env.STAGING_URL = sh (
                        script: "node_modules/.bin/node-jq -r '.deploy_url' deploy-output.json",
                        returnStdout: true
                    )
                }
            }    
        }

        stage('Staging E2E') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }
            environment {
                NPM_CONFIG_CACHE = "${WORKSPACE}/.npm-cache"
                CI_ENVIRONMENT_URL = "${env.STAGING_URL}"
            }
            steps {
                sh '''
                    npx playwright test --reporter=html
                '''
            }
            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Staging E2E Report', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }

        stage('Approval') {
            steps {
                timeout(time: 15, unit: 'MINUTES') {
                      input message: 'Do you want to deploy to production ?', ok: 'Yes, I am sure I want to deploy'
                }
            }
        }

        stage('Deploy production') {
            agent {
                docker {
                    image 'node:20-alpine'
                    args "-v /etc/passwd:/etc/passwd"
                    reuseNode true
                }
            }
            environment {
                HOME = "${WORKSPACE}"
                NPM_CONFIG_CACHE = "${WORKSPACE}/.npm-cache"
                NETLIFY_HOME = "${WORKSPACE}/.netlify"
            }
            steps {
                sh '''
                    npm install netlify-cli@23.9.1
                    node_modules/.bin/netlify --version
                    echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --no-build --dir=build --prod
                '''
            }
        }
        
        stage('Prod E2E') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }
            environment {
                NPM_CONFIG_CACHE = "${WORKSPACE}/.npm-cache"
                CI_ENVIRONMENT_URL = 'https://starlit-meringue-44391a.netlify.app'
            }
            steps {
                sh '''
                    npx playwright test --reporter=html
                '''
            }
            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Production E2E Report', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }
    }
}
