pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "https://github.com/bhavyansh001/EventApp:${env.BUILD_ID}"
    }

    stages {
        stage('Prepare Environment') {
            steps {
                script {
                    // Check if Docker CLI is available
                    if (!fileExists('/usr/bin/docker')) {
                        // Install Docker in rootless mode if not available
                        sh '''
                            curl -fsSL https://get.docker.com/rootless | sh
                        '''
                        // Add the current Jenkins user to the dockerd-rootless group
                        sh '''
                            sudo usermod -aG dockerd-rootless jenkins
                        '''
                        // Start the rootless Docker daemon
                        sh '''
                            dockerd-rootless-setuptool.sh install
                            dockerd-rootless-setuptool.sh start
                        '''
                        // Wait for dockerd-rootless to start
                        sh '''
                            sleep 10
                        '''
                    }
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
                    sudo 'docker build -t ' + DOCKER_IMAGE
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
