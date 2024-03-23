pipeline {
    agent any

    environment {
        // Define the Docker image you want to transfer and use, dynamically setting the version tag with each build
        DOCKER_IMAGE = "ward-module"
        DOCKER_TAG = "v1.0.${BUILD_NUMBER}" // Dynamically includes Jenkins build number
        IMAGE_FULL_NAME = "${DOCKER_IMAGE}:${DOCKER_TAG}"
        // Use the deployment name from your Kubernetes deployment manifest
        DEPLOYMENT_NAME = "ward-module-deployment"
        // Use the container name from your Kubernetes deployment manifest
        CONTAINER_NAME = "ward-module"
        // Path to Postman collection file in your Git repository
        POSTMAN_COLLECTION = "ward_collection.postman_collection.json"
    }

    stages {
        stage('Preparation') {
            steps {
                echo 'Preparing the environment...'
                // Windows-specific preparation steps can go here
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
                    // Stop and remove the existing container if running
                    bat "docker stop ${CONTAINER_NAME} || exit 0"
                    bat "docker rm ${CONTAINER_NAME} || exit 0"
                    // Run the new container with the updated image on port 3000
                    bat "docker run -d --name ${CONTAINER_NAME} -p 3000:3000 ${IMAGE_FULL_NAME}"
                }
            }
        }

        stage('Deploying to Kubernetes') {
            steps {
                script {
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
                // Run Postman tests
                script {
                    try {
                        // Assuming Newman is installed globally and accessible from the command line
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
                    // Check the rollout status to ensure it's successful
                    bat "kubectl rollout status deployment/${DEPLOYMENT_NAME}"
                    // Optionally, list the running pods to verify the update
                    bat "kubectl get pods --selector=app=${CONTAINER_NAME}"
                }
            }
        }
    }
}
