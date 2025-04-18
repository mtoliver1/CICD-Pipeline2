pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'mtolv/java-http-server:latest'
        SONARQUBE_SERVER = 'http://sonarqube:9001'
    }

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/mtoliver1/CICD-Pipeline2.git'
            }
        }

        stage('Build (Java 17)') {
            steps {
                script {
                    docker.image('maven:3.9.6-eclipse-temurin-17').inside('--network cicd-network2') {
                        sh 'mvn clean package'
                    }
                }
            }
        }

        stage('Test (Java 11)') {
            steps {
                script {
                    docker.image('maven:3.9.6-eclipse-temurin-11').inside('--network cicd-network2') {
                        sh 'mvn test'
                    }
                }
            }
        }

        stage('Static Code Analysis (Java 21)') {
            steps {
                script {
                    withDockerContainer(image: 'maven:3.9.6-eclipse-temurin-21', args: '--network cicd-network2') {
                        withSonarQubeEnv('SonarQube') {
                            withCredentials([string(credentialsId: 'SonarQube-Cred', variable: 'SONAR_TOKEN')]) {
                                sh 'mvn sonar:sonar -Dsonar.login=$SONAR_TOKEN'
                            }
                        }
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE .'
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-hub',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh 'echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin'
                    sh 'docker push $DOCKER_IMAGE'
                }
            }
        }

    }
}