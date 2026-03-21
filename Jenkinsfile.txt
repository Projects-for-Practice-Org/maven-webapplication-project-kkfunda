
pipeline{
agent any
tools{
maven "Maven-3.9.9"
}

stages{
stage('Start'){
    steps{
        script{
            notifyBuild('STARTED')
        }
    }
}

stage('git checkout'){
    steps{
       git branch: 'dev', credentialsId: 'd82d8b60-7443-4078-b374-a44e98f876cc', 
       url: 'https://github.com/Projects-for-Practice-Org/maven-webapplication-project-kkfunda.git'
    }
}
stage('Build Artifactory'){
    steps{
        sh "mvn clean package"
    }
}
stage('SonarQube Report'){
    steps{
        sh "mvn sonar:sonar"
    }
}
stage('Artifactory store into Nexus Repository'){
    steps{
        sh "mvn deploy"
    }
}
stage('Deploy into Tomcat'){
    steps{
        sh """
        curl -u appaji:appaji \
        --upload-file /var/lib/jenkins/workspace/Declarative-Job/target/maven-web-application.war \
        "http://98.81.255.43:8080/manager/text/deploy?path=/maven-web-application&update=true"
        """
    }
}
}

post{
    success{
        script{
            notifyBuild('SUCCESS')
        }
    }
    failure{
        script{
            notifyBuild('FAILURE')
        }
    }
}
}


def notifyBuild(String buildStatus = 'STARTED'){
        buildStatus = buildStatus ?: 'SUCCESS'
        //Default values
        colorCode = "#FF0000" //Red Color
        result = "${buildStatus}: JobName&BuildNo&BuildURL --> ${env.JOB_NAME} ${env.BUILD_NUMBER} ${env.BUILD_URL}"
        
        if(buildStatus == 'STARTED'){
            colorCode = "#FFFF00"  //Yellow Color
        }
        else if(buildStatus == 'SUCCESS'){
            colorCode = "#00FF00"
        }                
       slackSend(color: colorCode, message: result,channel: '#devops-channel') 
    
}
