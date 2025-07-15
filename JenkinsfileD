
pipeline {
    agent any

    options {
        timestamps()
    }

    parameters {
        choice(name: 'branchName', choices: ['master', 'development', 'feature'], description: 'development')
    }

    tools {
        maven "Maven-3.9.9"
    }

    triggers {
        githubPush()
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'development', url: 'https://github.com/awsprassadh/maven-webapplication-project-kkfunda.git'
            }
        }

        stage('Maven Compile') {
            steps {
                sh "mvn clean compile"
            }
        }

        stage('Maven Package') {
            steps {
                sh "mvn clean package"
            }
        }

        stage('SonarQube Report') {
            steps {
                sh "mvn sonar:sonar"
            }
        }

        stage('Deploy to Nexus') {
            steps {
                sh "mvn clean deploy"
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                sh """
                    curl -u prassadh:password \
                    --upload-file target/maven-web-application.war \
                    "http://13.232.252.38:9090/manager/text/deploy?path=/maven-web-application&update=true"
                """
            }
        }
    }

    post {
        success {
            script {
                notifyBuild(currentBuild.result)
            }
        }
        failure {
            script {
                notifyBuild(currentBuild.result)
            }
        }
    }
}

// Method must be wrapped inside script block
def notifyBuild(String buildStatus = 'STARTED') {
    buildStatus = buildStatus ?: 'SUCCESS'

    def colorCode
    def subject = "${buildStatus}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'"
    def summary = "${subject} (${env.BUILD_URL})"

    switch (buildStatus) {
        case 'STARTED':
            colorCode = '#FFFF00' // Yellow
            break
        case 'SUCCESS':
            colorCode = '#00FF00' // Green
            break
        default:
            colorCode = '#FF0000' // Red
    }

    slackSend(color: colorCode, message: summary, channel: '#qa-team')
    slackSend(color: colorCode, message: summary, channel: '#devops-team')
}
