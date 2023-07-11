pipeline {
    agent any
    stages {
        stage('Git Checkout') {
            steps {
                git url: 'https://github.com/sahilj713/sonarqube-jenkins-pipeline.git'    
		            echo "Code Checked-out Successfully!!";
            } 
        }
        
        stage('Package') {
            steps { 
                sh 'mvn package'    
		            echo "Maven Package Goal Executed Successfully!";
            } 
        }
        
        stage('JUnit Reports') {
            steps {
                    junit 'target/surefire-reports/*.xml'
		                echo "Publishing JUnit reports"
            }
        }
        
        stage('Jacoco Reports') {
            steps {
                  jacoco()
                  echo "Publishing Jacoco Code Coverage Reports";
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
}
}

