import groovy.json.JsonSlurper

def getFtpPublishProfile(def publishProfilesJson) {
  def pubProfiles = new JsonSlurper().parseText(publishProfilesJson)
  for (p in pubProfiles)
    if (p['publishMethod'] == 'FTP')
      return [url: p.publishUrl, username: p.userName, password: p.userPWD]
}

node {
  withEnv(['AZURE_SUBSCRIPTION_ID=5993c47c-7f69-4a6a-97a5-4615a4882233',
        'AZURE_TENANT_ID=0adb040b-ca22-4ca6-9447-ab7b049a22ff', 
  'AZURE_CLIENT_ID=a64768c1-4705-4241-9097-2a7e3726d37', 'AZURE_CLIENT_SECRET=sQDXHY3plIQQp32u4-MqK72OA~v6F2v.Bo']){
    stage('init') {
      checkout scm
    }
  
    stage('build') {
      sh 'mvn clean package'
    }
  
    stage('deploy') {
      def resourceGroup = 'QuickstartJenkins-rg'
      def webAppName = 'Mywebapp1511'
      // login Azure
      withCredentials([usernamePassword(credentialsId: 'AzureServicePrincipal', passwordVariable: 'sQDXHY3plIQQp32u4-MqK72OA~v6F2v.Bo', usernameVariable: 'a64768c1-4705-4241-9097-2a7e3726d371')]) {
       sh '''
          az login --service-principal -u a64768c1-4705-4241-9097-2a7e3726d371 -p sQDXHY3plIQQp32u4-MqK72OA~v6F2v.Bo -t $AZURE_TENANT_ID
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
