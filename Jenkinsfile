pipeline {
    agent any 
    environment {
        // ... (بقية المتغيرات زي ما هي) ...
        GITHUB_CRED = "github-cred" 
        SONAR_TOKEN = "sonarqube-cred" 
        DOCKER_CRED = "docker-cred" 
        GITOPS_REPO_URL = "https://github.com/ahmedsayedtalib/devops-gitops-demo.git" 
        GITOPS_CRED = "github-cred" 
        GITOPS_DEPLOYMENT_FILE = "k8s/erp-app/deployment.yaml" 
        DOCKER_REPO = "ahmedsayedtalib/devops-gitops-demo"
        SONAR_URL = "http://192.168.103.2:32000" 
    }

    stages {
        // ... (مرحلة Checkout ما فيها تغيير) ...
        stage ("Checkout") {
            steps {
                echo "Signing in to git repo"
                git branch: 'main', credentialsId: env.GITHUB_CRED, url: 'https://github.com/ahmedsayedtalib/devops-gitops-demo.git'
            }
        }
        
        stage ("Sonarqube Scan") {
            steps {
                withCredentials([string(credentialsId: env.SONAR_TOKEN, variable: 'SONAR_AUTH_TOKEN')]) {
                    echo "Starting SonarQube Scan..."
                    
                    // 🚨 التعديل يبدأ من هنا لحل مشكلة Permission Denied
                    script {
                        // 1. تحديد المسار الكامل للـ Sonar Scanner
                        def sonarScannerPath = tool 'sonar-scanner'
                        
                        // 2. منح صلاحية التنفيذ (Execute permission) للملف
                        sh "chmod +x ${sonarScannerPath}"
                        
                        // 3. تشغيل الـ Scanner باستخدام المسار المحدد
                        sh "${sonarScannerPath} -Dsonar.projectKey=my-project -Dsonar.sources=. -Dsonar.host.url=${env.SONAR_URL} -Dsonar.login=\$SONAR_AUTH_TOKEN"
                    }
                }
            }
        }

        // ... (بقية المراحل زي ما هي) ...
        stage ("Docker Build and Push") {
            steps {
                echo "Building Docker Image..."
                withCredentials([usernamePassword(credentialsId: env.DOCKER_CRED, 
                                                  usernameVariable: 'DOCKER_USER', 
                                                  passwordVariable: 'DOCKER_PASS')]) {
                    
                    sh "echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin"
                    
                    script {
                        env.DOCKER_IMAGE_TAG = "${env.BUILD_NUMBER}"
                        env.DOCKER_IMAGE_NAME_FULL = "${env.DOCKER_REPO}:${env.DOCKER_IMAGE_TAG}"
                    }
                    
                    sh "docker build -t ${env.DOCKER_IMAGE_NAME_FULL} ."
                    sh "docker push ${env.DOCKER_IMAGE_NAME_FULL}"
                    sh "docker logout"
                }
            }
        }
        
        stage ("ArgoCD GitOps Deployment") {
            steps {
                echo "Updating GitOps Deployment Manifest for ArgoCD sync..."
                
                script {
                    def newImage = "${env.DOCKER_REPO}:${env.DOCKER_IMAGE_TAG}"
                    sh "sed -i '/image:/c\\        image: ${newImage}' ${env.GITOPS_DEPLOYMENT_FILE}"
                
                    sh "git config user.email 'jenkins@ci.com'"
                    sh "git config user.name 'Jenkins CI'"
                    sh "git add ${env.GITOPS_DEPLOYMENT_FILE}"
                    sh "git commit -m 'CI: New Deployment Manifest with image tag ${env.DOCKER_IMAGE_TAG}'"

                    sh "git push origin main"
                }
                echo "GitOps Deployment file updated. ArgoCD will now sync the new image."
            }
        }
    }
    
    post{
        always {
            echo "Pipeline finished."
        }
        success {
            echo "CI/CD Succeeded! New image tag pushed to GitOps repo for ArgoCD sync."
        }
        failure {
            echo "Pipeline failed in one of the stages."
        }
    }
}
