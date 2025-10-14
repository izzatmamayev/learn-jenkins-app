pipeline {
    agent any

    stages {
        stage('Without_Docker') {
            steps {
                sh '''
                    echo "Without Docker"
                    ls -la
                    touch container-no.txt
                '''
            }
        }
        
        stage('With_Docker') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    echo "With Docker"
                    ls -la
                    touch container-yes.txt
                    npm --version
                '''
            }
        }
    }
}
