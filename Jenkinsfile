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
       stage('Static Code Analysis') {
        steps {
        script {
            try {
                sh 'mvn checkStyle:checkStyle pmd:pmd findbugs:findbugs'
                recordIssues(qualityGates: [[threshold: 800, type: 'TOTAL', unstable: false]],
                    tools: [checkStyle(pattern: 'target/checkstyle-result.xml'),
                           pmdParser(pattern: 'target/pmd.xml'),
                           findBugs(pattern: 'target/findbugsXml.xml')]
                )
            } catch (Exception e) {
                currentBuild.result = 'FAILURE'
                throw e
            }
        }
    }
}
        
        stage('Docker Build') {
            steps {
                sh 'docker build --no-cache -t feramin108/maven_lab3 .'
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
                    sh "docker push feramin108/maven_lab3"
                } finally {
                    sh "docker logout"
                }
            }
        }
    }
}

        stage('Deploy') {
            steps {
                sh 'docker pull feramin108/maven_lab3'
                sh 'docker run -d -p 5050:8080 feramin108/maven_lab3'
            }
        }
    }
}
