import groovy.json.JsonSlurper

def getFtpPublishProfile(def publishProfilesJson) {
  def pubProfiles = new JsonSlurper().parseText(publishProfilesJson)
  for (p in pubProfiles)
    if (p['publishMethod'] == 'FTP')
      return [url: p.publishUrl, username: p.userName, password: p.userPWD]
}

node {
  withEnv(['AZURE_SUBSCRIPTION_ID=e9a5817b-a6ea-44dc-b95b-bd6a9371aef1',
        'AZURE_TENANT_ID=0adb040b-ca22-4ca6-9447-ab7b049a22ff']) {
    stage('init') {
      checkout scm
    }
  
    stage('build') {
      sh 'mvn clean package'
    }
  
    stage('deploy') {
      def resourceGroup = 'QuickstartJenkins-rg'
      def webAppName = 'myserviceapp'
      // login Azure
      withCredentials([usernamePassword(credentialsId: 'd9da1349-763f-4809-b1f8-54282e69d488', passwordVariable: 'ahntr7LtJBn9_n02AtSA1WQf08sDOpBn1G', usernameVariable: 'myappservice')]) {
       sh '''
          az login --service-principal -u $3872c5f3-4c1d-4ca9-9649-8052776774fc -p $ahntr7LtJBn9_n02AtSA1WQf08sDOpBn1G -t $AZURE_TENANT_ID
          az account set -s $AZURE_SUBSCRIPTION_ID
        '''
      }
      // get publish settings
      def pubProfilesJson = sh script: "az webapp deployment list-publishing-profiles -g $resourceGroup -n $webAppName", returnStdout: true
      def ftpProfile = getFtpPublishProfile pubProfilesJson
      // upload package
      sh "curl -T target/calculator-1.0.war $ftpProfile.url/webapps/ROOT.war -u '$ftpProfile.username:$ftpProfile.password'"
      // log out
      sh 'az logout'
    }
  }
}
