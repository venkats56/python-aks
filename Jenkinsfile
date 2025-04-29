pipeline {
    agent any

    environment {
        registryName = "testacr2568"
        registryCredential = "ACR"
        registryUrl = "testacr2568.azurecr.io"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scmGit(
                    branches: [[name: '*/main']],
                    extensions: [],
                    userRemoteConfigs: [[url: 'https://github.com/venkats56/python-aks']]
                )
            }
        }

        stage('Build Docker') {
            steps {
                script {
                    // Declare dockerImage as a local variable
                    def dockerImage = docker.build("${registryName}")
                    // Save it to the environment for next stage
                    env.DOCKER_IMAGE = "${dockerImage.imageName()}"
                }
            }
        }

        stage('Push Image') {
            steps {
                script {
                    docker.withRegistry("https://${env.registryUrl}", env.registryCredential) {
                        def dockerImage = docker.image(env.DOCKER_IMAGE)
                        dockerImage.push()
                    }
                }
            }
        }

        stage('Deploy to AKS') {
            steps {
                withKubeConfig(
                    caCertificate: '',
                    clusterName: '',
                    contextName: '',
                    credentialsId: 'K8S',
                    namespace: '',
                    restrictKubeConfigAccess: false,
                    serverUrl: ''
                ) {
                    sh "kubectl apply -f azure-aks.yaml"
                }
            }
        }
    }
}
