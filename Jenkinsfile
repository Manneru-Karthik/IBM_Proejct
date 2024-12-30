pipeline {
    agent any
    environment {
        MONGO_URI = credentials('mongo-uri-id')
        JWT_SECRET = credentials('jwt-secret-id')
    }
    stages {
        stage('Prepare Environment') {
            steps {
                script {
                    bat '''
                    echo MONGO_URI=${MONGO_URI} > .env
                    echo JWT_SECRET=${JWT_SECRET} >> .env
                    '''
                }
            }
        }
        stage('Build Backend') {
            steps {
                script {
                    bat 'cd server && npm install'
                }
            }
        }
        stage('Build Frontend') {
            steps {
                script {
                    bat 'cd user && npm install'
                    bat 'cd user && set CI=false && npm run build'
                }
            }
        }
        stage('Test Backend') {
            steps {
                script {
                    bat 'cd server && npm test'
                }
            }
        }
        stage('Test Frontend') {
            steps {
                script {
                    bat 'cd user && npm test -- --passWithNoTests'
                }
            }
        }
        stage('Build Docker Images') {
            steps {
                script {
                    bat 'docker build -t icr.io/your-namespace/eventsphere-backend ./server'
                    bat 'docker build -t icr.io/your-namespace/eventsphere-frontend ./user'
                }
            }
        }
        stage('Publish Docker Images to IBM Cloud Registry') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'ibmcloud-credentials', usernameVariable: 'IBM_USERNAME', passwordVariable: 'IBM_PASSWORD')]) {
                        bat 'ibmcloud login -u %IBM_USERNAME% -p %IBM_PASSWORD%'
                        bat 'ibmcloud cr login'
                        bat 'docker push icr.io/your-namespace/eventsphere-backend:latest'
                        bat 'docker push icr.io/your-namespace/eventsphere-frontend:latest'
                    }
                }
            }
        }
    }
}
