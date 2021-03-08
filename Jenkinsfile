import java.text.SimpleDateFormat

node {
	def hcIntegratedBuildDir = "/var/lib/jenkins/workspace/hypercloud5-integrated"

	switch(distributionType){
	    case 'integrated':
	        DisApiServer()
	        DisSingleOperator()
	        DisMultiOperator()
	        SendMail()
	        break
        case 'only-api-server':
            DisApiServer()
            break
        case 'only-single-operator':
            DisSingleOperator()
            break
        case 'only-multi-operator':
            DisMultiOperator()
            break
        default:
            break
	}

	stage('clean repo') {
        sh "sudo rm -rf ${hcIntegratedBuildDir}/*"
    }
}

void SendMail(){
    stage('Integrated (send email)') {
        def version = "${params.majorVersion}.${params.minorVersion}.${params.tinyVersion}.${params.hotfixVersion}"
        def dateFormat = new SimpleDateFormat("yyyy.MM.dd E")
        def date = new Date()
        def today = dateFormat.format(date)

        emailext (
             subject: "Hypercloud5-Integrated-Distribution v${version}",
             body:
                """
                안녕하세요. ck2-3팀 우태건입니다.
                hypercloud5 통합 정기 배포 안내 메일입니다.

                ===

                ${today}
                통합 Install Guide : https://github.com/tmax-cloud/install-hypercloud

                API-server 배포
                * HyperCloud5 api server
                * version: v${version}
                * image: docker.io/tmaxcloudck/hypercloud-api-server:v${version}
                * GitHub : https://github.com/tmax-cloud/hypercloud-api-server

                Single-operator 배포
                * HyperCloud5 single operator
                * version: v${version}
                * image: docker.io/tmaxcloudck/hypercloud-single-operator:v${version}
                * GitHub : https://github.com/tmax-cloud/hypercloud-single-operator

                Multi-operator 배포
                * HyperCloud5 multi operator
                * version: v${version}
                * image: docker.io/tmaxcloudck/hypercloud-multi-operator:v${version}
                * GitHub : https://github.com/tmax-cloud/hypercloud-multi-operator

                ===

                """,
//             to: "cqa1@tmax.co.kr;ck1@tmax.co.kr;cqa1@tmax.co.kr;chanyong_jeon@tmax.co.kr;byongjohn_han@tmax.co.kr"
             to: "taegeon_woo@tmax.co.kr",
             from: "taegeon_woo@tmax.co.kr"
        )
    }
}

void DisApiServer() {
    def gitHubBaseAddress = "github.com"
    def gitAddress = "${gitHubBaseAddress}/tmax-cloud/hypercloud-api-server.git"
    def homeDir = "/var/lib/jenkins/workspace/hypercloud5-integrated"
    def buildDir = "${homeDir}/api-server"
    def scriptHome = "${buildDir}/scripts"
    def version = "${params.majorVersion}.${params.minorVersion}.${params.tinyVersion}.${params.hotfixVersion}"
    def imageTag = "b${version}"
    def userName = "dnxorjs1"
    def userEmail = "taegeon_woo@tmax.co.kr"


    dir(buildDir){
        stage('Api-server (git pull)') {
            git branch: "${params.apiServerBranch}",
            credentialsId: '${userName}',
            url: "http://${gitAddress}"

            // git pull
            sh "git checkout ${params.apiServerBranch}"
            sh "git config --global user.name ${userName}"
            sh "git config --global user.email ${userEmail}"
            sh "git config --global credential.helper store"

            sh "git fetch --all"
            sh "git reset --hard origin/${params.apiServerBranch}"
            sh "git pull origin ${params.apiServerBranch}"
        }

        stage('Api-server (image build & push)'){
           sh "sudo docker build --tag tmaxcloudck/hypercloud-api-server:${imageTag} ."
            sh "sudo docker push tmaxcloudck/hypercloud-api-server:${imageTag}"
            sh "sudo docker rmi tmaxcloudck/hypercloud-api-server:${imageTag}"
        }

        stage('Api-server (make change log)'){
            preVersion = sh(script:"sudo git describe --tags --abbrev=0", returnStdout: true)
            preVersion = preVersion.substring(1)
            echo "preVersion of apiServer : ${preVersion}"
            sh "sudo sh ${scriptHome}/hypercloud-changelog.sh ${version} ${preVersion}"
        }

        stage('Api-server (git push)'){
            sh "git checkout ${params.apiServerBranch}"

            sh "git config --global user.name ${userName}"
            sh "git config --global user.email ${userEmail}"
            sh "git config --global credential.helper store"
            sh "git add -A"

            sh (script:'git commit -m "[Distribution] Hypercloud-api-server- ${version} " || true')
            sh "git tag v${version}"

            sh "sudo git push -u origin +${params.apiServerBranch}"
            sh "sudo git push origin v${version}"

            sh "git fetch --all"
            sh "git reset --hard origin/${params.apiServerBranch}"
            sh "git pull origin ${params.apiServerBranch}"
        }
    }
}

