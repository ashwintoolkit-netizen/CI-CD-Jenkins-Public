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
      }
    }
  }

  post {
    always {
      sh 'docker logout || true'
    }
  }
}

