pipeline {
  agent any

  environment {
    DOCKER_BFLASK_IMAGE = "ashwinapple/ci-cd-jenkins:latest"
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

    stage('Deploy') {
      steps {

        withCredentials([
          usernamePassword(
            credentialsId: 'a11330ee-1fd5-49ca-b55c-6e0c5e6dd961',
            usernameVariable: 'DOCKER_USERNAME',
            passwordVariable: 'DOCKER_PASSWORD'
          )
        ]) {
          sh '''
            echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin
            docker push $DOCKER_BFLASK_IMAGE
          '''
        }

        sshagent(['deploy-ssh-key']) {
          sh '''
            ssh -o StrictHostKeyChecking=no comtel@192.168.101.62 << 'EOF'
              docker pull ashwinapple/ci-cd-jenkins:latest
              docker stop flask-app || true
              docker rm flask-app || true
              docker run -d --name flask-app -p 5000:5000 ashwinapple/ci-cd-jenkins:latest
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
  }
}

