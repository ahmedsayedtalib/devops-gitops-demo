pipeline {
    agent any

    environment {
        DOCKER_CRED = credentials('docker-cred')
        GITHUB_CRED = "github-cred"

        DOCKER_REPO = "ahmedsayedtalib/erp-ahmedsayed"
        IMAGE_TAG = "${BUILD_NUMBER}"

        GITOPS_FILE = "k8s/erp-app/deployment.yaml"
    }

    stages {

        stage("Checkout") {
            steps {
                git branch: 'main',
                    credentialsId: env.GITHUB_CRED,
                    url: 'https://github.com/ahmedsayedtalib/devops-gitops-demo.git'
            }
        }

        stage("Docker Build & Push") {
            steps {
                script {
                    sh """
                        echo ${DOCKER_CRED_PSW} | docker login -u ${DOCKER_CRED_USR} --password-stdin
                        docker build -t ${DOCKER_REPO}:${IMAGE_TAG} ./app
                        docker push ${DOCKER_REPO}:${IMAGE_TAG}
                    """
                }
            }
        }

        stage("Update GitOps Deployment") {
            steps {
                script {

                    // تحديث image داخل ملف الـ deployment
                    sh """
                        sed -i 's|image: .*|image: ${DOCKER_REPO}:${IMAGE_TAG}|g' ${GITOPS_FILE}
                    """

                    sh """
                        git config user.email 'jenkins@ci.com'
                        git config user.name 'Jenkins CI'
                        git add ${GITOPS_FILE}
                        git commit -m "Update image to ${IMAGE_TAG}"
                        git push origin main
                    """
                }
            }
        }
    }

    post {
        always {
            sh "docker logout"
        }
    }
}
