pipeline{
  agent { label "docker"}
  parameters { string(name: 'SPECIFIC_APP_VERSION', defaultValue: '', description: '') }
  environment { 
        ANSIBLE_HOST_KEY_CHECKING = false
        APP_MAIN_VERSION = '1.0'
        ARTIFACTORY_SERVER_ID = "artifactory"
        APP_VERSION = getAppVersion("${SPECIFIC_APP_VERSION}", "${APP_MAIN_VERSION}-${BUILD_NUMBER}") 
        MAVEN_HOME = '/usr/share/maven'
  }
  stages{
    stage("build and upload"){
      // 如果不指定部署版本，则执行构建
      when {
        expression{ return params.SPECIFIC_APP_VERSION == "" }
      }
      steps{
        script{
          docker.image('jenkins-docker-maven:3.6.1-jdk8').inside("--network 1-cd-platform_cd-in-practice -v /root/.m2:/root/.m2") {
              sh """
                mvn versions:set -DnewVersion=${APP_VERSION}
                mvn clean test install -U
                mvn deploy
              """
          }
        }
      }
    }
     stage("deploy app"){
       steps{
         script{
           docker.image('williamyeh/ansible:centos7').inside("--network 1-cd-platform_cd-in-practice") {
             checkout([$class: 'GitSCM', branches: [[name: "master"]], doGenerateSubmoduleConfigurations: false,
                       extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: "env-conf"]], submoduleCfg: [],
                       userRemoteConfigs: [[url: "https://github.com/cd-in-practice/1-env-conf.git"]]])
             sh "ls -al"
             sh """
                ansible-playbook --syntax-check deploy/playbook.yaml -i env-conf/dev
                ansible-playbook deploy/playbook.yaml -i env-conf/dev --extra-vars '{"app_version": "${APP_VERSION}"}'
             """
           }
         }
       }
     }
  }
}

def getAppVersion(specificAppVersion, defaultAppVersion){
  return specificAppVersion == null || specificAppVersion.trim() == "" ? defaultAppVersion.trim() : defaultAppVersion.trim()
}