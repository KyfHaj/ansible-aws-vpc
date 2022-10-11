def COLOR_MAP = [
    'SUCCESS': 'good',
    'FAILURE': 'danger',
]
pipeline {
  agent any

  environment {
    NEXUS_USER = 'admin'
    NEXUS_PASS = 'admin'
    RELEASE_REPO = 'vprofile-release'
    CENTRAL_REPO = 'vpro-maven-central'
    NEXUSIP = '10.0.0.126'
    NEXUSPORT = '8081'
    NEXUS_GRP_REPO = 'vpro-maven-group'
    NEXUS_LOGIN = 'nexuslogin'

  }
  stages {

         stage('Setup parameters') {
             steps {
                 script {
                     properties([
                         parameters([
                             string(
                                 defaultValue: '',
                                 name: 'BUILD', 
                             ),
 							               string(
                                 defaultValue: '',
                                 name: 'TIME',
                             )
                         ])
                     ])
                 }
             }
 		        }

    stage('Ansible Deploy to staging'){
        steps {
            ansiblePlaybook([
            inventory   : 'ansible/stage.inventory',
            playbook    : 'ansible/site.yml',
            installation: 'ansible',
            colorized   : true,
      credentialsId: 'applogin',
      disableHostKeyChecking: true,
            extraVars   : [
                USER: "${NEXUS_USER}",
                PASS: "${NEXUS_PASS}",
          nexusip: "${NEXUSIP}",
          reponame: "${RELEASE_REPO}",
          groupid: "QA",
          time: "${env.TIME}",
          build: "${env.BUILD}",
                artifactid: "vproapp",
          vprofile_version: "vproapp-${env.BUILD}-${env.TIME}.war"
            ]
         ])
        }
    }
}
  post {
        always {
            echo 'Slack Notifications.'
            slackSend channel: '#jenkinscicd',
                color: COLOR_MAP[currentBuild.currentResult],
                message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n More info at: ${env.BUILD_URL}"
        }
    }
}