void DisSingleOperator() {
    def gitHubBaseAddress = "github.com"
    def gitAddress = "${gitHubBaseAddress}/tmax-cloud/hypercloud-single-operator.git"
    def homeDir = "/var/lib/jenkins/workspace/hypercloud5-integrated"
    def buildDir = "${homeDir}/single-operator"
    def scriptHome = "${buildDir}/scripts"
    def version = "${params.majorVersion}.${params.minorVersion}.${params.tinyVersion}.${params.hotfixVersion}"
    def imageTag = "b${version}"
    def userName = "dnxorjs1"
    def userEmail = "taegeon_woo@tmax.co.kr"


    dir(buildDir){
        stage('Single-operator (git pull)') {
            git branch: "${params.singleOperatorBranch}",
            credentialsId: '${userName}',
            url: "http://${gitAddress}"

            // git pull
            sh "git checkout ${params.singleOperatorBranch}"
            sh "git config --global user.name ${userName}"
            sh "git config --global user.email ${userEmail}"
            sh "git config --global credential.helper store"

            sh "git fetch --all"
            sh "git reset --hard origin/${params.singleOperatorBranch}"
            sh "git pull origin ${params.singleOperatorBranch}"

            sh '''#!/bin/bash
                export PATH=$PATH:/usr/local/go/bin
                export GO111MODULE=on
                go build -o bin/manager main.go
                '''
        }

        stage('Single-operator (make manifests)') {
            sh "sed -i 's#{imageTag}#${imageTag}#' ./config/manager/kustomization.yaml"
            sh "sudo kubectl kustomize ./config/default/ > bin/hypercloud-single-operator-v${version}.yaml"
            sh "sudo kubectl kustomize ./config/crd/ > bin/crd-v${version}.yaml"
            sh "sudo tar -zvcf bin/hypercloud-single-operator-manifests-v${version}.tar.gz bin/hypercloud-single-operator-v${version}.yaml bin/crd-v${version}.yaml"

            sh "sudo mkdir -p build/manifests/v${version}"
            sh "sudo cp bin/*v${version}.yaml build/manifests/v${version}/"
        }

        stage('Single-operator (image build & push)'){
            sh "sudo docker build --tag tmaxcloudck/hypercloud-single-operator:${imageTag} ."
            sh "sudo docker push tmaxcloudck/hypercloud-single-operator:${imageTag}"
            sh "sudo docker rmi tmaxcloudck/hypercloud-single-operator:${imageTag}"
        }

        stage('Single-operator (make change log)'){
            preVersion = sh(script:"sudo git describe --tags --abbrev=0", returnStdout: true)
            preVersion = preVersion.substring(1)
            echo "preVersion of single-operator : ${preVersion}"
            sh "sudo sh ${scriptHome}/make-changelog.sh ${version} ${preVersion}"
        }

        stage('Single-operator (git push)'){
            sh "git checkout ${params.singleOperatorBranch}"
            sh "git add -A"
            sh "git reset ./config/manager/kustomization.yaml"
            def commitMsg = "[Distribution] Release commit for hypercloud-single-operator v${version}"
            sh (script: "git commit -m \"${commitMsg}\" || true")
            sh "git tag v${version}"
            sh "sudo git push -u origin +${params.singleOperatorBranch}"
            sh "sudo git push origin v${version}"

            sh "git fetch --all"
            sh "git reset --hard origin/${params.singleOperatorBranch}"
            sh "git pull origin ${params.singleOperatorBranch}"
        }
    }
}

