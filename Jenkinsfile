pipeline {
    agent {
        kubernetes {
            cloud 'kubernetes' // Name of your Kubernetes cloud in Jenkins
            yamlFile 'my-pod.yaml' // Path to your Pod YAML file
            containerTemplate {
                name 'hadolint' // Name of your container
                image 'hadolint/hadolint:latest-debian' // Docker image to use for the container
                command 'sh', '-c', 'echo "Hello, Kubernetes!"' // Command to run in the container
            }
        }
    }
    stages {
        stage('Run command in container') {
            steps {
                sh 'echo "Hello, Jenkins!"'
            }
        }
    }
}
