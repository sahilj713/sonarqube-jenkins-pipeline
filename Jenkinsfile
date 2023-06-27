// pipeline {
//     agent any
//     stages {
//         stage('Git Checkout') {
//             steps {
//                 git url: 'https://github.com/rchidana/calcwebapp.git'    
// 		            echo "Code Checked-out Successfully!!";
//             }
//         }
        
//         stage('Package') {
//             steps {
//                 bat 'mvn package'    
// 		            echo "Maven Package Goal Executed Successfully!";
//             }
//         }
        
//         stage('JUNit Reports') {
//             steps {
//                     junit 'target/surefire-reports/*.xml'
// 		                echo "Publishing JUnit reports"
//             }
//         }
        
//         stage('Jacoco Reports') {
//             steps {
//                   jacoco()
//                   echo "Publishing Jacoco Code Coverage Reports";
//             }
//         }

// 	stage('SonarQube analysis') {
//             steps {
// 		// Change this as per your Jenkins Configuration
//                 withSonarQubeEnv('SonarQube') {
//                     bat 'mvn package sonar:sonar'
//                 }
//             }
//         }

// 	stage("Quality gate") {
//             steps {
//                 waitForQualityGate abortPipeline: true
//             }
//         }
        
//     }
//     post {
        
//         success {
//             echo 'This will run only if successful'
//         }
//         failure {
//             echo 'This will run only if failed'
//         }
    
//     }
// }

pipeline {
 agent any
 //        {
 //         docker {
 //                 image 'maven'
 //                 args '-v $HOME/.m2:/root/.m2'
 //         }
 // }
 environment {
 AWS_ACCOUNT_ID="382904467012"
 AWS_DEFAULT_REGION="us-east-1" 
 IMAGE_REPO_NAME="react_app"
 // PARAM_IMAGE_TAG ="${IMAGE_TAG}"
 // IMAGE_TAG="${GIT_COMMIT}"  
 SELECTED_IMAGE_TAG = "${IMAGE_TAG}-${BRANCH_NAME}"
 BRANCH = "${BRANCH_NAME}" 
 REPOSITORY_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}"
 CLUSTER_NAME = "ecs-cluster"
 TASKDEF_NAME = "ecs_terraform_task_def"  
 SERVICE_NAME = "ecs_terraform_service"  
 }
 
 stages {
 
 stage('Cloning Git') {
 steps {
 checkout scmGit(branches: [[name: '${BRANCH_NAME}']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/sahilj713/multi-branch-cicd.git']])
 }
 }

stage('Quality Gate Status Check'){
 steps{
  script{
   withSonarQubeEnv('sonar-server'){
    sh "mvn sonar:sonar"
   }
   timeout(time: 1, unit:'HOURS'){
    def qg = waitForQualityGate()
      if (qg.status != 'OK'){
       error "Pipeline aborted due to quality gate failure: ${qg.status}"
      }
   }
   sh "mvn clean install"
  }
 }
}  
 
// Building Docker images
 stage('Building image') {
 steps{
 script {
 dockerImage = docker.build "${IMAGE_REPO_NAME}:${SELECTED_IMAGE_TAG}"  
 }
 }
 }
  
 stage('Logging into AWS ECR') {
 steps {
 script {
 sh "aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com"
 }
 }
 }
  
// Uploading Docker images into AWS ECR
 stage('Pushing to ECR') {
 steps{ 
 script {
 sh "docker tag ${IMAGE_REPO_NAME}:${SELECTED_IMAGE_TAG} ${REPOSITORY_URI}:$SELECTED_IMAGE_TAG"
 sh "docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}:${SELECTED_IMAGE_TAG}"
 }
 }
 }
  
 
 stage('manual-approval'){
  when {
        expression {
          BRANCH == 'prod' || BRANCH == 'uat'
        }
      }
      steps {
        echo 'Deploying...'
        input message: 'Do you want to deploy to production? (y/n)'
      }
 } 

 }
}
