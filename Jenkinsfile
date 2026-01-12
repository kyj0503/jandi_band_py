pipeline {
    agent any
    
    environment {
        GHCR_OWNER = 'kyj0503'
        IMAGE_NAME = 'jandi-band-py'
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Build and Push to GHCR') {
            when {
                anyOf {
                    branch 'main'
                    branch 'master'
                }
            }
            steps {
                script {
                    def fullImageName = "ghcr.io/${env.GHCR_OWNER}/${env.IMAGE_NAME}:${env.BUILD_NUMBER}"
                    def latestImageName = "ghcr.io/${env.GHCR_OWNER}/${env.IMAGE_NAME}:latest"
                    
                    echo "Building Docker image: ${fullImageName}"
                    
                    docker.withRegistry("https://ghcr.io", 'github-token') {
                        // 캐시용 이미지 Pull (실패해도 무시)
                        sh "docker pull ${latestImageName} || true"
                        
                        // 캐시를 활용하여 빌드
                        docker.build(fullImageName, "--cache-from ${latestImageName} .")
                        
                        // Push
                        echo "Pushing Docker image to GHCR..."
                        docker.image(fullImageName).push()
                        docker.image(fullImageName).push('latest')
                    }
                }
            }
        }

        // 배포는 home-server에서 담당
        stage('Trigger Deploy') {
            when {
                anyOf {
                    branch 'main'
                    branch 'master'
                }
            }
            steps {
                build job: 'home-server-deploy', wait: false, propagate: false
            }
        }
    }
    
    post {
        always {
            cleanWs()
        }
        success {
            echo '✅ Build and Push completed successfully!'
        }
        failure {
            echo '❌ Build failed!'
        }
    }
}
