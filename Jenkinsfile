def buildNumber = Jenkins.instance.getItem('jenkins_beanstalks_s3').lastSuccessfulBuild.number

def COLOR_MAP = [
    'SUCCESS': 'good',
    'FAILURE': 'danger',
]
pipeline {
  agent any

  environment {
    SNAP_REPO = 'vprofile-snapshot'
    NEXUS_USER = 'admin'
    NEXUS_PASS = 'admin'
    RELEASE_REPO = 'vprofile-release'
    CENTRAL_REPO = 'vpro-maven-central'
    NEXUSIP = '10.0.0.126'
    NEXUSPORT = '8081'
    NEXUS_GRP_REPO = 'vpro-maven-group'
    NEXUS_LOGIN = 'nexuslogin'
    SONARSERVER = 'sonarserver'
    SONARSCANNER = 'sonarscanner'

    ARTIFACT_NAME = "vprofile-v${buildNumber}.war"
    AWS_S3_BUCKET = 'vprocicdbean2611'
    AWS_EB_APP_NAME = 'prod_vpro'
    AWS_EB_ENVIRONMENT = 'Prodvpro-env'
    AWS_EB_APP_VERSION = "${buildNumber}"

  }

stages{
    stage('Deploy to Stage Bean'){
            steps {
              withAWS(credentials: 'awsbeancreds', region: 'ap-northeast-1') {
                sh 'aws elasticbeanstalk create-application-version --application-name $AWS_EB_APP_NAME --version-label $AWS_EB_APP_VERSION --source-bundle S3Bucket=$AWS_S3_BUCKET,S3Key=$ARTIFACT_NAME'
                 sh 'aws elasticbeanstalk update-environment --application-name $AWS_EB_APP_NAME --environment-name $AWS_EB_ENVIRONMENT --version-label $AWS_EB_APP_VERSION'
              }
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
