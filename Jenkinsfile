node('maven') {

    stage('checkout') {
       echo "checking out source"
       echo "Build: ${BUILD_ID}"
       checkout scm
    }

    stage('code quality check') {
            SONARQUBE_PWD = sh (
             script: 'oc env dc/sonarqube --list | awk  -F  "=" \'/SONARQUBE_ADMINPW/{print $2}\'',
             returnStdout: true
              ).trim()
           echo "SONARQUBE_PWD: ${SONARQUBE_PWD}"

           SONARQUBE_URL = sh (
               script: 'oc get routes -o wide --no-headers | awk \'/sonarqube/{ print match($0,/edge/) ?  "https://"$2 : "http://"$2 }\'',
               returnStdout: true
                  ).trim()
           echo "SONARQUBE_URL: ${SONARQUBE_URL}"

           dir('sonar-runner') {
            sh returnStdout: true, script: './gradlew sonarqube -Dsonar.host.url=${SONARQUBE_URL} -Dsonar.verbose=true --stacktrace --info  -Dsonar.sources=..'
        }
    }
}

stage ('Compile microservice')
{
	openshiftBuild(buildConfig: 'document-microservice', showBuildLogs: 'true')
	openshiftTag destStream: 'document-microservice', verbose: 'true', destTag: '$BUILD_ID', srcStream: 'document-microservice', srcTag: 'latest'
	openshiftTag destStream: 'document-microservice', verbose: 'true', destTag: 'dev', srcStream: 'document-microservice', srcTag: 'latest'
}
	
stage ('Compile front end')
{
	openshiftBuild(buildConfig: 'dmod', showBuildLogs: 'true')
	openshiftTag destStream: 'dmod', verbose: 'true', destTag: '$BUILD_ID', srcStream: 'dmod', srcTag: 'latest'
	openshiftTag destStream: 'dmod', verbose: 'true', destTag: 'dev', srcStream: 'dmod', srcTag: 'latest'
}

node('maven'){
   stage('validation') {
          dir('functional-tests'){
                // sh './gradlew --debug --stacktrace phantomJsTest'
      }
   }
}

stage('deploy-test') {
  input "Deploy to test?"
  openshiftTag destStream: 'gwells', verbose: 'true', destTag: 'test', srcStream: 'gwells', srcTag: '$BUILD_ID'
}

stage('deploy-prod') {
  input "Deploy to prod?"
  openshiftTag destStream: 'gwells', verbose: 'true', destTag: 'prod', srcStream: 'gwells', srcTag: '$BUILD_ID'
}


