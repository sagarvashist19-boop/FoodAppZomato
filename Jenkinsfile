pipeline {

agent any

environment {
    AWS_ACCOUNT_ID = "070203626273"
    AWS_REGION = "us-east-1"
    IMAGE_NAME = "foodapp"
    ECR_REPO = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${IMAGE_NAME}"
}

stages {

    stage('Clone Repo') {
        steps {
            checkout scm
        }
    }

    stage('Build Docker Image') {
        steps {
            sh "docker build -t ${IMAGE_NAME}:latest ."
        }
    }

    stage('Login to ECR') {
        steps {
            sh """
            aws ecr get-login-password --region ${AWS_REGION} \
            | docker login --username AWS \
            --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
            """
        }
    }

    stage('Tag Image') {
        steps {
            sh "docker tag ${IMAGE_NAME}:latest ${ECR_REPO}:latest"
        }
    }

    stage('Push Image') {
        steps {
            sh "docker push ${ECR_REPO}:latest"
        }
    }

    stage('Configure EKS Access') {
        steps {
            sh "aws eks update-kubeconfig --region ${AWS_REGION} --name devops-cluster"
        }
    }

    stage('Deploy to EKS') {
        steps {
            sh "kubectl apply -f deployment.yml"
            sh "kubectl apply -f service.yml"
        }
    }
}

}
