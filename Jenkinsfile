pipeline{
    agent any
    environment {
      VERSION = "${env.BUILD_ID}"
      }
    stages{
        stage('Sonar Quality Check') {
          agent {
             docker {
                 image 'maven'
                 args '-u root'
                   }
               }
            steps{
              script{
                 withSonarQubeEnv(credentialsId: 'sonar-token') {
                 sh 'mvn clean package sonar:sonar'
                 }
              }
           }
       }
       stage('Quality Gate Status'){
         steps{
           script{
             waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
               }
            }
        }
      stage('Docker build & Docker image push to Nexus'){
         steps{
           script{
            withCredentials([string(credentialsId: 'nexus_passwd', variable: 'nexus_cred')]){
             sh '''
             docker build -t 34.219.223.174:8083/springapp:${VERSION} .
             docker login -u admin -p ${nexus_cred} 34.219.223.174:8083
             docker push 34.219.223.174:8083/springapp:${VERSION}
             docker rmi 34.219.223.174:8083/springapp:${VERSION}
             '''
                 }
               }
            }
        }
        stage('k8s connect'){
            steps{
                sshagent(['kube8']) {
                sh 'scp -o StrictHostKeyChecking=no /var/lib/jenkins/workspace/sonarandnexus/newdeploy.yml ubuntu@18.236.246.12'
                sh 'kubectl create -f newdeploy.yml'
                }
            }
        }
    }
}
