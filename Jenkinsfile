
pipeline {
  agent any

  environment {
    DOCKER_BFLASK_IMAGE = "ashwinapple/ci-cd-jenkins:latest"
    DEPLOY_HOST = "192.168.101.62"
    DEPLOY_USER = "comtel"
    DEPLOY_CONTAINER = "flask-app"
  }

  stages {

    stage('Build') {
      steps {
        sh 'docker build -t my-flask-app:latest .'
        sh 'docker tag my-flask-app:latest $DOCKER_BFLASK_IMAGE'
      }
    }

    stage('Test') {
      steps {
        sh 'docker run --rm my-flask-app:latest python -m pytest app/tests/'
      }
    }

    stage('Push Image') {
      steps {
        withCredentials([
          usernamePassword(
            credentialsId: 'a11330ee-1fd5-49ca-b55c-6e0c5e6dd961',
            usernameVariable: 'DOCKER_USERNAME',
            passwordVariable: 'DOCKER_PASSWORD'
          )
        ]) {
          sh '''
            echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin
            docker push $DOCKER_BFLASK_IMAGE
          '''
        }
      }
    }

    stage('Deploy to Server') {
      steps {
        sshagent(['deploy-ssh-key']) {
          sh '''
ssh -o StrictHostKeyChecking=no ${DEPLOY_USER}@${DEPLOY_HOST} << 'EOF'
echo " Deploy started on $(hostname)"
docker pull ashwinapple/ci-cd-jenkins:latest

docker stop flask-app || true
docker rm flask-app || true

docker run -d \
  --restart unless-stopped \
  --name flask-app \
  -p 5000:5000 \
  ashwinapple/ci-cd-jenkins:latest

docker ps | grep flask-app
echo "Deploy completed"
EOF
'''
        }
      }
    }
  }

  post {
    always {
      sh 'docker logout || true'
    }

    success {
      echo " CI/CD Pipeline SUCCESS – App deployed!"
    }

    failure {
      echo " Pipeline FAILED – Check logs"
    }
  }
}

