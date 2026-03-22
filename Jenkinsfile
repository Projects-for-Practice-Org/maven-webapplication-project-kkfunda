pipeline
{
agent any
tools
{
maven "Maven-3.9.9"
}
triggers 
    {
  githubPush()
}
parameters {
  name: 'BRANCH',
  choice choices: ['master', 'dev', 'qa', 'uat', 'f1'], 
  description: 'These are branches in Github  in my project', name: 'Branches '
}

stages{
stage('Start'){
    steps{
        script{
            slackNotify(currentBuild.result)
        }
    }
}

stage('git checkout'){
    steps{
       git branch: params['Branches '], credentialsId: 'd82d8b60-7443-4078-b374-a44e98f876cc', 
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
            slackNotify(currentBuild.result)  //Success
        }
    }
    failure{
        script{
            slackNotify(currentBuild.result)  //Failure
        }
    }
    unstable {
        script{
            slackNotify(currentBuild.result)
        }
  }
  aborted {
        script{
            slackNotify(currentBuild.result)
        }
  }

}
}


def slackNotify(String buildStatus = 'STARTED'){
        buildStatus = buildStatus ?: 'STARTED'
        //Default values
       def colorCode = "#FF0000" //Red Color
       def result = "${buildStatus}: JobName&BuildNo&BuildURL --> ${env.JOB_NAME} ${env.BUILD_NUMBER} ${env.BUILD_URL}"
       def channelName = "#devops-practice-channel"
        
        if(buildStatus == 'STARTED'){
            colorCode = "#FFFF00"  //Yellow Color
        }
        else if(buildStatus == 'SUCCESS'){
            colorCode = "#00FF00"
        } 
        else if(buildStatus == 'ABORTED' || buildStatus == 'UNSTABLE')
        {
            colorCode = "#27F5F5"
        }

     //slack Notifications can take dynamically based on branch

    if(params.BRANCH == 'dev'){
        channelName = "#devops-channel"
    }
    else if(params.BRANCH == 'qa'){
        channelName = "#qa-channel"
    }
    else if(params.BRANCH == 'uat'){
        channelName = "#uat-channel"
    }
       slackSend(color: colorCode, message: result,channel: channelName) 
}





