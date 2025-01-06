pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'mostafataha12/frappe'  // Docker image name
        DOCKER_TAG = 'modern-v9'  // Docker image tag (can be commit hash or version)
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

        stage('Update Docker Compose') {
            steps {
                script {
                     echo "Updating docker-compose.yml with the new image tag..."
                    // Replace the image tag in docker-compose.yml with the new tag
                    sh """
                        sed -i 's|image: ${DOCKER_IMAGE}:.*|image: ${DOCKER_IMAGE}:${DOCKER_TAG}|g' ${WORKSPACE}/docker-compose.yml
                        cat ${WORKSPACE}/docker-compose.yml
                    """
                    
                }
            }
        }

        stage('Deploy with Ansible') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'ansible-ssh-key', usernameVariable: 'SSH_USER', passwordVariable: 'SSH_PASSWORD')]) {
                    sh """
                        echo "Using inventory file:"
                        cat ${ANSIBLE_INVENTORY}
                        export ANSIBLE_HOST_KEY_CHECKING=False
                        export ANSIBLE_PASSWORD="${SSH_PASSWORD}"
                        ansible-playbook -i ansible/inventory ansible/playbook.yml --user=${SSH_USER} --ask-pass --extra-vars "ansible_password=${SSH_PASSWORD} ansible_become_pass=${SSH_PASSWORD}" || { echo "Ansible playbook failed"; exit 1; }

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
