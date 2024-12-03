pipeline {
    agent any

    environment {
        PROJECT_ID = 'cbt-project-442207'
        CLUSTER_NAME = 'my-gke-cluster'
        CLUSTER_REGION = 'asia-northeast3'
        REPOSITORY = 'test-repository'
        IMAGE_NAME = 'test-nginx'
        TAG = '3'
        ARTIFACT_REGISTRY = "${CLUSTER_REGION}-docker.pkg.dev/${PROJECT_ID}/${REPOSITORY}/${IMAGE_NAME}:${TAG}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${IMAGE_NAME}:${TAG}")
                }
            }
        }

        stage('Push to Artifact Registry') {
            steps {
                script {
                    docker.withRegistry("https://${CLUSTER_REGION}-docker.pkg.dev", 'artifact-registry-credentials') {
                        docker.image("${IMAGE_NAME}:${TAG}").push()
                    }
                }
            }
        }

        stage('Deploy to GKE') {
            steps {
                script {
                    withCredentials([file(credentialsId: 'jenkins-gke-key', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                        sh """
                            gcloud auth activate-service-account --key-file=${GOOGLE_APPLICATION_CREDENTIALS}
                            gcloud config set project ${PROJECT_ID}
                            gcloud container clusters get-credentials ${CLUSTER_NAME} --region ${CLUSTER_REGION}
                            kubectl set image deployment/test test=${ARTIFACT_REGISTRY}
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment successful!'
        }
        failure {
            echo 'Deployment failed.'
        }
    }
}
