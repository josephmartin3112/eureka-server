pipeline {
    agent any

    tools {
        maven 'Maven-3.9.8'
        jdk 'JDK-21'
    }

    stages {
        stage('Checkout') {
            steps {
                script {
                    catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                        git url: 'https://github.com/josephmartin3112/eureka-server', branch: 'main'
                    }
                }
            }
        }

        stage('Pre-Build') {
            steps {
                script {
                    catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                        bat '''
                        docker stop eureka-sr || true
                        docker rm eureka-sr || true
                        docker rmi -f eureka-sr:latest || true
                        '''
                    }
                }
            }
        }

        stage('Build') {
            steps {
                script {
                    catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                        bat 'mvn clean install'
                    }
                }
            }
        }

        stage('Create Docker Network') {
            steps {
                script {
                    // Create a Docker network, ignore errors if it already exists
                    catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                        bat 'docker network create my-network || echo "Network already exists"'
                    }
                }
            }
        }

        stage('Post-Build') {
            steps {
                script {
                    catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                        bat '''
                        docker build -t eureka-sr:latest .
                        docker run -d --network my-network -p 8761:8761 --name eureka-sr eureka-sr:latest
                        '''
                    }
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline execution completed!'
        }
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline had errors, but execution continued.'
        }
    }
}