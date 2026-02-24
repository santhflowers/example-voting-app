pipeline {
    agent any

    environment {
        DOCKER_HUB_USER = 'santhflowers'
        IMAGE_NAME = 'ecommerce-prod'
        REGISTRY_CREDENTIALS = 'dockerhubcre' // ID from Jenkins Credentials
        K8S_CREDENTIALS = 'k8s-config' // ID of the kubeconfig file you added
    }

    stages {
        stage('Clone Code') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("${DOCKER_HUB_USER}/${IMAGE_NAME}:${BUILD_NUMBER}")
                }
            }
        }

        stage('Push to DockerHub') {
            steps {
                script {
                    docker.withRegistry('', REGISTRY_CREDENTIALS) {
                        dockerImage.push()
                        dockerImage.push("latest")
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                // Use the kubeconfig file to talk to the cluster
                configFileProvider([configFile(fileId: K8S_CREDENTIALS, variable: 'KUBECONFIG')]) {
                    sh "sed -i 's|\${DOCKER_IMAGE}|${DOCKER_HUB_USER}/${IMAGE_NAME}:${BUILD_NUMBER}|g' deployment.yaml"
                    sh "kubectl apply -f deployment.yaml --kubeconfig=${KUBECONFIG}"
                }
            }
        }
    }
}
