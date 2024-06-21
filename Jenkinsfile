pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "your-repo/EventApp:${env.BUILD_ID}"
    }

    stages {
        stage('Clone Repository') {
            steps {
                checkout([
                    $class: 'GitSCM', 
                    branches: [[name: '*/jenkins']], 
                    userRemoteConfigs: [[url: 'https://github.com/bhavyansh001/EventApp.git']]
                ])
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    docker.build(DOCKER_IMAGE)
                }
            }
        }
        stage('Run RSpec Tests') {
            steps {
                script {
                    docker.image(DOCKER_IMAGE).inside {
                        sh 'bundle install'
                        sh 'bundle exec rspec'
                    }
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
