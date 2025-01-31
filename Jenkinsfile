pipeline {
    agent any
    
    environment {
        AWS_REGION = 'ap-northeast-1'
        ECR_REPO = 'my-frappe-app'
        IMAGE_TAG = "latest"
    }

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'develop', url: 'https://github.com/ravirajjagtap/CRM.git'
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                     docker.build('my-frappe-app:latest', '-f https://github.com/ravirajjagtap/CRM/blob/develop/docker/docker-compose.yml .')
                    sh 'docker build -t $my-frappe-app:$latest.'
            }
        }
        stage('Push to AWS ECR') {
            steps {
                withAWS(region: "$ap-northeast-1", credentials: 'aws-jenkins') {
                    sh 'aws ecr get-login-password --region $ap-northeast-1 | docker login --username AWS --password-stdin $my-frappe-app'
                    sh 'docker tag $my-frappe-app:$latest $my-frappe-app:$latest'
                    sh 'docker push $my-frappe-app:$latest'
                }
            }
        }
        stage('Deploy to EC2') {
            steps {
                sshagent(['ec2-key']) {
                    sh 'ssh -o StrictHostKeyChecking=no ubuntu@13.114.40.20 "docker pull $my-frappe-app:$latest && docker run -d -p 80:80 $my-frappe-app:$latest"'
                }
            }
        }
    }
}
