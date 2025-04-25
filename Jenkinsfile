pipeline {
    agent any
    
    tools {
        maven 'maven-3.9'
    }

    environment {
        dockerimagename = "ashrefg/project_pipeline"
        registryUrl = 'https://registry.hub.docker.com'
        registryCredential = 'docker-hub-repo'
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
                }
            }
        }

        stage('Pushing Image') {
            steps {
                script {
                    withCredentials([usernamePassword(
                        credentialsId: env.registryCredential,
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )]) {
                        sh '''docker login -u $DOCKER_USER --password-stdin $registryUrl <<< "$DOCKER_PASS"'''
                        sh "docker push ${dockerimagename}:latest"
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withKubeConfig([credentialsId: 'mykubeconfig', serverUrl: 'https://192.168.49.2:8443']) {
                    sh 'kubectl apply -f deployment-k8s.yaml'
                }
            }
        }
    }
}
