pipeline {
    agent any
    environment {
        AWS_ACCOUNT_ID="864899865567"
        AWS_DEFAULT_REGION="us-east-1"
        IMAGE_REPO_NAME="aisdlc"
        IMAGE_TAG="latest"
        REPOSITORY_URI = "864899865567.dkr.ecr.us-east-1.amazonaws.com/aisdlc"
	IMAGE_NAME = "aisdlc"
    }
   
    stages {
        
         stage('Logging into AWS ECR') {
            steps {
                script {
                sh """aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 864899865567.dkr.ecr.us-east-1.amazonaws.com """
                }
                 
            }
        }
        
        stage('Cloning Git') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/main']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: '', url: 'https://github.com/XI-4568radhakrishna/Fast-api-2.git']]])     
            }
        }
  
    // Building Docker images
    stage('Building image') {
      steps{
        script {
        sh """docker build -t aisdlc ."""
        }
      }
    }
   
     	    
    // Uploading Docker images into AWS ECR
    stage('Pushing to ECR') {
     steps{  
         script {
                sh """ docker tag aisdlc:latest 864899865567.dkr.ecr.us-east-1.amazonaws.com/aisdlc:latest"""
                sh """docker push 864899865567.dkr.ecr.us-east-1.amazonaws.com/aisdlc:latest"""
         }
        }
      }
	stage('Install kubectl') {
            steps {
                script {
                    sh '''
                    curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
                    chmod +x kubectl
		    sudo mv kubectl /usr/local/bin/
                    kubectl version --client
                    '''
                }
            }
        }
	stage('Set up Kubeconfig for EKS') {
            steps {
                script {
                    
                    sh "aws eks update-kubeconfig --region us-east-1 --name sdlc-eks-cluster"

                    
                }
            }
       }
       stage('Wait Before Test') {
            steps {
                echo 'Waiting for 10 seconds before deployment...'
                sleep time: 10, unit: 'SECONDS'  // Sleep for 10 seconds
            }
        } 
    //Deploy manifest files to EKS cluster
      stage('Deploy to EKS') {
            steps {
                script {
                    sh 'kubectl apply -f "sample-k8s-deployment.yaml"'
                    sh 'kubectl apply -f "sample-k8s-service.yaml"'
                }
            }
        }
    }
    
}
