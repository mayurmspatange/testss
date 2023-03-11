pipeline {
    agent {
        kubernetes {
            cloud 'kubernetes' // Name of your Kubernetes cloud in Jenkins
            yamlFile 'my-pod.yaml' // Path to your Pod YAML file
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
