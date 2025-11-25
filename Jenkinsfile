pipeline {
    agent any
    
    tools {
        jdk 'jdk17'
        maven 'maven3'
    }

    environment{
        SCANNER_HOME= tool 'sonar-scanner'
        PROJECT_ID = 'applied-pager-476808-j5'
        REGION = 'us-central1'
        REPO = 'boardgame-repo'
        IMAGE = 'boardgame-app'
        TAG = 'latest'
    }
    
    stages {   
          stage('Code Checkout') {
           steps {
                git branch: 'main', changelog: false, poll: false, url: 'https://github.com/TCRDINSEH/Boardgame.git'
        }
    }
        stage('Compile') {
            steps {
            sh 'mvn compile'
            }
        }
        
        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }
        
        stage('File System scan') {
            steps {
                sh 'trivy fs --format table -o trivy-fs-report.html .'
            }
        }
        stage('Sonarqube Analysis') {
            steps {
                 withSonarQubeEnv('sonar-server') {
            sh '''
            $SCANNER_HOME/bin/sonar-scanner \
              -Dsonar.projectKey=Boardgame \
              -Dsonar.projectName=Boardgame \
              -Dsonar.sources=. \
              -Dsonar.java.binaries=. \
              -Dsonar.exclusions=target/**
            '''
            }
        }
    }  
        stage('Maven Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }
        //  stage('Publish to Nexus') {
        //     steps {
        //         sh 'mvn clean package -DskipTests'
        //     }
        // }
    stage('Docker Build and Tag') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolname: 'docker') {
                        sh "docker build -t tcrd4323/${IMAGE}:${TAG} "
                } 
            }
        }
    }
        stage('Docker Image scan') {
            steps {
                    sh "trivy image  --format table -o trivy-fs-report.html tcrd4323/webapp:latest "
            }
        }
     stage('Docker Push') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolname: 'docker') {
                        sh "docker push tcrd4323/${IMAGE}:${TAG} "
                } 
            }
        }
     }

    }
}
