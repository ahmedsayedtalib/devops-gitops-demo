pipeline {
    agent any 
    environment {
        // IDs for Credentials items stored in Jenkins.
        GITHUB_CRED = "github-cred" 
        SONAR_TOKEN = "sonarqube-cred" 
        DOCKER_CRED = "docker-cred" 
        // KUBE_CRED لم يعد مستخدما للنشر المباشر
        
        // متغيرات ArgoCD/GitOps (الآن كل شيء في نفس المستودع)
        // أنت تستخدم نفس المستودع لكل من الكود وملفات النشر (Mono-Repo)
        GITOPS_REPO_URL = "https://github.com/ahmedsayedtalib/devops-gitops-demo.git" // تم تحديثه لرابطك
        GITOPS_CRED = "github-cred" 
        // المسار لملف الـ Deployment اللي محتاج يتحدث
        GITOPS_DEPLOYMENT_FILE = "k8s/erp-app/deployment.yaml" 
        
        // متغيرات إضافية
        DOCKER_REPO = "ahmedsayedtalib/devops-gitops-demo"
        // تصحيح الـ URL: إزالة الـ 'http://' المكرر و الـ 'http://'
        SONAR_URL = "http://192.168.103.2:32000" 
    }

    stages {
        stage ("Checkout") {
            steps {
                echo "Signing in to git repo"
                // هذا الـ checkout لملفات الكود و Manifests
                git branch: 'main', credentialsId: env.GITHUB_CRED, url: 'https://github.com/ahmedsayedtalib/devops-gitops-demo.git'
            }
        }
        
        stage ("Sonarqube Scan") {
            steps {
                withCredentials([string(credentialsId: env.SONAR_TOKEN, variable: 'SONAR_AUTH_TOKEN')]) {
                    echo "Starting SonarQube Scan..."
                    sh "${tool 'sonar-scanner'} -Dsonar.projectKey=my-project -Dsonar.sources=. -Dsonar.host.url=${env.SONAR_URL} -Dsonar.login=\$SONAR_AUTH_TOKEN"
                }
            }
        }

        stage ("Docker Build and Push") {
            steps {
                echo "Building Docker Image..."
                withCredentials([usernamePassword(credentialsId: env.DOCKER_CRED, 
                                                  usernameVariable: 'DOCKER_USER', 
                                                  passwordVariable: 'DOCKER_PASS')]) {
                    
                    sh "echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin"
                    
                    script {
                        // استخدام رقم البناء كـ Tag للصورة
                        env.DOCKER_IMAGE_TAG = "${env.BUILD_NUMBER}"
                        env.DOCKER_IMAGE_NAME_FULL = "${env.DOCKER_REPO}:${env.DOCKER_IMAGE_TAG}"
                    }
                    
                    sh "docker build -t ${env.DOCKER_IMAGE_NAME_FULL} ."
                    sh "docker push ${env.DOCKER_IMAGE_NAME_FULL}"
                    sh "docker logout"
                }
            }
        }
        
        // 🚨 مرحلة ArgoCD GitOps (لتحديث ملف الـ Deployment)
        stage ("ArgoCD GitOps Deployment") {
            steps {
                echo "Updating GitOps Deployment Manifest for ArgoCD sync..."
                
                script {
                    // 1. تحديد الصورة الجديدة بالـ Tag بتاع رقم البناء
                    def newImage = "${env.DOCKER_REPO}:${env.DOCKER_IMAGE_TAG}"
                    
                    // 2. تحديث ملف الـ Deployment مباشرةً (Deployment Manifest Update)
                    // بنستخدم sed عشان نحدث السطر اللي فيه 'image: ...' في deployment.yaml
                    // بيبحث عن السطر اللي بيبدأ بـ 'image:' وبيستبدله بالقيمة الجديدة
                    // (على افتراض أن ملف deployment.yaml لا يحتوي على مسافات بادئة كثيرة قبل image:)
                    sh "sed -i '/image:/c\\        image: ${newImage}' ${env.GITOPS_DEPLOYMENT_FILE}"
                
                    // 3. عمل Commit للتغيير في الـ Manifest
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