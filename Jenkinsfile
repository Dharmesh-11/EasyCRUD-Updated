pipeline {
agent {
label 'agnet'
}

environment {
    AWS_REGION = 'eu-north-1'
    EKS_CLUSTER = 'my-eks'
    BACKEND_IMAGE = 'dharmesh111/backend:app1'
    FRONTEND_IMAGE = 'dharmesh111/frontend:app2'
}

stages {

    stage('Pull Code') {
        steps {
            git branch: 'main',
                url: 'https://github.com/Dharmesh-11/EasyCRUD-Updated.git'
        }
    }

    stage('Build Backend Image') {
        steps {
            dir('backend') {
                sh '''
                docker build -t ${BACKEND_IMAGE} .
                '''
            }
        }
    }

    stage('Build Frontend Image') {
        steps {
            dir('frontend') {
                sh '''
                docker build -t ${FRONTEND_IMAGE} .
                '''
            }
        }
    }

    stage('Docker Login') {
        steps {
            withCredentials([
                usernamePassword(
                    credentialsId: 'Docker-11',
                    usernameVariable: 'USERNAME',
                    passwordVariable: 'PASSWORD'
                )
            ]) {
                sh '''
                echo "$PASSWORD" | docker login -u "$USERNAME" --password-stdin
                '''
            }
        }
    }

    stage('Push Images') {
        steps {
            sh '''
            docker push ${BACKEND_IMAGE}
            docker push ${FRONTEND_IMAGE}
            '''
        }
    }

    stage('Configure EKS') {
        steps {
            withCredentials([
                [$class: 'AmazonWebServicesCredentialsBinding',
                 credentialsId: 'AWS-Keys']
            ]) {
                sh '''
                echo "===== AWS Identity ====="
                aws sts get-caller-identity

                echo "===== Configure Kubeconfig ====="
                aws eks update-kubeconfig \
                  --region ${AWS_REGION} \
                  --name ${EKS_CLUSTER}

                echo "===== Current Context ====="
                kubectl config current-context

                echo "===== Cluster Nodes ====="
                kubectl get nodes
                '''
            }
        }
    }

    stage('Deploy to EKS') {
        steps {
            withCredentials([
                [$class: 'AmazonWebServicesCredentialsBinding',
                 credentialsId: 'AWS-Keys']
            ]) {
                sh '''
                kubectl apply -f k8s/backend-deployment.yaml
                kubectl apply -f k8s/backend-service.yaml

                kubectl apply -f k8s/frontend-deployment.yaml
                kubectl apply -f k8s/frontend-service.yaml

                kubectl rollout status deployment/backend --timeout=300s
                kubectl rollout status deployment/frontend --timeout=300s
                '''
            }
        }
    }

    stage('Verify Deployment') {
        steps {
            withCredentials([
                [$class: 'AmazonWebServicesCredentialsBinding',
                 credentialsId: 'AWS-Keys']
            ]) {
                sh '''
                echo "===== Deployments ====="
                kubectl get deployments

                echo "===== Pods ====="
                kubectl get pods -o wide

                echo "===== Services ====="
                kubectl get svc

                echo "===== Namespace Events ====="
                kubectl get events --sort-by=.metadata.creationTimestamp | tail -20
                '''
            }
        }
    }
}

post {

    success {
        echo 'Application Successfully Deployed to Amazon EKS'
    }

    failure {
        echo 'Pipeline Failed'
    }

    always {
        cleanWs()
    }
}


}
