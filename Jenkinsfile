pipeline {
    agent any

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Build Apache Image') {
            steps {
                sh 'docker build -t apache-site .'
            }
        }

        stage('Build Node API Image') {
            steps {
                dir('node-api') {
                    sh 'docker build -t node-api .'
                }
            }
        }

        stage('Build Python App Image') {
            steps {
                dir('python-app') {
                    sh 'docker build -t python-app .'
                }
            }
        }
    }

post {
    always {
        sh 'docker system prune -f --volumes'
    }
}

}

