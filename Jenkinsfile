pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'mostafataha12/frappe'  // Docker image name
        DOCKER_TAG = 'test'  // Docker image tag (can be commit hash or version)
        ANSIBLE_INVENTORY = 'ansible/inventory'  // Path to Ansible inventory file
        
    }

    stages {
        stage('Build Docker Image') {
            steps {
                script {
                    echo "Building Docker image..."
                    // Build the Docker image with a specific tag
                    sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ."
                }
            }
        }

        stage('Push Images to Docker Hub') {
            steps {
                script {
                    echo "Pushing Docker image to Docker Hub..."
                    withCredentials([usernamePassword(credentialsId: 'docker-hub', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                        sh "echo $PASS | docker login -u $USER --password-stdin"
                        sh "docker push ${DOCKER_IMAGE}:${DOCKER_TAG}"
                    }
                }
            }
        }

        stage('Approval') {
            steps {
                script {
                    // Pause pipeline and wait for user approval before deploying
                    input message: 'Do you want to proceed with deployment?', ok: 'Deploy'
                }
            }
        }

        stage('Update Docker Compose') {
            steps {
                script {
                    echo "Updating docker-compose.yml with the new image tag..."

                    // Replace the image tag in docker-compose.yml with the new tag
                    sh """
                        sed -i 's|image: ${DOCKER_IMAGE}:.*|image: ${DOCKER_IMAGE}:${DOCKER_TAG}|g' docker-compose.yml
                    """
                }
            }
        }

        stage('Deploy with Ansible') {
            steps {
                script {
                    echo "Running Ansible to deploy updated docker-compose.yml..."

            steps {
                sshagent(['ansible-ssh-key']) {
                    sh """
                        cd ansible
                        ansible-playbook -i ${ANSIBLE_INVENTORY} playbook.yml
                    """
                }
            }
        }
    }

    post {
        always {
            // Cleanup actions or post-deployment steps (e.g., notifications, logs)
            echo "Pipeline execution completed."
        }
    }
}

