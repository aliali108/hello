pipeline {
  agent any

  environment {
    DOCKERHUB_REPO = 'liloakram/hello'
    DOCKERHUB_CRED = 'dockerhub-creds'
    EC2_HOST       = 'ec2-18-221-201-132.us-east-2.compute.amazonaws.com'
    EC2_USER       = 'liloakram'
    EC2_CREDS      = 'ec2-creds'
    APP_PORT       = '8080'
    PATH           = "/usr/local/bin:/opt/homebrew/bin:/Applications/Docker.app/Contents/Resources/bin:${env.PATH}"
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Docker Build') {
      steps {
        script {
          env.IMAGE_TAG = sh(script: 'date +%Y%m%d-%H%M', returnStdout: true).trim()
        }
        sh 'docker version'
        sh 'docker build --platform linux/amd64 -t ${DOCKERHUB_REPO}:${IMAGE_TAG} .'
      }
    }

    stage('Push Docker image') {
      steps {
        withCredentials([usernamePassword(credentialsId: env.DOCKERHUB_CRED,
                                          usernameVariable: 'DH_USER',
                                          passwordVariable: 'DH_PASS')]) {
          sh '''
            echo "$DH_PASS" | docker login -u "$DH_USER" --password-stdin
            docker push ${DOCKERHUB_REPO}:${IMAGE_TAG}
          '''
        }
      }
    }

    stage('Deploy on EC2 (pull & run)') {
      steps {
        withCredentials([sshUserPrivateKey(credentialsId: env.EC2_CREDS,
                                           keyFileVariable: 'EC2_KEY')]) {
          sh '''
            ssh -T -o BatchMode=yes -o ConnectTimeout=15 -o StrictHostKeyChecking=no -o IdentitiesOnly=yes -i "$EC2_KEY" ${EC2_USER}@${EC2_HOST} "docker pull ${DOCKERHUB_REPO}:${IMAGE_TAG} && (docker rm -f hello || true) && docker run -d --name hello -p 9090:${APP_PORT} ${DOCKERHUB_REPO}:${IMAGE_TAG}"
          '''
        }
      }
    }
  }

  post {
    success {
      echo 'Pipeline finished.'
      echo "Deployed: http://${EC2_HOST}:9090/"
    }
    failure {
      echo 'Pipeline failed. Check the console output above.'
    }
  }
}
