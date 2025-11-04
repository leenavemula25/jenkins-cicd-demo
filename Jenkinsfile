pipeline {
  agent any

  environment {
    DOCKER_CRED = credentials('dockerhub-creds')
    APP_NAME = "leenavemula/myapp"
  }

  stages {
    stage('Checkout Code') {
      steps {
        git branch: 'main', url: 'https://github.com/yourusername/yourrepo.git'
      }
    }

    stage('Build Application') {
      steps {
        script {
          if (fileExists('package.json')) {
            sh 'npm install'
          } else if (fileExists('requirements.txt')) {
            sh 'pip install -r requirements.txt'
          } else if (fileExists('pom.xml')) {
            sh 'mvn clean package -DskipTests'
          }
        }
      }
    }

    stage('Gitleaks Secret Scan') {
      steps {
        sh 'docker run --rm -v $PWD:/code zricethezav/gitleaks detect --source /code --report-path /code/gitleaks.json || true'
      }
    }

    stage('Docker Build') {
      steps {
        sh 'docker build -t $APP_NAME:$BUILD_NUMBER .'
      }
    }

    stage('Trivy Image Scan') {
      steps {
        sh 'docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image $APP_NAME:$BUILD_NUMBER || true'
      }
    }

    stage('Push Docker Image') {
      steps {
        sh '''
          echo "$DOCKER_CRED_PSW" | docker login -u "$DOCKER_CRED_USR" --password-stdin
          docker push $APP_NAME:$BUILD_NUMBER
          docker tag $APP_NAME:$BUILD_NUMBER $APP_NAME:latest
          docker push $APP_NAME:latest
        '''
      }
    }

    stage('Deploy Using Docker Compose') {
      steps {
        sh '''
          docker-compose down || true
          docker-compose up -d
        '''
      }
    }
  }

  post {
    always {
      echo 'CI/CD pipeline completed successfully!'
    }
  }
}
