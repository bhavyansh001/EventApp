pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "https://github.com/bhavyansh001/EventApp:${env.BUILD_ID}"
        POSTGRES_USER = "${env.POSTGRES_USER}"
        POSTGRES_PASSWORD = "${env.POSTGRES_PASSWORD}"
        POSTGRES_DB = "${env.POSTGRES_DB}"
    }

    stages {
        stage('Prepare Environment') {
            steps {
                script {
                    // Download and install Docker if not already available
                    sh '''
                        set -e
                        if ! command -v docker &> /dev/null
                        then
                            curl -fsSL https://get.docker.com -o get-docker.sh
                            sudo sh get-docker.sh
                            sudo usermod -aG docker jenkins
                            sudo systemctl restart docker
                        fi
                    '''
                }
                script {
                    // Install Docker Compose
                    sh '''
                        if ! command -v docker-compose &> /dev/null
                        then
                            sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
                            sudo chmod +x /usr/local/bin/docker-compose
                        fi
                    '''
                }
            }
        }

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
                     sh "cd ./EventApp; sudo docker-compose build"
                }
            }
        }

        stage('Run RSpec Tests') {
            steps {
                script {
                    // Run commands inside the Docker container
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
            // Clean workspace after execution
            cleanWs()
        }
    }
}
