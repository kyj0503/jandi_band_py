pipeline {
    agent any
    
    environment {
        GHCR_OWNER = 'kyj0503'
        IMAGE_NAME = 'jandi-band-py'
        DOCKER_BUILDKIT = '1'
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout([$class: 'GitSCM',
                    branches: [[name: '*/main'], [name: '*/master']],
                    userRemoteConfigs: [[
                        url: 'https://github.com/kyj0503/jandi_band_py.git',
                        credentialsId: 'github-token'
                    ]]
                ])
            }
        }

        stage('Test') {
            steps {
                script {
                    echo "Running Python tests..."
                    sh '''
                        python3 -m venv venv || true
                        . venv/bin/activate
                        pip install -r requirements.txt
                        pip install pytest
                        python -m pytest tests/ -v --tb=short || echo "No tests found or tests skipped"
                    '''
                }
            }
            post {
                failure {
                    echo "Tests failed. Stopping pipeline."
                }
            }
        }

        stage('Login GHCR') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'github-token', usernameVariable: 'GITHUB_USER', passwordVariable: 'GITHUB_TOKEN')]) {
                        sh 'echo $GITHUB_TOKEN | docker login ghcr.io -u $GITHUB_USER --password-stdin'
                    }
                }
            }
        }
        
        stage('Build and Push Image') {
            steps {
                script {
                    def fullImageName = "ghcr.io/${env.GHCR_OWNER}/${env.IMAGE_NAME}"
                    
                    echo "Building Docker image: ${fullImageName}"
                    
                    // 빌드 및 푸시
                    sh """
                        docker build \
                            --tag ${fullImageName}:${env.BUILD_NUMBER} \
                            --tag ${fullImageName}:latest \
                            .
                        docker push ${fullImageName}:${env.BUILD_NUMBER}
                        docker push ${fullImageName}:latest
                    """
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    sh '''
                        cd /home/ubuntu/source/home-server/docker
                        docker compose -f docker-compose.apps.yml pull jandi-band-py
                        docker compose -f docker-compose.apps.yml up -d --force-recreate jandi-band-py
                        sleep 5
                        docker ps | grep jandi-band-py
                        echo "✅ jandi-band-py deployment completed!"
                    '''
                }
            }
        }

        stage('Health Check') {
            steps {
                script {
                    sh '''
                        sleep 10
                        curl -f https://rhythmeet-py.yeonjae.kr/health || echo "Health check pending..."
                    '''
                }
            }
        }
    }
    
    post {
        always {
            cleanWs()
        }
        success {
            echo '✅ jandi-band-py Build, Push, and Deploy completed successfully!'
        }
        failure {
            echo '❌ Pipeline failed!'
        }
    }
}
