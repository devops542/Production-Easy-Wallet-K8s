pipeline {

    agent any

    tools {
        maven 'M3'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh '''
                    mvn org.sonarsource.scanner.maven:sonar-maven-plugin:sonar \
                    -Dsonar.projectKey=Production-Easy-Wallet-K8s
                    '''
                }
            }
        }

        stage('Copy files to Docker Server') {
            steps {
                sshagent(credentials: ['docker-server']) {
                    sh '''
                    scp -o StrictHostKeyChecking=no Dockerfile \
                    ec2-user@172.31.37.89:/home/ec2-user/docker-build/

                    scp -o StrictHostKeyChecking=no target/Production-Easy-Wallet-K8s.war \
                    ec2-user@172.31.37.89:/home/ec2-user/docker-build/target/
                    '''
                }
            }
        }

        stage('Build and Deploy Docker Image') {
            steps {
                sshagent(credentials: ['docker-server']) {
                    sh """
                    ssh -o StrictHostKeyChecking=no ec2-user@172.31.37.89 '
                    cd /home/ec2-user/docker-build &&
                    docker build -t production-easy-wallet-k8s:1.0 . &&
                    docker stop easy-wallet || true &&
                    docker rm easy-wallet || true &&
                    docker run -d --name easy-wallet -p 9090:8080 production-easy-wallet-k8s:1.0
                    '
                    """
                }
            }
        }

    }
}
