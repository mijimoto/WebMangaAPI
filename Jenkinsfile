pipeline {
    agent any

    environment {
        SONARQUBE_ENV = 'MySonar'
        DOCKER_IMAGE = 'ghcr.io/mijimoto/webmangaapi:latest'
    }

    stages {
        stage('Checkout') {
            steps {
                git credentialsId: 'github-creds', url: 'https://github.com/mijimoto/WebMangaAPI.git'
            }
        }

        stage('SonarQube Code Scan') {
            steps {
                withSonarQubeEnv(SONARQUBE_ENV) {
                    bat "dotnet sonarscanner begin /k:\"webmangaapi\" /d:sonar.login=$SONAR_TOKEN"
                    bat "dotnet build"
                    bat "dotnet sonarscanner end /d:sonar.login=$SONAR_TOKEN"
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("webmangaapi")
                }
            }
        }

        stage('Push to GHCR') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'ghcr-token', variable: 'GHCR_TOKEN')]) {
                        sh '''
                            echo $GHCR_TOKEN | docker login ghcr.io -u mijimoto --password-stdin
                            docker tag webmangaapi ghcr.io/mijimoto/webmangaapi:latest
                            docker push ghcr.io/mijimoto/webmangaapi:latest
                        '''
                    }
                }
            }
        }

        stage('Run Image (Optional)') {
            steps {
                sh 'docker run -d -p 8081:80 ghcr.io/mijimoto/webmangaapi:latest'
            }
        }
    }
}
