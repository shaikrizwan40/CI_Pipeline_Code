pipeline{

/*
  Tools Required            : Docker, SonarQube Server Setuo, AWS CLI Tool
  Keep Maven Name           : Maven
  SonarQube Setting Name as : Sonarqube
  AWS Credential name as    : AWS_Credentials
  Git credential name as    : Git_Credentials
*/

  
agent any 
tools
{
     maven 'Maven'
}  

stages{
    stage("Clean_WS"){
           steps
           {
              cleanWs()
           }}
      
    stage('Git Checkout'){
          steps{
             git credentialsId: 'Git_Credentials', url: 'https://github.com/shaikrizwan40/springboot-maven-micro' 
              }}
    
    stage("Unit testing"){
          steps{
             sh 'mvn test'  
          }}
  
    stage('Project_Build'){
          steps{
             sh 'mvn verify'      }
          post{
              always{
                  junit 'target/surefire-reports/*.xml'
                  jacoco execPattern: 'target/jacoco.exec',
                  classPattern: 'target/classes',
                  sourcePattern: 'src/main/java'
              }}}
     
      stage('Static_Code_Analysis'){
            steps{
                  withSonarQubeEnv(installationName:'Sonarqube'){
                  sh 'mvn sonar:sonar'
                  }
             //timeout(time: 5, unit: 'MINUTES') {
            //script {
            //def qg = waitForQualityGate() // polls Sonar using task ID from workspace
            //if (qg.status != 'OK') {
          //  error "Failed: ${qg.status}"
        //}
    //}
//}
      }}
      
      stage("Build_Docker Image"){
          steps{
              script{
               def Registry='448049787147.dkr.ecr.us-east-1.amazonaws.com/docker_charts'
               def Image_Name="rizwan_spring"
               env.Tag_Name="${Registry}/${Image_Name}:${env.BUILD_NUMBER}"
               sh "docker build -t ${env.Tag_Name} ."
          }}
      }
  
      stage('Push_Image_To_ECR'){
          steps {
            withCredentials([usernamePassword(
            credentialsId: 'AWS_Credentials',
            usernameVariable: 'AWS_ACCESS_KEY_ID',     
            passwordVariable: 'AWS_SECRET_ACCESS_KEY'  
                )]) {
              
          script {
              sh """

                  aws configure set aws_access_key_id $AWS_ACCESS_KEY_ID
                  aws configure set aws_secret_access_key $AWS_SECRET_ACCESS_KEY
                  aws configure set default.region us-east-1

                  # Authenticate Docker with ECR
                  aws ecr get-login-password | docker login --username AWS --password-stdin 448049787147.dkr.ecr.us-east-1.amazonaws.com
           
            # Push the Docker image
              docker push ${env.Tag_Name}
              """ 
                 
                          
              }}}}}}
                      
                  
