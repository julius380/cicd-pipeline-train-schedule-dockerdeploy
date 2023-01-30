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
            script{
            docker.withRegistry('https://public.ecr.aws/c3y2i9q7/jenkins','docker-credentials'){
              app.push("${env.BUILD_NUMBER}")
              app.push("latest")
            }
          }}
        }

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
                sh 'sshpass -p '$PASSWORD' -v ssh -o StrictHostKeyChecking=no $USERNAME@prod-ip \"docker run -p 8080:8080 --name train-schedule --restart always -d julius380/train-schedule:${env.BUILD_NUMBER}\"'
              }
            }

          }

        }
                 
          }
        }
  
