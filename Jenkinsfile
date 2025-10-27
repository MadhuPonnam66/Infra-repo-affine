pipeline {
    agent any

    environment {
        GHCR_USER   = credentials('gh-user-name')
        GHCR_TOKEN  = credentials('gh-user-pat')
        GHCR_IMAGE  = "ghcr.io/MadhuPonnam66/affine"
        INFRA_REPO  = "git@github.com:MadhuPonnam66/infra-repo-affine"
        INFRA_BRANCH = "main"
    }

    stages {
        stage('Checkout Source') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/MadhuPonnam66/AFFiNE.git'
            }
        }

        stage('Build & Push Image') {
            steps {
                script {
                    def tag = sh(script: "date +%Y%m%d%H%M%S", returnStdout: true).trim()
                    env.IMAGE_TAG = tag

                    sh """
                        docker build -t ${GHCR_IMAGE}:${tag} -t ${GHCR_IMAGE}:latest .
                        echo "${GHCR_TOKEN}" | docker login ghcr.io -u "${GHCR_USER}" --password-stdin
                        docker push ${GHCR_IMAGE}:${tag}
                        docker push ${GHCR_IMAGE}:latest
                    """
                }
            }
        }

        stage('Update Infra Repo') {
            steps {
                script {
                    sh """
                        git config --global user.name "Jenkins CI"
                        git config --global user.email "jenkins@ci.local"
                    """

                    sh "rm -rf infra && git clone -b ${INFRA_BRANCH} ${INFRA_REPO} infra"

                    sh """
                        cd infra
                        sed -i 's#ghcr.io/.*/affine:.*#${GHCR_IMAGE}:${IMAGE_TAG}#' k8s/affine-deployment.yaml
                        git add k8s/affine-deployment.yaml
                        git commit -m "Update Affine image to ${IMAGE_TAG}"
                        git push origin ${INFRA_BRANCH}
                    """
                }
            }
        }
    }

    post {
        always {
            node {
                sh 'docker logout ghcr.io || true'
                sh 'docker system prune -af || true'
            }
        }
        success {
            echo "✅ Image pushed and infra repo updated to tag: ${IMAGE_TAG}"
        }
        failure {
            echo "❌ Pipeline failed"
        }
    }
}

