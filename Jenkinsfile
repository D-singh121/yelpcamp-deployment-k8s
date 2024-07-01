pipeline {
    agent any

    tools {
        nodejs 'node20'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {
        stage('Git checkout') {
            steps {
                git  url: 'https://github.com/D-singh121/yelpcamp-deployment-k8s.git'
            }
        }

        stage('Install Package dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Unit Testing') {
            steps {
                sh 'npm test'
            }
        }

        stage('File Scan Trivy') {
            steps {
                sh 'trivy fs --format table -o fs-report.html . '
            }
        }

        stage('SonarQube scan') {
            steps {
                sh "$SCANNER_HOME/bin/sonar-scanner  -Dsonar.projectKey=Campground  -Dsonar.projectName=Campground"
            }
        }

        stage('Docker Build & Tag') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh 'docker build -t devesh121/yelmcamp:latest .'
                    }
                }
            }
        }

        stage('Trivy Image Scan') {
            steps {
                sh 'trivy image --format table -o fs-report.html devesh121/yelmcamp:latest'
            }
        }

		stage('Docker Push Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh 'docker push devesh121/yelmcamp:latest'
                    }
                }
            }
        }
		
		stage('Docker Container Run') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh 'docker run -d -p 3000:3000 devesh121/yelmcamp:latest'
                    }
                }
            }
        }
		
    }
}
