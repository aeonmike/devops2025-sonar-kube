pipeline {
  agent { label 'master' }

  options {
    buildDiscarder(logRotator(numToKeepStr: '5'))
  }

  environment {
    DOCKERHUB_CREDENTIALS = credentials('dockerhub')
    SONARQUBE_ENV = 'sonarqube'
    DOCKER_REPO = 'mikejc30/devops2025'
    IMAGE_TAG = "build-${env.BUILD_NUMBER}"
    IMAGE_NAME = "${env.DOCKER_REPO}:${env.IMAGE_TAG}"
    KUBE_DEPLOYMENT_NAME = 'webapp-deployment'
    KUBE_CONTAINER_NAME = 'webapp'
    KUBE_NAMESPACE = 'testing'
  }

  stages {
    stage('Prepare') {
      steps {
        script {
          echo "Using image: ${env.IMAGE_NAME}"
        }
      }
    }

    stage('SonarQube Scan') {
      steps {
        withSonarQubeEnv('sonarqube') {
          sh '''
            sonar-scanner \
              -Dsonar.projectKey=devops2025-sonar-kube \
              -Dsonar.sources=. \
              -Dsonar.host.url=$SONAR_HOST_URL \
              -Dsonar.login=$SONAR_AUTH_TOKEN
          '''
        }
      }
    }

    stage('Build') {
      steps {
        sh 'docker build -t $IMAGE_NAME .'
      }
    }

    stage('Login') {
      steps {
        sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
      }
    }

    stage('Push') {
      steps {
        sh 'docker push $IMAGE_NAME'
      }
    }

    stage('Deploy to Kubernetes') {
      steps {
        script {
          sh """
            kubectl set image deployment/$KUBE_DEPLOYMENT_NAME \
              $KUBE_CONTAINER_NAME=$IMAGE_NAME \
              --namespace=$KUBE_NAMESPACE
          """
        }
      }
    }
  }
}
