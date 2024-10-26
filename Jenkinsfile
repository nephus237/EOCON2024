pipeline {
    agent {
        label "deployment"
    }
    tools {
        maven 'mavenapp'
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
        stage('Snyk Security Scan') {
            steps {
                withCredentials([string(credentialsId: 'snyk-api-token', variable: 'SNYK_TOKEN')]) {
                    script {
                        // Install Snyk if it's not already installed
                        sh '''
                            if ! command -v snyk &> /dev/null
                            then
                                npm install -g snyk
                            fi
                        '''
                        // Authenticate Snyk using the API token
                        sh 'snyk auth $SNYK_TOKEN'
                        
                        // Run the Snyk code vulnerability test
                        echo "Running Snyk Code Vulnerability Scan..."
                        sh 'snyk test --all-projects'

                        // Run the Snyk Dependency Check
                        echo "Running Snyk Dependency Check..."
                        sh 'snyk test'

                        // Optionally, monitor the project for ongoing vulnerability tracking
                        echo "Monitoring the project with Snyk..."
                        sh 'snyk monitor'
                    }
                }
            }
        }
        stage('Docker Build') {
            steps {
                sh 'docker build --no-cache -t feramin108/mavenapp .'
            }
        }
        stage('Docker Scan') {
            steps {
                withCredentials([string(credentialsId: 'snyk-api-token', variable: 'SNYK_TOKEN')]) {
                    script {
                        // Authenticate Snyk for Docker image scanning
                        sh 'snyk auth $SNYK_TOKEN'
                        
                        // Scan the built Docker image for vulnerabilities
                        echo "Scanning Docker image for vulnerabilities..."
                        sh 'snyk container test feramin108/mavenapp'
                    }
                }
            }
        }
        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    script {
                        def dockerUsername = env.DOCKER_USERNAME
                        def dockerPassword = env.DOCKER_PASSWORD

                        try {
                            sh "docker login -u $dockerUsername -p $dockerPassword"
                            sh "docker push feramin108/mavenapp"
                        } finally {
                            sh "docker logout"
                        }
                    }
                }
            }
        }
        stage('Deploy') {
            steps {
                sh 'docker pull feramin108/mavenapp'
                sh 'docker run -d -p 5050:8080 feramin108/mavenapp'
            }
        }
    }
}
