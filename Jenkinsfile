node{
    try{
    def maven_home = tool name: 'mvn3.9', type: 'maven'
    def Tomcat_IP ="192.168.40.135"
    stage('Checkout Git Code'){
        git branch: 'main', credentialsId: 'GitHub_Cred', url: 'https://github.com/maheshnemade98/student-reg-webapp.git'
    }
    stage('Build Package'){
        sh "echo ${maven_home}"
        sh "${maven_home}/bin/mvn clean package"
    }
    
    stage('Sonarquabe Scan'){
        withCredentials([string(credentialsId: 'sonarqube_token', variable: 'SonarQubeScan')]) {
            sh "${maven_home}/bin/mvn clean verify sonar:sonar -Dsonar.token=${SonarQubeScan} -Dsonar.host.ur=http://sonarqube-sonarqube-sonarqube.apps.cluster-p79ph.p79ph.sandbox924.opentlc.com"
    
           }
    }
    stage('Upload to nexus'){
        sh "${maven_home}/bin/mvn clean deploy"
    }
    stage('deply to container'){
        sshagent(['Tomcat_server_SSH']) {
    sh """  
         ssh -o StrictHostKeyChecking=no admin@${Tomcat_IP} sudo systemctl stop tomcat
         
         sleep 5
         
         ssh -o StrictHostKeyChecking=no admin@${Tomcat_IP} rm -f /opt/tomcat/webapps/student-reg-webapp.war
         
         scp -o StrictHostKeyChecking=no target/student-reg-webapp*.war admin@${Tomcat_IP}:/opt/tomcat/webapps/student-reg-webapp.war
         
         ssh -o StrictHostKeyChecking=no admin@${Tomcat_IP} sudo systemctl start tomcat
         """
        }
    }
    
    }catch(Exception e){
        currentBuild.result = 'FAILURE'
     }finally{
        def buildSatus = currentBuild.result ?: 'SUCCESS'
        def colorcode = 'good'
        if (buildSatus == "FAILURE"){
            colorcode = 'danger'
        }
        slackSend channel: 'jenkins-test-slack-notification', color: "${colorcode}", message: "Jenkins Job'${env.JOB_NAME}-${env.BUILD_NUMBER}'Please check job is ${buildSatus}"
        emailext body:  "Jenkins Job\"${env.JOB_NAME}-${env.BUILD_NUMBER}\"Please check job is ${buildSatus}", subject: "${env.JOB_NAME}-${env.BUILD_NUMBER}-${buildSatus}", to: 'mahesh.nemade20@gmail.com,mahesh.nemade08@gmail.com'
     }

     }



