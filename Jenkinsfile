pipeline {
    agent any
    environment {
        AWS_ACCOUNT_ID="767397705569"
        AWS_DEFAULT_REGION="ap-southeast-2"
        IMAGE_REPO_NAME="demo"
        IMAGE_TAG="BUILD_NUMBER"
        REPOSITORY_URI = "767397705569.dkr.ecr.ap-southeast-2.amazonaws.com/demo"
    }
   
    stages {
        
         stage('Logging into AWS ECR') {
            steps {
                script {
                sh """aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com"""
                }
                 
            }
        }
        
        stage('Cloning Git') {
             steps {
 checkout([$class: 'GitSCM', branches: [[name: '*/main']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: '', url: 'https://github.com/gitkul/jenkins-cicd.git']]])  
                }
        }
  
    // Building Docker images
    stage('Building image') {
      steps{
        script {
          dockerImage = docker.build "${IMAGE_REPO_NAME}:${IMAGE_TAG}"
        }
      }
    }
   
    // Uploading Docker images into AWS ECR
    stage('Pushing to ECR') {
     steps{  
         script {
                sh """docker tag ${IMAGE_REPO_NAME}:${IMAGE_TAG} ${REPOSITORY_URI}:$IMAGE_TAG"""
                sh """docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}:${IMAGE_TAG}"""
         }
        }
      }

    stage('Update Values File for helm deployment') {
        environment {
            GIT_REPO_NAME = "jenkins-cicd"
            GIT_USER_NAME = "gitkul"
        }
      steps {
             withCredentials([string(credentialsId: 'github-token', variable: 'GITHUB_TOKEN')]) {
                sh '''
                    git pull https://github.com/gitkul/jenkins-cicd.git
                    git config user.email "die4kuldeep@yahoo.com"
                    git config user.name "gitkul"
                    BUILD_NUMBER=${BUILD_NUMBER}
                    chmod -R 777 /var/lib/jenkins
                    cp -rvf deployment.yml deploy/deployment.yml
                    

                    sed -i "s/imagetag/demo:${BUILD_NUMBER}/" deploy/deployment.yml

                    git add deploy/deployment.yml
                    git commit -m "Updated  with build image number ${BUILD_NUMBER}"


                    git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
                '''
           }
         }

        }




    }
}

