pipeline {
    agent {
        label "deployment"
    }
    tools {
        maven 'maven_lab3'
    }
    stages {
        stage('Fetch code') {
            steps {
                // Checkout the Git repository
                checkout scm
            }
        }
        stage('Build') {
            steps {
                sh 'mvn install -DskipTests'
            } 

            post {
                success {
                    echo 'Now Archiving it...'
                    archiveArtifacts artifacts: '**/target/*.war'
                }
            }
        }
        stage('Build & Unit Test') {
            steps {
                sh 'mvn test'
            }
        }
        stage('Integration Test') {
            steps {
                sh 'mvn verify'
            }
        }
       
        
        stage('Docker Build') {
            steps {
                sh 'docker build --no-cache -t feramin108/maven_lab3 .'
            }
        }
        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'feramin108', passwordVariable: 'mavebuild123')]) {
                    sh '''
                        docker login -u $feramin108 -p $mavebuild123
                        docker push feramin108/maven_lab3
                    '''
                }
            }
        }
        stage('Deploy') {
            steps {
                sh 'docker pull feramin108/maven_lab3'
                sh 'docker-compose -f docker-compose.yaml up -d'
            }
        }
    }
}
