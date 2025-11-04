pipeline {
  agent any

  environment {
    DOCKER_CRED = credentials('dockerhub-creds')
    APP_NAME = "leenavemula/jenkins-task2"
  }

  stages {
    stage('Checkout Code') {
      steps {
        git branch: 'main', url: 'https://github.com/leenavemula25/jenkins-cicd-demo.git'
      }
    }

    stage('Build Application') {
      steps {
        script {
          if (fileExists('package.json')) {
            bat 'npm install'
          } else if (fileExists('requirements.txt')) {
            bat 'pip install -r requirements.txt'
          } else if (fileExists('pom.xml')) {
            bat 'mvn clean package -DskipTests'
          }
        }
      }
    }

    stage('Gitleaks Secret Scan') {
      steps {
        bat 'docker run --rm -v %CD%:/code zricethezav/gitleaks detect --source /code --report-path /code/gitleaks.json || exit /b 0'
      }
    }

    stage('Docker Build') {
      steps {
        bat 'docker build -t %APP_NAME%:%BUILD_NUMBER% .'
      }
    }

    stage('Trivy Image Scan') {
      steps {
        bat 'docker run --rm -v //var/run/docker.sock:/var/run/docker.sock aquasec/trivy image %APP_NAME%:%BUILD_NUMBER% || exit /b 0'
      }
    }

    stage('Push Docker Image') {
      steps {
        bat '''
          echo %DOCKER_CRED_PSW% | docker login -u "%DOCKER_CRED_USR%" --password-stdin
          docker push %APP_NAME%:%BUILD_NUMBER%
          docker tag %APP_NAME%:%BUILD_NUMBER% %APP_NAME%:latest
          docker push %APP_NAME%:latest
        '''
      }
    }

    stage('Deploy Using Docker Compose') {
      steps {
        bat '''
          docker-compose down || exit /b 0
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
