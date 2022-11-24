/* pipeline 변수 설정 */
def appImage
node {
    // gitlab으로부터 소스 다운하는 stage
    stage('Checkout') {
            checkout scm   
    }

    // mvn 툴 선언하는 stage, 필자의 경우 maven 3.6.0을 사용중
    stage('Ready'){  
        sh "git status"
    }

    //dockerfile기반 빌드하는 stage ,git소스 root에 dockerfile이 있어야한다
    
    stage('Build image'){   
        script {
                    def dockerHome = tool 'docker'
                    env.PATH = "${dockerHome}/bin:${env.PATH}"
                    env.PATH = "/home/jenkins/agent/tools/org.jenkinsci.plugins.docker.commons.tools.DockerTool/docker/bin:${env.PATH}"
                }
        appImage = docker.build("kjin17/jenkinstest")
    }

    //docker image를 push하는 stage, 필자는 dockerhub에 이미지를 올렸으나 보통 private image repo를 별도 구축해서 사용하는것이 좋음
    //docker.withRegistry에 dockerhub는 앞서 설정한 dockerhub credentials의 ID이다.
    stage('Push image') {
        script {
            sh "docker login -u kjin17 -p dckr_pat_RpHbYSfSwcXYLMMn2SaozZqSayU https://registry.hub.docker.com"
            sh "docker tag kjin17/jenkinstest:latest kjin17/jenkinstest:${env.BUILD_NUMBER}"
            sh "docker push kjin17/jenkinstest:${env.BUILD_NUMBER}"
            
            /*
            docker.withRegistry('https://registry.hub.docker.com', dockerhub-id) {
                appImage.push("${env.BUILD_NUMBER}")
                appImage.push("latest")
            }
            */
        }
    }
    /*
    stage('Image Clean up') {
        script {
            sh "docker rmi kjin17/jenkinstest:latest"
        }
    }
    */
    // kubernetes에 배포하는 stage, 배포할 yaml파일(필자의 경우 test.yaml)은 jenkinsfile과 마찬가지로 git소스 root에 위치시킨다.
    // kubeconfigID에는 앞서 설정한 Kubernetes Credentials를 입력하고 'sh'는 쿠버네티스 클러스터에 원격으로 실행시킬 명령어를 기술한다.
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
                // echo ${previousTAG}
                // sed -i 's/jenkinstest:${previousTAG}/jenkinstest:${env.BUILD_NUMBER}/g' env/dev/deployment_patch.yaml
                // add new line
                cd env/dev && kustomize edit set image kjin17/jenkinstest:${BUILD_NUMBER}
                git add env/dev/kustomization.yml
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
}