void DisMultiOperator() {
    def gitHubBaseAddress = "github.com"
    def gitAddress = "${gitHubBaseAddress}/tmax-cloud/hypercloud-multi-operator.git"
    def homeDir = "/var/lib/jenkins/workspace/hypercloud5-integrated"
    def buildDir = "${homeDir}/multi-operator"
    def scriptHome = "${buildDir}/scripts"
    def version = "${params.majorVersion}.${params.minorVersion}.${params.tinyVersion}.${params.hotfixVersion}"
    def imageTag = "b${version}"
    def userName = "dnxorjs1"
    def userEmail = "taegeon_woo@tmax.co.kr"

    dir(buildDir){
        stage('Multi-operator (git pull)') {
            git branch: "${params.multiOperatorBranch}",
            credentialsId: '${userName}',
            url: "http://${gitAddress}"

            // git pull
            sh "git checkout ${params.multiOperatorBranch}"
            sh "git config --global user.name ${userName}"
            sh "git config --global user.email ${userEmail}"
            sh "git config --global credential.helper store"

            sh "git fetch --all"
            sh "git reset --hard origin/${params.multiOperatorBranch}"
            sh "git pull origin ${params.multiOperatorBranch}"

            sh '''#!/bin/bash
                export PATH=$PATH:/usr/local/go/bin
                export GO111MODULE=on
                go build -o bin/manager main.go
                '''
        }

        stage('Multi-operator (make manifests)') {
            sh "sed -i 's#{imageTag}#${imageTag}#' ./config/manager/kustomization.yaml"
            sh "sudo kubectl kustomize ./config/default/ > bin/hypercloud-multi-operator-v${version}.yaml"
            sh "sudo kubectl kustomize ./config/crd/ > bin/crd-v${version}.yaml"
            sh "sudo tar -zvcf bin/hypercloud-multi-operator-manifests-v${version}.tar.gz bin/hypercloud-multi-operator-v${version}.yaml bin/crd-v${version}.yaml"

            sh "sudo mkdir -p build/manifests/v${version}"
            sh "sudo cp bin/*v${version}.yaml build/manifests/v${version}/"
        }

        stage('Multi-operator (image build & push)'){
            sh "sudo docker build --tag tmaxcloudck/hypercloud-multi-operator:${imageTag} ."
            sh "sudo docker push tmaxcloudck/hypercloud-multi-operator:${imageTag}"
            sh "sudo docker rmi tmaxcloudck/hypercloud-multi-operator:${imageTag}"
        }

        stage('Multi-operator (make change log)'){
            preVersion = sh(script:"sudo git describe --tags --abbrev=0", returnStdout: true)
            preVersion = preVersion.substring(1)
            echo "preVersion of multi-operator : ${preVersion}"
            sh "sudo sh ${scriptHome}/make-changelog.sh ${version} ${preVersion}"
        }

        stage('Multi-operator (git push)'){
            sh "git checkout ${params.multiOperatorBranch}"
            sh "git add -A"
            sh "git reset ./config/manager/kustomization.yaml"
            def commitMsg = "[Distribution] Release commit for hypercloud-multi-operator v${version}"
            sh (script: "git commit -m \"${commitMsg}\" || true")
            sh "git tag v${version}"
            sh "sudo git push -u origin +${params.multiOperatorBranch}"
            sh "sudo git push origin v${version}"

            sh "git fetch --all"
            sh "git reset --hard origin/${params.multiOperatorBranch}"
            sh "git pull origin ${params.multiOperatorBranch}"
        }
    }
}




