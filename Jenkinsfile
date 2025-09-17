pipeline {
    agent any

    environment {
        PROJECT_ID = "your-gcp-project-id"
        CLUSTER = "gke-nikhil"
        REGION = "us-central1"
        DOCKER_REGISTRY = "gcr.io/${PROJECT_ID}"
        APP_NAME = "demo-app"
        VERSION = "${env.BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/your-repo/demo-app.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                docker build -t $DOCKER_REGISTRY/$APP_NAME:$VERSION .
                docker push $DOCKER_REGISTRY/$APP_NAME:$VERSION
                """
            }
        }

        stage('Connect to GKE') {
            steps {
                sh """
                gcloud container clusters get-credentials $CLUSTER --region $REGION --project $PROJECT_ID
                """
            }
        }

        stage('Deploy to Green') {
            steps {
                sh """
                kubectl set image deployment/app-green demo=$DOCKER_REGISTRY/$APP_NAME:$VERSION --record || \
                kubectl apply -f k8s/deployment-green.yaml
                """
            }
        }

        stage('Switch Traffic') {
            input {
                message "Do you want to switch traffic from Blue to Green?"
                ok "Yes, switch"
            }
            steps {
                sh 'kubectl patch service demo-service -p \'{"spec":{"selector":{"app":"demo","version":"green"}}}\''
            }
        }

        stage('Cleanup Old') {
            steps {
                input {
                    message "Do you want to scale down Blue?"
                    ok "Yes, scale down"
                }
                sh "kubectl scale deployment app-blue --replicas=0"
            }
        }
    }
}
