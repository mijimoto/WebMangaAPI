pipeline {
  agent any

  environment {
    // SonarQube settings
    SONARQUBE_NAME = 'MySonar'                          // Name configured in Jenkins → SonarQube servers
    SONAR_TOKEN    = credentials('sonar-token')         // Secret Text credential ID for Sonar token

    // Docker Hub settings
    DOCKER_CREDENTIALS = 'docker-creds'                 // Credential ID in Jenkins for Docker Hub
    DOCKER_REGISTRY = 'https://index.docker.io/v1/'     // Docker Hub registry URL
    IMAGE_NAMESPACE = 'mijimoto'                        // Your Docker Hub username or org
    IMAGE_TAG = "${env.BUILD_NUMBER}"                   // Use Jenkins build number as tag
  }

  stages {
    stage('Checkout API') {
      steps {
        echo '📦 Checking out WebMangaAPI...'
        git(
          url: 'https://github.com/mijimoto/WebMangaAPI.git',
          branch: 'main',
          credentialsId: 'github-creds'                 // GitHub credential ID
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
          // Use `bat` for Windows shell
          bat """
            dotnet sonarscanner begin ^
              /k:"WebMangaAPI" ^
              /d:sonar.login=${SONAR_TOKEN}
            dotnet build MangaAPI.sln
            dotnet sonarscanner end /d:sonar.login=${SONAR_TOKEN}
          """
        }
      }
    }

    stage('Build and Push Docker Images') {
      steps {
        echo '🐳 Building and pushing Docker images...'
        script {
          docker.withRegistry("${DOCKER_REGISTRY}", "${DOCKER_CREDENTIALS}") {

            // Backend API image
            def apiImage = docker.build("${IMAGE_NAMESPACE}/webmanga-api:${IMAGE_TAG}", ".")

            // Frontend image (in ./frontend directory)
            def feImage = docker.build("${IMAGE_NAMESPACE}/webmanga-frontend:${IMAGE_TAG}", "frontend")

            // Push both
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
