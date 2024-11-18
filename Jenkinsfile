pipeline {
    agent any

    environment {
        DOCKERHUB_CREDS = 'dockerhub-creds' // Jenkins credentials ID for DockerHub
        DOCKERHUB_REPO = 'basherr/lab-repository'
        DOCKER_IMAGE_TAG = 'latest'
        ANSIBLE_PLAYBOOK = './Ansible/deploy-to-k8s.yml' // Path to the Ansible playbook
        ANSIBLE_INVENTORY = './Ansible/inventory.yml' // Path to the inventory file
        VAR_FILE = './Ansible/vars.yml' // Path to the variable file
    }

    stages {
        stage('Clone Repository') {
            steps {
                script {
                    checkout scm
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image...'
                script {
                    sh "docker build -t ${DOCKERHUB_REPO}:${DOCKER_IMAGE_TAG} ."
                }
            }
        }

        stage('Push to DockerHub') {
            steps {
                echo 'Pushing Docker image to DockerHub...'
                script {
                    sh """
                    docker login -u ${DOCKERHUB_USERNAME} -p ${DOCKERHUB_PASSWORD}
                    docker push ${DOCKERHUB_REPO}:${DOCKER_IMAGE_TAG}
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo 'Deploying updated image to Kubernetes using Ansible...'
                script {
                    sh """
                    ansible-playbook -i ${ANSIBLE_INVENTORY} ${ANSIBLE_PLAYBOOK} --extra-vars "@${VAR_FILE}"
                    """
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
