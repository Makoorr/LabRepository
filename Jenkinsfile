pipeline {
    agent any

    environment {
        DOCKERHUB_CREDS = 'dockerhub-creds'
        DOCKERHUB_REPO = 'makoorr/lab-repository'
        DOCKER_IMAGE_TAG = 'latest'
        ANSIBLE_PLAYBOOK = './ansible/deploy-to-k8s.yml'
        ANSIBLE_INVENTORY = './ansible/inventory.yml'
        VAR_FILE = './ansible/vars.yml'
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
                    docker.build("${DOCKERHUB_REPO}")
                }
            }
        }
        stage('Push to DockerHub') {
            steps {
                echo 'Pushing Docker image to DockerHub...'
                script {
                    docker.withRegistry('https://index.docker.io/v1/', "${DOCKERHUB_CREDS}") {
                        docker.image("${DOCKERHUB_REPO}").push('latest')
                    }
                }
            }
        }


        stage('Provision Terraform') {
            steps {
                echo 'Provisioning infrastructure using Terraform...'
                script {
                    sh 'cd infrastructure && terraform init'
                    sh 'cd infrastructure && terraform apply -auto-approve'
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo 'Deploying updated image to Kubernetes using Ansible...'
                script {
                    ansiblePlaybook(
                        playbook: "${ANSIBLE_PLAYBOOK}",
                        inventory: "${ANSIBLE_INVENTORY}",
                        extras: '--extra-vars "@${VAR_FILE}"',
                        colorized: true
                    )
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
