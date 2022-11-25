/* Slack 연동 */

def notifySlack(STATUS, COLOR) {
    slackSend channel: '#jenkins', 
	message: STATUS+" : " + "${env.JOB_NAME}[${env.BUILD_NUMBER}] (${env.BUILD_URL})", 
	color: COLOR, tokenCredentialId: 'slack_jenkins', 
	teamDomain: 'parkkj'


/* pipeline 변수 설정 */
def appImage
node {
  try {
    notifySlack("STARTED", "#0000FF")
    // gitlab으로부터 소스 다운하는 stage
    stage('Checkout') {
            checkout scm   
    }

    stage('Ready'){  
        sh "git status"
    }
    
    stage('Build image'){   
        script {
                    def dockerHome = tool 'docker'
                    env.PATH = "${dockerHome}/bin:${env.PATH}"
                    env.PATH = "/home/jenkins/agent/tools/org.jenkinsci.plugins.docker.commons.tools.DockerTool/docker/bin:${env.PATH}"
                }
        appImage = docker.build("kjin17/jenkinstest:${env.BUILD_NUMBER}")
    }

    stage('Push image') {
        script {
            sh "docker login -u kjin17 -p dckr_pat_RpHbYSfSwcXYLMMn2SaozZqSayU https://registry.hub.docker.com"
            // sh "docker tag kjin17/jenkinstest:latest kjin17/jenkinstest:${env.BUILD_NUMBER}"
            sh "docker push kjin17/jenkinstest:${env.BUILD_NUMBER}"
            
        }
    }

    stage('Kubernetes deploy') {
        //
        
        sh("""
           #!/usr/bin/env bash
           git config --global user.name "kjin17"
           git config --global user.email "kjin17@gmail.com"
           git checkout -B main
        """)
        
        script {
            previousTAG = sh(script : 'echo `expr ${BUILD_NUMBER} - 1`', returnStdout: true).trim()
        }
        
        withCredentials([usernamePassword(credentialsId: 'github-id', usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD')]) {
            sh("""
                git config --local credential.helper "!f() { echo username=\\$GIT_USERNAME; echo password=\\$GIT_PASSWORD; }; f"
                echo ${previousTAG}
                sed -i 's/newTag: \"${previousTAG}\"/newTag: \"${env.BUILD_NUMBER}\"/g' env/dev/kustomization.yaml
                git add env/dev/kustomization.yaml
                git status
                git commit -m "update the image tag"
                git push origin HEAD:main
            """)
        }
        
        //
    }

    stage('Complete') {
        sh "echo 'The end'"
    }
    notifySlack("SUCCESS", "#00FF00")
  }
  catch(e) {
    	// 빌드 실패시
        notifySlack("FAILED", "#FF0000")
  }
}
