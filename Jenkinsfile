pipeline {
  agent any

  parameters {
    string(name: 'Branch', defaultValue: 'main', description: '')
  }

  environment {
    AWS_REGION = "us-west-1"
    AWS_CREDENTIALS = credentials('aws-creds')
    REPO_URI = "992151247814.dkr.ecr.us-west-1.amazonaws.com/pavandas/flask-app"
    IMAGE_TAG = "latest"
    FULL_IMAGE = "${REPO_URI}:${IMAGE_TAG}"
  }

  stages {
    stage('Clone') {
      steps {
        git url: 'https://github.com/pavan931/jenkins-docker-build.git',
            branch: "${params.Branch}",
            credentialsId: 'git-creds'
      }
    }

   stage('Login to AWS ECR') {
  steps {
    withCredentials([usernamePassword(
  credentialsId: 'aws-creds',
  usernameVariable: 'AWS_ACCESS_KEY_ID',
  passwordVariable: 'AWS_SECRET_ACCESS_KEY'
)]) {
  sh '''
    apt update && apt install -y awscli
    aws configure set aws_access_key_id $AWS_ACCESS_KEY_ID
    aws configure set aws_secret_access_key $AWS_SECRET_ACCESS_KEY
    aws configure set region ap-south-1

    aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin 123456789012.dkr.ecr.ap-south-1.amazonaws.com
  '''
}

  }
}




    stage('Build Image') {
      steps {
        sh "docker build -t ${FULL_IMAGE} ."
      }
    }

    stage('Push Image to ECR') {
      steps {
        sh "docker push ${FULL_IMAGE}"
      }
    }

    stage('Deploy Container') {
      steps {
        sh '''
          docker stop my-app || true
          docker rm my-app || true
          docker run -d --name my-app -p 8001:80 $FULL_IMAGE
        '''
      }
    }
  }
}

