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
                        sudo docker-compose up --detach
                    """
                }
            }
        }

        stage('Run RSpec Tests') {
            steps {
                script {
                    sh """
                        sudo docker-compose exec -T web bundle install
                        sudo docker-compose exec -T web bundle exec bin/rails db:migrate
                        sudo docker-compose exec -T web bundle exec rspec
                    """
                }
            }
        }

        stage('SonarQube Analysis') {
            environment {
                scannerHome = tool 'sonar4.7'
                reportPath = "${WORKSPACE}/sonar-report"
            }
            steps {
                withSonarQubeEnv('sonar') {

                    // Create the report directory if it doesn't exist
                     sh "mkdir -p ${reportPath}"

                    // Run SonarQube Scan
                    sh '''
                        ${scannerHome}/bin/sonar-scanner \
                            -Dsonar.projectName=eventapp \
                            -Dsonar.projectVersion=1.0 \
                            -Dsonar.sources=app/controllers/ \
                            -Dsonar.login=your_sonarqube_token \
                            -Dsonar.projectKey=ruby-eventapp \
                            -Dsonar.sources=. \
                            -Dsonar.exclusions=**/vendor/**,**/config/**,**/log/**,**/tmp/** \
                            -Dsonar.reportPaths=${reportPath}
                    '''
                }
            }
        }
    }
}

    post {
        always {
            // Bring down docker-compose
            sh 'sudo docker-compose down'
            cleanWs()
        }
    }
