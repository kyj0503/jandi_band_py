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
                checkout scm
            }
        }

        stage('Setup Buildx') {
            when {
                anyOf {
                    branch 'main'
                    branch 'master'
                }
            }
            steps {
                script {
                    // Docker Buildx 설정 (멀티 아키텍처 빌드용)
                    sh '''
                        docker buildx create --name multiarch-builder --use --bootstrap || docker buildx use multiarch-builder
                        docker buildx inspect --bootstrap
                    '''
                }
            }
        }
        
        stage('Build and Push Multi-Arch Image') {
            when {
                anyOf {
                    branch 'main'
                    branch 'master'
                }
            }
            steps {
                script {
                    def fullImageName = "ghcr.io/${env.GHCR_OWNER}/${env.IMAGE_NAME}"
                    
                    echo "Building multi-arch Docker image: ${fullImageName}"
                    
                    // GHCR 로그인
                    withCredentials([usernamePassword(credentialsId: 'github-token', usernameVariable: 'GITHUB_USER', passwordVariable: 'GITHUB_TOKEN')]) {
                        sh 'echo $GITHUB_TOKEN | docker login ghcr.io -u $GITHUB_USER --password-stdin'
                    }
                    
                    // 멀티 아키텍처 빌드 및 푸시 (AMD64 + ARM64)
                    sh """
                        docker buildx build \
                            --platform linux/amd64,linux/arm64 \
                            --tag ${fullImageName}:${env.BUILD_NUMBER} \
                            --tag ${fullImageName}:latest \
                            --push \
                            .
                    """
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
