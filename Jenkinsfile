node{
    def mavenHome = tool name: 'Maven-3.9.9'
    try{
    stage('Git Checkout')
    {
        notifyBuild('START')
        git branch: 'dev', credentialsId: 'd82d8b60-7443-4078-b374-a44e98f876cc', 
        url: 'https://github.com/Projects-for-Practice-Org/maven-webapplication-project-kkfunda.git'
    }
    stage('Build Artifactory')
    {
     sh "${mavenHome}/bin/mvn clean install"
    }
    stage('SonarQube Report')
    {
        sh "${mavenHome}/bin/mvn sonar:sonar"
    }
    stage('Nexus Artifactory')
    {
        sh "${mavenHome}/bin/mvn deploy"
    }
    stage('Deploy into Tomcat')
    {
        sh"""
        curl -u appaji:appaji \
        --upload-file /var/lib/jenkins/workspace/scripted-piplines/target/maven-web-application.war \
        "http://54.92.177.225:8080/manager/text/deploy?path=/maven-web-application&update=true"
        """
        
    }
}
    
    catch(e){
        currentBuild.status = "FAILED"
        throw e
    }
    finally{
        notifyBuild(currentBuild.result)
    }
}


def notifyBuild(String buildStatus = 'STARTED')
 {
    buildStatus = buildStatus ?: 'SUCCESS'
    
    //Default Values
    colorCode = '#FF0000' //Red Color
    def jobStatus = "${buildStatus} : Job Name and Build Number is: ${env.JOB_NAME} ${env.BUILD_NUMBER}"
    def summary = "${jobStatus} ${env.BUILD_URL}"
    
    if(buildStatus == 'START'){
        colorCode = "#FFFF00"
    }else if (buildStatus == 'SUCCESS'){
        colorCode = "#00FF00"
    }
    slackSend(color: colorCode, message: summary, channel: '#devops-practice-channel')
  }
