pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('Build Image'){
          when {
            branch 'master'
          }
          steps{
            script{
              app = docker.build("julius380/train-schedule")
              }
            }
        }

        stage('Test Image'){
          steps{
            script{
          app.inside {
            sh 'echo $(curl localhost:8080)'
          }}
        }}

        stage('Push Image to Repo'){
          when {
            branch 'master'
          }
          steps{
              withCredentials([usernamePassword(credentialsId: 'docker-credentials', usernameVariable: '$USERNAME', passwordVariable: '$PASSWORD')]){
            script{               
            sh 'aws ecr get-login-password --region eu-west-1 | login --username "$USERNAME" --password-stdin 459689308206.dkr.ecr.eu-west-1.amazonaws.com'
            sh 'docker push 459689308206.dkr.ecr.eu-west-1.amazonaws.com/jenkins:latest'
            }
            }
          }}
        
        stage('Deploy to Production'){
          when {
            branch 'master'
          }
          steps{
            input 'Has the Image been built?'
            milestone(1)
            withCredentials([usernamePassword(credentialsId: 'SSH-user', usernameVariable: '$USERNAME', passwordVariable: '$PASSWORD')]){
              script{
                try{
                sh 'sshpass -p '$PASSWORD' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod-ip \"docker stop train-schedule\"'
                sh 'sshpass -p '$PASSWROD' -v ssh -o StrictHostKeyChecking=no $USERNAME@prod-ip \"docker rm train-schedule\"'
                } catch (err) {
                  echo : 'caught error: $err'
                }
                sh 'sshpass -p '$PASSWORD' -v ssh -o StrictHostKeyChecking=no $USERNAME@prod-ip \"docker run -p 8080:8080 --name train-schedule --restart always -d c3y2i9q7/jenkins:${env.BUILD_NUMBER}\"'
              }
            }

          }

        }     
}
