pipeline {
    agent any
    
    tools {
        nodejs 'node20'
    }
    
    environment {
		SCANNER_HOME= tool 'sonar-scanner'
		NVD_API_KEY = credentials('NVD_API_KEY')
	}

    stages {
        stage('Git checkout') {
            steps {
                git branch: 'main', url: "https://github.com/D-singh121/yelpcamp-deployment-k8s.git"
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
                sh 'trivy fs --format table -o fs-report.html .'
            }
        }
        
		//Always use OWSP 6.5.1 version for smoot work.and add NVD api-key in secret credentials.
        stage('OWASP Dependency Check') {
            steps {
                dependencyCheck additionalArguments: "--scan ./ --nvdApiKey $NVD_API_KEY", odcInstallation: 'DP-Check'
                sh 'ls -l'
            }
            post {
                always {
                    dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
                }
            }
        }
        
        stage('SonarQube scan') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh "$SCANNER_HOME/bin/sonar-scanner  -Dsonar.projectKey=Campground  -Dsonar.projectName=Campground"
                }
            }
        }
        
        stage('Docker Build & Tag') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh "docker build -t devesh121/yelmcamp:latest ."
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
                        sh "docker push devesh121/yelmcamp:latest"
                    }
                }
            }
        }
		
		stage('Docker Container Running') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh "docker run -d -p 3000:3000 devesh121/yelmcamp:latest ."
                    }
                }
            }
        }
    }
}


