pipeline {
    agent any

    environment {
        DOCKERHUB_USER = "vrush13"
        AWS_REGION     = "ap-south-1"
        AWS_ACCOUNT_ID = "621703783626"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    url:'https://github.com/Vrushk13/Apache-site.git'
            }
        }

        /* ===================== TASK 1 ===================== */
        stage('Build Apache Image') {
            steps {
                dir('apache') {
                    sh 'docker build -t apache-site .'
                }
            }
        }

        stage('Push Apache Image') {
            steps {
                script {
                    dockerPush('apache-site')
                }
            }
        }

        /* ===================== TASK 2 ===================== */
        stage('Build Node API Image') {
            steps {
                dir('node-api') {
                    sh 'docker build -t node-api .'
                }
            }
        }

        stage('Push Node API Image') {
            steps {
                script {
                    dockerPush('node-api')
                }
            }
        }

        /* ===================== TASK 3 ===================== */
        stage('Build Python App Image') {
            steps {
                dir('python-app') {
                    sh 'docker build -t python-app .'
                }
            }
        }

        stage('Push Python Image') {
            steps {
                script {
                    dockerPush('python-app')
                }
            }
        }

        /* ===================== TASK 4 ===================== */
        stage('MySQL Volume Test') {
            steps {
                sh '''
                docker run -d --name mysql-novol -e MYSQL_ROOT_PASSWORD=root mysql:8
                sleep 20
                docker rm -f mysql-novol

                docker volume create mysql-data
                docker run -d --name mysql-vol -e MYSQL_ROOT_PASSWORD=root \
                -v mysql-data:/var/lib/mysql mysql:8
                sleep 20
                docker rm -f mysql-vol
                '''
            }
        }
    }

    post {
        always {
            sh 'docker system prune -f'
        }
    }
}

/* ===================== FUNCTIONS ===================== */
def dockerPush(imageName) {

    withCredentials([usernamePassword(
        credentialsId: 'dockerhub-creds',
        usernameVariable: 'DOCKER_USER',
        passwordVariable: 'DOCKER_PASS'
    )]) {
        sh """
        docker tag ${imageName} ${DOCKERHUB_USER}/${imageName}:latest
        echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin
        docker push ${DOCKERHUB_USER}/${imageName}:latest
        """
    }

    withCredentials([[$class: 'AmazonWebServicesCredentialsBinding',
        credentialsId: 'aws-creds'
    ]]) {
        sh """
        docker tag ${imageName} ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${imageName}:latest

        aws ecr get-login-password --region ${AWS_REGION} |
        docker login --username AWS --password-stdin \
        ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com

        docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${imageName}:latest
        """
    }
}
