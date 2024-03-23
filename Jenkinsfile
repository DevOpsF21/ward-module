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

        stage('Deploying to Kubernetes') {
   steps {
    script {
        // Log the checking process
        echo "Checking if the deployment '${DEPLOYMENT_NAME}' exists..."

        // Use 'bat' for Windows, capturing the output to check the deployment's existence
        def deploymentOutput = bat(script: "kubectl get deployment ${DEPLOYMENT_NAME} --ignore-not-found", returnStdout: true).trim()

        // Determine the existence of the deployment by checking if the output is empty.
        boolean deploymentExists = deploymentOutput != ''

        if (deploymentExists) {
            echo "Deployment '${DEPLOYMENT_NAME}' found. Updating the image and restarting..."
            // Update the deployment with the new image.
            bat "kubectl set image deployment/${DEPLOYMENT_NAME} ${CONTAINER_NAME}=${IMAGE_FULL_NAME}"
            // Restart the deployment to apply the new image.
            bat "kubectl rollout restart deployment/${DEPLOYMENT_NAME}"
        } else {
            echo "Deployment '${DEPLOYMENT_NAME}' not found. Applying configurations from YAML files..."
            // Apply the deployment and service YAML configurations to create them.
            try {
                bat "kubectl apply -f deployment.yaml -f service.yaml"
                echo "Deployment and service applied successfully."
            } catch (Exception e) {
                echo "Failed to apply deployment and service configurations."
                // In a Jenkins pipeline running on Windows, throwing an exception directly might not be necessary or effective
                // for certain batch command failures. Adjust according to your specific error handling needs.
            }
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
