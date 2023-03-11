pipeline {
    agent {
        kubernetes {
            cloud 'kubernetes' // Name of your Kubernetes cloud in Jenkins
            yamlFile 'my-pod.yaml' // Path to your Pod YAML file
            label 'my-label' // Label to use for the agent pod
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
