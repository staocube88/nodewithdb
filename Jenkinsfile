pipeline {
    agent any

    environment {
        REGISTRY = "ghcr.io"
        IMAGE = "${REGISTRY}/staocube88/nodewithdb"
        COMMIT_SHA = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
        IMAGE_TAG = "${IMAGE}:${COMMIT_SHA}"
        GIT_PASS = "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'"
    }

    stages {
        stage('Checkout') {
            steps {
                git credentialsId: ${GIT_PASS}, url: 'https://github.com/staocube88/nodewithdb.git'
            }
        }

        stage('Build Image') {
            steps {
                script {
                    sh "docker build -t ${IMAGE_TAG} -f docker/Dockerfile ."
                }
            }
        }

        stage('Push Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-creds-id', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh """
                    echo $PASS | docker login $REGISTRY -u $USER --password-stdin
                    docker push ${IMAGE_TAG}
                    """
                }
            }
        }

        stage('Update Manifest') {
            steps {
                sh """
                yq -i '.spec.template.spec.containers[0].image = "${IMAGE_TAG}"' k8s/appdeploy.yaml
                git config user.name "jenkins"
                git config user.email "jenkins@local"
                git add manifests/deployment.yaml
                git commit -m "Update image to ${IMAGE_TAG}"
                git push origin main
                """
            }
        }
    }
}
