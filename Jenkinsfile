pipeline {
  agent none

  stages {
    stage('Create Packer AMI') {
        agent {
          docker {
            image 'jeduoliveira/terraform-packer-aws-alpine:latest'
          }
        }
        steps {
          withCredentials([
            usernamePassword(credentialsId: '63715168-c881-45f2-a269-873208bf331e', passwordVariable: 'AWS_SECRET', usernameVariable: 'AWS_KEY')
          ]) {
            sh 'packer build -var aws_access_key=${AWS_KEY} -var aws_secret_key=${AWS_SECRET} packer/packer.json'
        }
      }
    }
    stage('AWS Deployment') {
      agent {
        docker {
          image 'ksandermann/cloud-toolbox:latest'
        }
      }

      steps {
          withCredentials([
            usernamePassword(credentialsId: '63715168-c881-45f2-a269-873208bf331e', passwordVariable: 'AWS_SECRET', usernameVariable: 'AWS_KEY'),
            sshUserPrivateKey(credentialsId: 'github_ssh', keyFileVariable: 'SSH_KEY')
          ]) {
            sh 'rm -rf node-app-terraform'
            sh 'GIT_SSH_COMMAND="ssh -i $SSH_KEY" git clone git@github.com:dangkischnell/node-app-terraform.git'
            sh '''
               cd node-app-terraform
               terraform init
               terraform apply -auto-approve -var access_key=${AWS_KEY} -var secret_key=${AWS_SECRET}
               git add terraform.tfstate
               git -c user.name="Shashwat Tripathi" -c user.email="shashwat2691@gmail.com" commit -m "terraform state update from Jenkins"
               sh 'GIT_SSH_COMMAND="ssh -i $SSH_KEY" git push git@github.com:dangkischnell/node-app-terraform.git master
            '''
        }
      }
    }
  }
  environment {
    npm_config_cache = 'npm-cache'
  }
}
