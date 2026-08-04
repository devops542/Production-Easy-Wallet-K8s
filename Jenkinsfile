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
                    -Dsonar.projectKey=Production-Easy-Wallet
                    '''
                }
            }
        }

        stage('Upload to Nexus') {
            steps {
                sh 'mvn deploy -DskipTests'
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                deploy adapters: [
                    tomcat9(
                        credentialsId: 'tomcat',
                        path: '',
                        url: 'http://3.108.234.45:8080'
                    )
                ],
                contextPath: 'easy-wallet',
                war: '**/*.war'
            }
        }
    }
}
