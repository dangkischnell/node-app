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
          ]) {
            sh '''
               rm -rf node-app-terraform
               git clone https://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@github.com/dangkischnell/node-app-terraform.git
               cd node-app-terraform
               terraform init        
               terraform apply -auto-approve -var access_key=${AWS_KEY} -var secret_key=${AWS_SECRET}
               git add terraform.tfstate
               git -c user.name="dawefd" -c user.email="ffff@gmail.com" commit -m "terraform state update from Jenkins"               
            '''
            }
            sh 'git push https://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@github.com/dangkischnell/node-app-terraform.git master'          
    }
  }
 } 
  environment {
    npm_config_cache = 'npm-cache'
  }
}

