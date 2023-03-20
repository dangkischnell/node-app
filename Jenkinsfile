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
            string(credentialsId: 'github_personal_token', variable: 'GITHUB_TOKEN'),
          ]) {
            sh '''
               rm -rf node-app-terraform
               git clone https://${GITHUB_TOKEN}@github.com/dangkischnell/node-app-terraform.git
               cd node-app-terraform
               terraform init        
               terraform apply -auto-approve -var access_key=${AWS_KEY} -var secret_key=${AWS_SECRET}
               git add terraform.tfstate
               git -c user.name="dawefd" -c user.email="ffff@gmail.com" commit -m "terraform state update from Jenkins"               
               git remote remove origin
               git remote add origin https://dangkischnell:${GITHUB_TOKEN}@github.com/dangkischnell/node-app-terraform.git
               git push https://${GITHUB_TOKEN}@github.com/dangkischnell/node-app-terraform.git master
                
            '''       
         }   
    }
  }
 } 
  environment {
    npm_config_cache = 'npm-cache'
  }
}

