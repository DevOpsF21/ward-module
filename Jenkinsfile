pipeline {
    agent any // Consider using agent labels for specific nodes

    environment {
        DOCKER_IMAGE = "ward-module"
        DOCKER_TAG = "v1.0"
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
            // Step 1: Stop existing Docker container
            bat label: 'Stop Existing Container', script: "docker stop ${CONTAINER_NAME} || exit 0"
            
            // Step 2: Remove existing Docker container
            bat label: 'Remove Existing Container', script: "docker rm ${CONTAINER_NAME} || exit 0"
            
            // Step 3: Run new Docker container
            bat label: 'Run New Container', script: "docker run -d --name ${CONTAINER_NAME} -p 3000:9191 ${IMAGE_FULL_NAME}"
           }
         }
        }
      
      stage('Transfer Image to Minikube') {
         steps {
           script {
            
            bat "docker save ${IMAGE_FULL_NAME}> image.tar" 
            // Load the image into Minikube's Docker environment
            bat "minikube -p minikube image load image.tar"

            //  Clean up the tar file after loading
             bat " rm image.tar "
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
            // Check the rollout status of the deployment. This command waits until the deployment is done or fails.
            bat "kubectl rollout status deployment/${DEPLOYMENT_NAME}"
            
            // After checking the rollout status, list the pods with a specific label to see their current status.
            // This can be useful for debugging if there are issues with the rollout.
            bat "kubectl get pods --selector=app=${CONTAINER_NAME}"
        }
    }
}

    }
}
