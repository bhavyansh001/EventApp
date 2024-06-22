pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "https://github.com/bhavyansh001/EventApp:${env.BUILD_ID}"
        
    }

    stages {
        stage('Prepare Environment') {
            steps {
                script {
                    // Download and install Docker
                    sh '''
                        set -e
                        if ! command -v docker &> /dev/null
                        then
                            curl -fsSL https://get.docker.com -o get-docker.sh
                            sudo sh get-docker.sh
                            sudo usermod -aG docker jenkins
                            sudo systemctl restart docker
                         else
                            echo "Docker is already installed."
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
                         else
                            echo "Docker Compose is already installed."
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
            //  environment {
            //     POSTGRES_USER = "${env.POSTGRES_USER}"
            //     POSTGRES_PASSWORD = "${env.POSTGRES_PASSWORD}"
            //     POSTGRES_DB = "${env.POSTGRES_DB}"
            // }
            steps {
                script {
                     // Create .env file with the environment variables
                    sh """
                        echo "POSTGRES_USER=${env.POSTGRES_USER}" > .env
                        echo "POSTGRES_PASSWORD=${env.POSTGRES_PASSWORD}" >> .env
                        echo "POSTGRES_DB=${env.POSTGRES_DB}" >> .env
                        sudo docker-compose build
                    """
                }
            }
        }

        stage('Run RSpec Tests') {
            steps {
                script {
                    sh """
                        docker-compose exec web bundle install
                        docker-compose exec web bundle exec rspec
                        sudo docker-compose build
                    """
                }
            }
        }
    }

    post {
        always {
            // Clean workspace
            // cleanWs()
        }
    }
}
