pipeline {
    agent any
    environment {
        AWS_ACCOUNT_ID="864899865567"
        AWS_DEFAULT_REGION="us-east-1"
        IMAGE_REPO_NAME="aisdlc"
        IMAGE_TAG="latest"
        REPOSITORY_URI = "864899865567.dkr.ecr.us-east-1.amazonaws.com/aisdlc"
        EKS_CLUSTER_NAME = "sdlc-eks-cluster"
        AWS_CREDENTIALS_ID = "awscred"
        SONAR_URL = 'http://54.89.31.89:9000/'
        SONAR_USER = "admin"
        SONAR_PASS = "sonar@123"
        SONAR_AUTH_TOKEN = "sonar-devops"
    }
   
    stages {
        stage('SonarQube Analysis') {
            steps {
                withCredentials([string(credentialsId: 'sonarcred', variable: 'SONAR_AUTH_TOKEN')]) {
                    sh '''
                        sonar-scanner \
                        -Dsonar.projectKey=sonar-devops \
                        -Dsonar.host.url=$SONAR_URL \
                        -Dsonar.login=$SONAR_AUTH_TOKEN
                    '''
                }
            }
        }

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
   stage('SonarQube Scan') {
            steps {
                script {
                    // Run SonarQube scanner to analyze the code
                    withSonarQubeEnv(SONARQUBE_SERVER) {
                        sh 'sonar-scanner -Dsonar.login=$SONARQUBE_TOKEN'
                    }
                }
                
            }
        }
        
    // Building Docker images
    stage('Building image') {
      steps{
        script {
          dockerImage = docker.build "${IMAGE_REPO_NAME}:${IMAGE_TAG}"
        }
      }
    }
   
    // Uploading Docker images into AWS ECR
    stage('Pushing to ECR') {
     steps{  
         script {
                sh """docker tag ${IMAGE_REPO_NAME}:${IMAGE_TAG} ${REPOSITORY_URI}:$IMAGE_TAG"""
                sh """docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}:${IMAGE_TAG}"""
         }
        }
      }
    stage('Quality Gate') {
            steps {
                script {
                    // Wait for SonarQube Quality Gate to pass
                    waitForQualityGate abortPipeline: true
                }
            }
        }
     stage('Install kubectl') {
            steps {
                script {
                    sh """
                    curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.32.0/2024-12-20/bin/linux/amd64/kubectl
                    chmod +x kubectl
                    mv kubectl /usr/local/bin/
                    """
                }
            }
        }
    stage('Verify kubectl') {
            steps {
                sh 'kubectl version --client'
            }
        }
     /*// Setting up EKS cluster configuration
     stage('Set up Kubeconfig for EKS') {
            steps {
                script {
                    withAWS(credentials: AWS_CREDENTIALS_ID, region: AWS_REGION) {
                        sh "aws eks update-kubeconfig --region ${AWS_REGION} --name ${EKS_CLUSTER_NAME}"
                        sh "kubectl get ns"
                    }
                }
            }
        }*/

       stage('Wait Before Test') {
            steps {
                echo 'Waiting for 10 seconds before deployment...'
                sleep time: 10, unit: 'SECONDS'  // Sleep for 10 seconds
            }
        } 
    /*//Deploy manifest files to EKS cluster
      stage('Deploy to EKS') {
            steps {
                script {
                    sh 'kubectl apply -f "sample-k8s-deployment.yaml"'
                    sh 'kubectl apply -f "sample-k8s-service.yaml"'
                }
            } 
      }*/
      stage('Deploying fastapi App to Kubernetes') {
          
            steps {
                script {
                    
                    sh ('aws eks update-kubeconfig --name sdlc-eks-cluster --region us-east-1')
                    sh "kubectl get ns"
                    sh "kubectl apply -f sample-k8s-deployment.yaml"
                    sh "kubectl apply -f sample-k8s-service.yaml"
                }
             }
        }
    }
}
