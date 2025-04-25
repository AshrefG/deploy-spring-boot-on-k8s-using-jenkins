pipeline {
    agent any
    
    tools {
        maven 'maven-3.9'
    }

    environment {
        dockerimagename = "ashrefg/project_pipeline"
        registryUrl = 'registry.hub.docker.com'  // Without https://
    }

    stages {
        stage('Build App') {
            steps {
                sh 'mvn clean package'
            }
        }
        
        stage('Build image') {
            steps {
                script {
                    sh "docker build -t ${dockerimagename} ."
                    // Tag with both latest and build number
                    sh "docker tag ${dockerimagename} ${dockerimagename}:latest"
                    sh "docker tag ${dockerimagename} ${dockerimagename}:${env.BUILD_NUMBER}"
                }
            }
        }

        stage('Pushing Image') {
            steps {
                script {
                    withCredentials([usernamePassword(
                        credentialsId: 'docker-hub-repo',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )]) {
                        sh '''
                            echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin "$registryUrl"
                        '''
                        // Push both tags
                        sh "docker push ${dockerimagename}:latest"
                        sh "docker push ${dockerimagename}:${env.BUILD_NUMBER}"
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withKubeConfig([credentialsId: 'mykubeconfig', serverUrl: 'https://192.168.49.2:8443']) {
                    // Update deployment with new image tag
                    sh "kubectl set image deployment/<your-deployment-name> *=${dockerimagename}:${env.BUILD_NUMBER}"
                    sh "kubectl rollout status deployment/<your-deployment-name>"
                }
            }
        }
    }

    post {
        always {
            // Clean up Docker credentials
            sh "docker logout ${registryUrl}"
            
            // Clean up local images to save space
            sh "docker rmi ${dockerimagename}:latest || true"
            sh "docker rmi ${dockerimagename}:${env.BUILD_NUMBER} || true"
        }
        success {
            echo "Deployment successful! Image: ${dockerimagename}:${env.BUILD_NUMBER}"
            // Add notification here (Slack, email, etc.)
        }
        failure {
            echo "Deployment failed. Check logs for details."
            // Add failure notification here
        }
    }
}
