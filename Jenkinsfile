def COLOR_MAP = [
    'SUCCESS': 'good',
    'FAILURE': 'danger',
]
pipeline {
  agent any
  tools {
    maven "MAVEN3"
    jdk "OracleJDK8"
  }
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

    ARTIFACT_NAME = "vprofile-v${BUILD_ID}.war"
    AWS_S3_BUCKET = 'vprocicdbean2611'
    AWS_EB_APP_NAME = 'vproapp'
    AWS_EB_ENVIRONMENT = 'Vproapp-env'
    AWS_EB_APP_VERSION = "${BUILD_ID}"

  }
  stages {
    stage('Build') {
      steps {
        sh 'mvn -s settings.xml -DskipTests install'
      }
      post {
        success {
        echo "Now Archiving."
        archiveArtifacts artifacts: '**/*.war'
                }
            }
    }
    stage('Test') {
      steps {
        sh 'mvn -s settings.xml test'
      }
    }

    stage('Deploy to Stage Bean'){
            steps {
              withAWS(credentials: 'awsbeancreds', region: 'ap-northeast-1') {
                 sh 'aws s3 cp ./target/vprofile-v2.war s3://$AWS_S3_BUCKET/$ARTIFACT_NAME'
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
