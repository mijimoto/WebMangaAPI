pipeline {
  agent any

  environment {
    // SonarQube settings
    SONARQUBE_NAME = 'MySonar'
    SONAR_TOKEN    = credentials('sonar-token')         // Secret text ID for SonarQube

    // Docker Hub settings
    DOCKER_CREDENTIALS = 'docker-creds'
    DOCKER_REGISTRY    = 'https://index.docker.io/v1/'
    IMAGE_NAMESPACE    = 'mijimoto'
    IMAGE_TAG          = "${env.BUILD_NUMBER}"
  }

  stages {

    stage('Checkout API') {
      steps {
        echo '📦 Checking out WebMangaAPI...'
        git(
          url: 'https://github.com/mijimoto/WebMangaAPI.git',
          branch: 'main',
          credentialsId: 'github-creds'
        )
      }
    }

    stage('Checkout Frontend') {
      steps {
        echo '🌐 Checking out WebManga (frontend)...'
        dir('frontend') {
          git(
            url: 'https://github.com/mijimoto/WebManga.git',
            branch: 'main',
            credentialsId: 'github-creds'
          )
        }
      }
    }

    stage('SonarQube Scan') {
      steps {
        echo '🔍 Running SonarQube scan...'
        withSonarQubeEnv("${SONARQUBE_NAME}") {
          bat '''
          C:\\Users\\Admin\\.dotnet\\tools\\dotnet-sonarscanner begin /k:"WebMangaAPI" /d:sonar.login=%SONAR_TOKEN%
          dotnet build MangaAPI.sln
          C:\\Users\\Admin\\.dotnet\\tools\\dotnet-sonarscanner end /d:sonar.login=%SONAR_TOKEN%
          '''
        }
      }
    }

    stage('Build and Push Docker Images') {
      steps {
        echo '🐳 Building and pushing Docker images...'
        script {
          docker.withRegistry("${DOCKER_REGISTRY}", "${DOCKER_CREDENTIALS}") {
            def apiImage = docker.build("${IMAGE_NAMESPACE}/webmanga-api:${IMAGE_TAG}", ".")
            def feImage  = docker.build("${IMAGE_NAMESPACE}/webmanga-frontend:${IMAGE_TAG}", "frontend")

            apiImage.push()
            feImage.push()
          }
        }
      }
    }
  }

  post {
    success {
      echo "✅ Build #${env.BUILD_NUMBER} completed successfully!"
    }
    failure {
      echo "❌ Build failed. Check console output for errors."
    }
  }
}
