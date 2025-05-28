// Jenkinsfile

pipeline {

    // Agent configuration:

    // This pipeline will run inside a Docker container that has Docker CLI.

    // Jenkins needs access to the host's Docker daemon.

    // Ensure your Jenkins master/agent setup allows this (e.g., by mounting /var/run/docker.sock).

    agent {

        docker {

            image 'docker:latest' // Uses an official Docker image which has Docker CLI

            args '-v /var/run/docker.sock:/var/run/docker.sock -u root' // Mount Docker socket

            // If your Jenkins user inside the container needs specific UID/GID for Docker socket access:

            // args '-v /var/run/docker.sock:/var/run/docker.sock -u root' // Or map to a user with docker group access

        }

    }
 
    // Environment variables used throughout the pipeline

    environment {

        // IMPORTANT: Replace 'yourdockerhubusername' with your actual Docker Hub username.

        // IMPORTANT: Replace 'yourimagename' with your desired image name.

        IMAGE_NAME = "yukthiprabha/myflaskapp"

        // ID of the Docker Hub credentials stored in Jenkins (see Step 1)

        DOCKERHUB_CREDENTIALS_ID = "docker-hub-credentials"

    }
 
    stages {

        stage('Checkout Code') {

            steps {

                // Get some code from a GitHub repository

                // This will checkout the repository where this Jenkinsfile is located.

                checkout scm

                echo "Code checked out successfully."

            }

        }
 
        stage('Build Docker Image') {

            steps {

                script {

                    // Use the Jenkins build number as the image tag for uniqueness

                    def imageTag = env.BUILD_NUMBER

                    // Construct the full image name with tag

                    def fullImageName = "${IMAGE_NAME}:${imageTag}"

                    // An additional tag, e.g., 'latest'

                    def latestImageName = "${IMAGE_NAME}:latest"
 
                    echo "Building Docker image: ${fullImageName}"

                    // Build the Docker image

                    // The '.' indicates that the Dockerfile is in the current directory (workspace root)

                    sh "docker build -t ${fullImageName} ."
 
                    echo "Tagging image ${fullImageName} also as ${latestImageName}"

                    sh "docker tag ${fullImageName} ${latestImageName}"
 
                    echo "Docker image built and tagged successfully."

                }

            }

        }
 
        stage('Login to Docker Hub') {

            steps {

                echo "Logging into Docker Hub..."

                // Securely access Docker Hub credentials

                // DOCKER_USER and DOCKER_PASS will be available within this block

                withCredentials([usernamePassword(credentialsId: DOCKERHUB_CREDENTIALS_ID,

                                                   usernameVariable: 'DOCKER_USER',

                                                   passwordVariable: 'DOCKER_PASS')]) {

                    // Use --password-stdin for better security than passing password on command line

                    sh 'HOME=/root echo "${DOCKER_PASS}" | docker login -u "${DOCKER_USER}" --password-stdin'

                }

                echo "Logged into Docker Hub successfully."

            }

        }
 
        stage('Push Image to Docker Hub') {

            steps {

                script {

                    def imageTag = env.BUILD_NUMBER

                    def fullImageName = "${IMAGE_NAME}:${imageTag}"

                    def latestImageName = "${IMAGE_NAME}:latest"
 
                    echo "Pushing image ${fullImageName} to Docker Hub..."

                    sh "docker push ${fullImageName}"
 
                    echo "Pushing image ${latestImageName} to Docker Hub..."

                    sh "docker push ${latestImageName}"
 
                    echo "Docker images pushed successfully."

                }

            }

        }

    }
 
    post {

        always {

            echo "Pipeline finished. Logging out from Docker Hub..."

            // Best practice to logout after operations

            // This command might fail if not logged in, so handle errors if necessary

            // or ensure it's only run if login was successful.

            // For simplicity, we'll just attempt it.

            sh "docker logout"

            echo "Cleaning up workspace..."

            cleanWs() // Cleans the Jenkins workspace

        }

        success {

            echo "Pipeline SUCCEEDED!"

        }

        failure {

            echo "Pipeline FAILED."

            // You could add notification steps here (e.g., email, Slack)

        }

    }

}
 
