


node
{

   echo "git branch name: ${env.BRANCH_NAME}"
   echo "build number is: ${env.BUILD_NUMBER}"
   echo "node name is: ${env.NODE_NAME}"


     // /var/lib/jenkins/tools/hudson.tasks.Maven_MavenInstallation/Maven-3.9.9/bin
    def mavenHome=tool name: "Maven-3.9.9"
    try
    {

    stage('git checkout') 
    {   // stage starting

       notifyBuild('STARTED')
       git branch: 'development ', url: 'https://github.com/awsprassadh/maven-webapplication-project-kkfunda.git'

    }   // stage ending

    stage('COMPILE')
    { 
      sh "${mavenHome}/bin/mvn clean compile"
    }
    stage('Build')
    {
      sh "${mavenHome}/bin/mvn clean package"
    }
    stage('SQ REPORT')
    {
      sh "${mavenHome}/bin/mvn sonar:sonar"
    }
    stage('Upload Artifact to Nexus')
    {
      sh "${mavenHome}/bin/mvn clean deploy"
    }
    stage('Deploy to Tomcat') 
    {

      echo "Deploying WAR file using curl..."

      sh """
            curl -u prassadh:password \
            --upload-file /var/lib/jenkins/workspace/jio-scripted-way-pl/target/maven-web-application.war \
            "http://http://13.204.101.111:9090/manager/text/deploy?path=/maven-web-application&update=true"
        """
          
    }

    }  //try ending

    catch (e) {
   
       currentBuild.result = "FAILED"

  } finally {
    // Success or failure, always send notifications
    notifyBuild(currentBuild.result)
  }
  
} // node ending


def notifyBuild(String buildStatus = 'STARTED') {
  // build status of null means successful
  buildStatus =  buildStatus ?: 'SUCCESS'

  // Default values
  def colorName = 'RED'
  def colorCode = '#FF0000'
  def subject = "${buildStatus}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'"
  def summary = "${subject} (${env.BUILD_URL})"

  // Override default values based on build status
  if (buildStatus == 'STARTED') {
    color = 'YELLOW'
    colorCode = '#FFFF00'
  } else if (buildStatus == 'SUCCESS') {
    color = 'GREEN'
    colorCode = '#00FF00'
  } else {
    color = 'RED'
    colorCode = '#FF0000'
  }

  // Send notifications
  slackSend (color: colorCode, message: summary, channel: '#qa-team')
  slackSend (color: colorCode, message: summary, channel: '#devops-team')
}
