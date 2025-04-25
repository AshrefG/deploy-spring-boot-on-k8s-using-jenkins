pipeline {
    agent any
    
    tools {
        maven 'maven-3.9'
        // Add Docker tool configuration
        docker 'docker'
    }

    environment {
        dockerimagename = "ashrefg/project_pipeline"
        // Don't need to declare dockerImage here as we'll use it in script blocks
        registryUrl = 'https://registry.hub.docker.com'
        registryCredential = 'docker-hub-repo'
    }

    stages {
        stage('Build App') {
            steps {
                sh 'mvn clean package'
            }
        }
        
        stage('Build Image') {
            steps {
                script {
                    // Explicitly build from Dockerfile in current directory
                    dockerImage = docker.build("${dockerimagename}", ".")
                }
            }
        }

        stage('Pushing Image') {
            steps {
                script {
                    docker.withRegistry(env.registryUrl, env.registryCredential) {
                        dockerImage.push("latest")
                        // Optionally push with build number tag
                        dockerImage.push("${env.BUILD_NUMBER}")
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                // Ensure kubectl is available in your Jenkins agent
                withKubeConfig([credentialsId: 'mykubeconfig', serverUrl: 'https://192.168.49.2:8443']) {
                    sh 'kubectl apply -f deployment-k8s.yaml'
                    // Add rollout status check
                    sh 'kubectl rollout status deployment/<your-deployment-name>'
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment successful!'
            // Optional: Add notification steps
        }
        failure {
            echo 'Deployment failed.'
            // Optional: Add failure notification steps
        }
    }
}
