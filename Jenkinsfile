pipeline {
    agent any
    
    tools {
        nodejs 'node20'
    }
    
    environment {
		SCANNER_HOME= tool 'sonar-scanner'
		// NVD_API_KEY = credentials('NVD_API_KEY')
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
                sh 'trivy fs --format json -o fs-report.html .'
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
		
		// stage('Docker Container Running') {
        //     steps {
        //         script {
        //             withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
        //                 sh "docker run -d -p 3000:3000 devesh121/yelmcamp:latest ."
        //             }
        //         }
        //     }
        // }

        stage('Deploy To Kubernetes') {
            steps {
                withKubeCredentials(kubectlCredentials: [[caCertificate: '', clusterName: 'EKS-2', contextName: '', credentialsId: 'k8-token', namespace: 'webapps', serverUrl: 'https://CA8B5C8C7644A26B8C0D8CF5EC1ECDF4.gr7.us-east-1.eks.amazonaws.com']]) {
                    sh "kubectl apply -f ./Manifests/dss.yml"
                }
            }
        }

        stage('verify Deployment') {
            steps {
                withKubeCredentials(kubectlCredentials: [[caCertificate: '', clusterName: 'EKS-2', contextName: '', credentialsId: 'k8-token', namespace: 'webapps', serverUrl: 'https://CA8B5C8C7644A26B8C0D8CF5EC1ECDF4.gr7.us-east-1.eks.amazonaws.com']]) {
                    sh "kubectl get pods -n webapps"
                }
            }
        }
    }
}


