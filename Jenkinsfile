pipeline {
    agent any

    stages {
        stage('Build') {
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
                // Clean workspace before running npm
                cleanWs()

                sh '''
                    echo "===== Environment Info ====="
                    node --version
                    npm --version

                    echo "===== Cleaning npm cache ====="
                    npm cache clean --force

                    echo "===== Installing dependencies ====="
                    npm ci

                    echo "===== Building project ====="
                    npm run build

                    echo "===== Listing build output ====="
                    ls -la
                '''
            }
        }
    }
}
