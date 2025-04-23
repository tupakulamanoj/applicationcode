pipeline {
    agent any

    environment {
        IMAGE_NAME = "manoj758/cdimages"
        IMAGE_TAG = "v${BUILD_NUMBER}"
        FULL_IMAGE = "${IMAGE_NAME}:${IMAGE_TAG}"

        APP_REPO_URL = "https://github.com/tupakulamanoj/applicationcode.git"
        MANIFEST_REPO_URL = "https://github.com/tupakulamanoj/manifests.git"
        MANIFEST_REPO_BRANCH = "main"
    }

    stages {
        stage('Checkout App Repo') {
            steps {
                git url: "${APP_REPO_URL}", branch: 'main'
            }
        }

        stage('Build and Push Docker Image') {
            environment {
                DOCKER_CREDS = credentials('dockerhub-credentials')
            }
            steps {
                sh """
                    echo \$DOCKER_CREDS_PSW | docker login -u \$DOCKER_CREDS_USR --password-stdin
                    docker build -t \$FULL_IMAGE .
                    docker push \$FULL_IMAGE
                """
            }
        }

        stage('Clone Manifest Repo and Update Image Tag') {
            environment {
                TOKEN = credentials('github-token')
            }
            steps {
                sh """
                    rm -rf manifests
                    git clone --branch \$MANIFEST_REPO_BRANCH \$MANIFEST_REPO_URL manifests
                    cd manifests
                    git config user.name "jenkins"
                    git config user.email "jenkins@ci.local"

                    git checkout -b update-image-\$BUILD_NUMBER
                    sed -i "s|image: .*|image: \$FULL_IMAGE|" dev/deployment.yaml

                    git add dev/deployment.yaml
                    git commit -m "Update image to \$FULL_IMAGE"
                    git push https://\$TOKEN@github.com/tupakulamanoj/manifests.git update-image-\$BUILD_NUMBER
                """
            }
        }

        stage('Create Pull Request') {
            environment {
                TOKEN = credentials('github-token')
            }
            steps {
                dir('manifests') {
                    sh """
                        echo "\$TOKEN" | gh auth login --with-token
                        gh pr create \\
                          --title "Update image to \$FULL_IMAGE" \\
                          --body "Automated PR to update image to \$FULL_IMAGE" \\
                          --head update-image-\$BUILD_NUMBER \\
                          --base main
                    """
                }
            }
        }
    }
}
