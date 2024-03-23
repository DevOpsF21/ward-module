pipeline {
    agent any // Consider using agent labels for specific nodes

    environment {
        DOCKER_IMAGE = "ward-module"
        DOCKER_TAG = "v1.0.${BUILD_NUMBER}"
        IMAGE_FULL_NAME = "${DOCKER_IMAGE}:${DOCKER_TAG}"
        DEPLOYMENT_NAME = "ward-module-deployment"
        CONTAINER_NAME = "ward-module"
        POSTMAN_COLLECTION = "ward_collection.postman_collection.json"
    }

    stages {
        stage('Preparation') {
            steps {
                echo 'Preparing the environment...'
            }
        }

        stage('Building Docker Image') {
            steps {
                bat "docker build -t ${IMAGE_FULL_NAME} ."
            }
        }
      
        stage('Run Docker Container Locally') {
            steps {
                script {
                    // bat "docker stop ${CONTAINER_NAME} || exit 0"
                    // bat "docker rm ${CONTAINER_NAME} || exit 0"
                    bat "docker run -d --name ${CONTAINER_NAME} -p 3000:9191 ${IMAGE_FULL_NAME}"
                }
            }
        }

 
        stage('Deploying to Minikube') {
            steps {
                script {
                    // Ensure kubectl is using Minikube's Docker environment
                    bat 'minikube -p minikube docker-env'
                    
                    // Check if the deployment exists
                    def deploymentExists = bat(script: "kubectl get deployment ${DEPLOYMENT_NAME}", returnStatus: true) == 0

                    if (deploymentExists) {
                        // Update the deployment to use the new Docker image
                        bat "kubectl set image deployment/${DEPLOYMENT_NAME} ${CONTAINER_NAME}=${IMAGE_FULL_NAME}"
                        
                        // Restart the pods
                        bat "kubectl rollout restart deployment/${DEPLOYMENT_NAME}"
                    } else {
                        // Apply the deployment and service YAML files
                        bat "kubectl apply -f deployment.yaml -f service.yaml"
                    }
                }
            }
        }


        stage('Postman Testing') {
            steps {
                script {
                    try {
                        bat "newman run ${POSTMAN_COLLECTION}"
                    } catch (Exception e) {
                        echo "Postman tests failed but build continues..."
                    }
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                script {
                    bat "kubectl rollout status deployment/${DEPLOYMENT_NAME}"
                    bat "kubectl get pods --selector=app=${CONTAINER_NAME}"
                }
            }
        }
    }
}
