pipeline {
    agent any

    environment  {
      DOCKER_CRED = "docker-cred"
      GITHUB_CRED = "github-cred"
      KUBE_CRED = "kubernetes-cred"
      SONARQUBE_CRED = "sonarqube-cred"
    }
    stages{
        stage("Checkout"){
            steps{
                git credentialsId: "${GITHUB_CRED}", url: "git@github.com:ahmedsayedtalib/devops-gitops-demo.git"
            }
        }
        post{
                always{
                    echo "Github Stage Output:- "
                }
                success{
                    echo "========A executed successfully========"
                }
                failure{
                    echo "========A execution failed========"
                }
            }
        stage("SonarQube Test & Analysis"){
            steps{
                withCredentials([string(credentialId:"${SONARQUBE_CRED}"),variable:"SONAR_TOKEN"])
            }
        }
            post{
                always{
                    echo "SonarQube Stage Output:- "
                }
                success{
                    echo "========SonarQube executed successfully========"
                }
                failure{
                    echo "========SonarQube execution failed========"
                }
            }
        }
}
