pipeline {
    agent any

    environment {
        GITHUB_USER = 'MadhuPonnam66'      // original case for cloning
        GHCR_USER   = 'madhuponnam66'      // lowercase for Docker image
        GHCR_TOKEN  = credentials('gh-user-pat')
        REPO_NAME   = 'AFFiNE'
        DOCKERFILE_PATH = 'AFFiNE/custom-dockerfile/Dockerfile'
    }

    stages {
        stage('Checkout Source') {
            steps {
                git branch: 'canary', url: "https://github.com/${GITHUB_USER}/${REPO_NAME}.git"
            }
        }

        stage('Build & Push Image') {
            steps {
                script {
                    def tag = sh(script: "date +%Y%m%d%H%M%S", returnStdout: true).trim()
                    env.IMAGE_TAG = tag

                    sh 'pwd && ls -R' // 👈 Debug, you can remove after verifying

                    sh """
                        docker build -f ${DOCKERFILE_PATH} \
                            -t ghcr.io/${GHCR_USER}/affine:${tag} \
                            -t ghcr.io/${GHCR_USER}/affine:latest .
                        echo '${GHCR_TOKEN}' | docker login ghcr.io -u '${GHCR_USER}' --password-stdin
                        docker push ghcr.io/${GHCR_USER}/affine:${tag}
                        docker push ghcr.io/${GHCR_USER}/affine:latest
                    """
                }
            }
        }
    }

    post {
        always {
            sh 'docker logout ghcr.io || true'
            sh 'docker system prune -af || true'
        }
        success {
            echo "✅ Image pushed and infra repo updated to tag: ${IMAGE_TAG}"
        }
        failure {
            echo "❌ Pipeline failed"
        }
    }
}

