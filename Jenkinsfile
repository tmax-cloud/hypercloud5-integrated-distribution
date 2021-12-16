import java.text.SimpleDateFormat

node {
	def hcIntegratedBuildDir = "/var/lib/jenkins/workspace/hypercloud5-integrated"

	switch(distributionType){
        case 'integrated':
            DisApiServer()
            DisSingleOperator()
            DisMultiOperator()
            DisMultiAgent()
            DisTFCOperator()
            UploadCRD()
            SendMail()
            break
        case 'only-api-server':
            DisApiServer()
            break
        case 'only-single-operator':
            DisSingleOperator()
            UploadCRD()
            break
        case 'only-multi-operator':
            DisMultiOperator()
            UploadCRD()
            break
        case 'only-multi-agent':
            DisMultiAgent()
            break
        case 'only-tfc-operator':
            DisTFCOperator()
            break
        case 'test':
            TestDisSingleOperator()
            MakeKeyMappingFile()
            TestUploadCRD()
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
                안녕하세요. ck1-3팀 이승원입니다.
                hypercloud5 통합 정기 배포 안내 메일입니다.

                ===

                ${today}
                통합 Install Guide : https://github.com/tmax-cloud/install-hypercloud

                API-server 배포
                * HyperCloud5 api server
                * version: b${version}
                * image: docker.io/tmaxcloudck/hypercloud-api-server:b${version}
                * GitHub : https://github.com/tmax-cloud/hypercloud-api-server
                * ChangeLog : https://github.com/tmax-cloud/hypercloud-api-server/blob/master/CHANGELOG.md

                Single-operator 배포
                * HyperCloud5 single operator
                * version: b${version}
                * image: docker.io/tmaxcloudck/hypercloud-single-operator:b${version}
                * GitHub : https://github.com/tmax-cloud/hypercloud-single-operator
                * ChangeLog : https://github.com/tmax-cloud/hypercloud-single-operator/blob/main/CHANGELOG.md
                * CRD : https://github.com/tmax-cloud/hypercloud-single-operator/tree/main/build/manifests/v${version}

                Multi-operator 배포
                * HyperCloud5 multi operator
                * version: b${version}
                * image: docker.io/tmaxcloudck/hypercloud-multi-operator:b${version}
                * GitHub : https://github.com/tmax-cloud/hypercloud-multi-operator
                * ChangeLog : https://github.com/tmax-cloud/hypercloud-multi-operator/blob/master/CHANGELOG.md
                * CRD : https://github.com/tmax-cloud/hypercloud-multi-operator/tree/master/build/manifests/v${version}
		
                Multi-agent 배포
                * HyperCloud5 multi agent
                * version: b${version}
                * image: docker.io/tmaxcloudck/hypercloud-multi-agent:b${version}
                * GitHub : https://github.com/tmax-cloud/hypercloud-multi-agent/tree/dev/cho
                * ChangeLog : https://github.com/tmax-cloud/hypercloud-multi-agent/blob/dev/cho/CHANGELOG.md
		
                tfc-operator 배포
                * Terraform Apply Claim Operator 배포
                * version: b${version}
                * image: docker.io/tmaxcloudck/tfc-operator:b${version}
                * GitHub : https://github.com/tmax-cloud/tfc-operator
                * ChangeLog : https://github.com/tmax-cloud/tfc-operator/blob/main/CHANGELOG.md
                * CRD: https://github.com/tmax-cloud/tfc-operator/tree/main/manifests/v${version}

                ===

                """,
             to: "cqa1@tmax.co.kr;ck1@tmax.co.kr;kyunghoon_min@tmax.co.kr;byongjohn_han@tmax.co.kr;",
             from: "seungwon_lee@tmax.co.kr"
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
    def userName = "aldlfkahs"
    def userEmail = "seungwon_lee@tmax.co.kr"
    def githubUserToken = "${params.githubUserToken}"

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

            def commitMsg = "[Distribution] Release commit for Hypercloud-api-server- v${version}"
            sh (script: "git commit -m \"${commitMsg}\" || true")
            sh "git tag v${version}"

            sh "git remote set-url origin https://${githubUserToken}@github.com/tmax-cloud/hypercloud-api-server.git"
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
    def userName = "aldlfkahs"
    def userEmail = "seungwon_lee@tmax.co.kr"


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
            sh "sudo cp bin/hypercloud-single-operator-v${version}.yaml ${homeDir}/"
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
            sh "git remote set-url origin https://${githubUserToken}@github.com/tmax-cloud/hypercloud-single-operator.git"
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
    def pkgVersion = "v0.${params.majorVersion}.${params.minorVersion}-b${params.tinyVersion}f${params.hotfixVersion}"
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
            sh "sudo cp bin/hypercloud-multi-operator-v${version}.yaml ${homeDir}/"
            sh "sudo cp ./config/capi-template/capi-aws-template.yaml build/manifests/v${version}/capi-aws-template-v${version}.yaml"
            sh "sudo cp ./config/capi-template/capi-vsphere-template.yaml build/manifests/v${version}/capi-vsphere-template-v${version}.yaml"
            sh "sudo cp build/manifests/v${version}/capi-*-template-v${version}.yaml ${homeDir}/"            
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
            def commitMsg = "[Distribution] Release commit for Hypercloud-multi-operator v${version}"
            sh (script: "git commit -m \"${commitMsg}\" || true")
            sh "git tag v${version}"
            sh "git tag ${pkgVersion}"
            sh "git remote set-url origin https://${githubUserToken}@github.com/tmax-cloud/hypercloud-multi-operator.git"
            sh "sudo git push -u origin +${params.multiOperatorBranch}"
            sh "sudo git push origin v${version}"
            sh "sudo git push origin ${pkgVersion}"

            sh "git fetch --all"
            sh "git reset --hard origin/${params.multiOperatorBranch}"
            sh "git pull origin ${params.multiOperatorBranch}"
        }
    }
}

void DisMultiAgent() {
    def gitHubBaseAddress = "github.com"
    def gitAddress = "${gitHubBaseAddress}/tmax-cloud/hypercloud-multi-agent.git"
    def homeDir = "/var/lib/jenkins/workspace/hypercloud5-integrated"
    def buildDir = "${homeDir}/multi-agent"
    def scriptHome = "${buildDir}/scripts"
    def version = "${params.majorVersion}.${params.minorVersion}.${params.tinyVersion}.${params.hotfixVersion}"
    def imageTag = "b${version}"
    def userName = "dnxorjs1"
    def userEmail = "taegeon_woo@tmax.co.kr"


    dir(buildDir){
        stage('Multi-agent (git pull)') {
            git branch: "${params.multiAgentBranch}",
            credentialsId: '${userName}',
            url: "http://${gitAddress}"

            // git pull
            sh "git checkout ${params.multiAgentBranch}"
            sh "git config --global user.name ${userName}"
            sh "git config --global user.email ${userEmail}"
            sh "git config --global credential.helper store"

            sh "git fetch --all"
            sh "git reset --hard origin/${params.multiAgentBranch}"
            sh "git pull origin ${params.multiAgentBranch}"
        }

        stage('Multi-agent (image build & push)'){
            sh "sudo make docker-build IMG=tmaxcloudck/hypercloud-multi-agent:${imageTag} ."
            sh "sudo make docker-push IMG=tmaxcloudck/hypercloud-multi-agent:${imageTag}"
            sh "sudo docker rmi tmaxcloudck/hypercloud-multi-agent:${imageTag}"
        }

        stage('Multi-agent (make change log)'){
            preVersion = sh(script:"sudo git describe --tags --abbrev=0", returnStdout: true)
            preVersion = preVersion.substring(1)
            echo "preVersion of multiAgent : ${preVersion}"
            sh "sudo sh ${scriptHome}/make-changelog.sh ${version} ${preVersion}"
        }

        stage('Multi-agent (git push)'){
            sh "git checkout ${params.multiAgentBranch}"

            sh "git config --global user.name ${userName}"
            sh "git config --global user.email ${userEmail}"
            sh "git config --global credential.helper store"
            sh "git add -A"

            def commitMsg = "[Distribution] Release commit for Hypercloud-multi-agent v${version}"
            sh (script: "git commit -m \"${commitMsg}\" || true")		
            sh "git tag v${version}"

            sh "git remote set-url origin https://${githubUserToken}@github.com/tmax-cloud/hypercloud-multi-agent.git"
            sh "sudo git push -u origin +${params.multiAgentBranch}"
            sh "sudo git push origin v${version}"

            sh "git fetch --all"
            sh "git reset --hard origin/${params.multiAgentBranch}"
            sh "git pull origin ${params.multiAgentBranch}"
        }
    }
}

void DisTFCOperator() {
    def gitHubBaseAddress = "github.com"
    def gitAddress = "${gitHubBaseAddress}/tmax-cloud/tfc-operator.git"
    def homeDir = "/var/lib/jenkins/workspace/hypercloud5-integrated"
    def buildDir = "${homeDir}/tfc-operator"
    def scriptHome = "${buildDir}/scripts"
    def version = "${params.majorVersion}.${params.minorVersion}.${params.tinyVersion}.${params.hotfixVersion}"
    def imageTag = "b${version}"
    def userName = "gyeongyeol-choi"
    def userEmail = "gyeongyeol_choi@tmax.co.kr"


    dir(buildDir){
        stage('tfc-operator (git pull)') {
            git branch: "${params.tfcOperatorBranch}",
            credentialsId: '${userName}',
            url: "http://${gitAddress}"

            // git pull
            sh "git checkout ${params.tfcOperatorBranch}"
            sh "git config --global user.name ${userName}"
            sh "git config --global user.email ${userEmail}"
            sh "git config --global credential.helper store"

            sh "git fetch --all"
            sh "git reset --hard origin/${params.tfcOperatorBranch}"
            sh "git pull origin ${params.tfcOperatorBranch}"

            sh '''#!/bin/bash
                export PATH=$PATH:/usr/local/go/bin
                export GO111MODULE=on
                go build -o bin/manager main.go
                '''
        }

        stage('tfc-operator (make manifests)') {
            sh "sed -i 's#{imageTag}#${imageTag}#' ./config/manager/kustomization.yaml"
            sh "sudo kubectl kustomize ./config/default/ > bin/tfc-operator-v${version}.yaml"
            sh "sudo kubectl kustomize ./config/crd/ > bin/crd-v${version}.yaml"
            sh "sudo tar -zvcf bin/tfc-operator-manifests-v${version}.tar.gz bin/tfc-operator-v${version}.yaml bin/crd-v${version}.yaml"

            sh "sudo mkdir -p build/manifests/v${version}"
            sh "sudo cp bin/*v${version}.yaml build/manifests/v${version}/"
        }

        stage('tfc-operator (image build & push)'){
            sh "sudo docker build --tag tmaxcloudck/tfc-operator:${imageTag} ."
            sh "sudo docker push tmaxcloudck/tfc-operator:${imageTag}"
            sh "sudo docker rmi tmaxcloudck/tfc-operator:${imageTag}"
        }

        stage('tfc-operator (make change log)'){
            preVersion = sh(script:"sudo git describe --tags --abbrev=0", returnStdout: true)
            preVersion = preVersion.substring(1)
            echo "preVersion of tfc-operator : ${preVersion}"
            sh "sudo sh ${scriptHome}/make-changelog.sh ${version} ${preVersion}"
        }

        stage('tfc-operator (git push)'){
            sh "git checkout ${params.tfcOperatorBranch}"
            sh "git add -A"
            sh "git reset ./config/manager/kustomization.yaml"
            def commitMsg = "[Distribution] Release commit for tfc-operator v${version}"
            sh (script: "git commit -m \"${commitMsg}\" || true")
            sh "git tag v${version}"
            sh "git remote set-url origin https://${githubUserToken}@github.com/tmax-cloud/tfc-operator.git"
            sh "sudo git push -u origin +${params.tfcOperatorBranch}"
            sh "sudo git push origin v${version}"

            sh "git fetch --all"
            sh "git reset --hard origin/${params.tfcOperatorBranch}"
            sh "git pull origin ${params.tfcOperatorBranch}"
        }
    }
}

void UploadCRD() {
    def gitHubBaseAddress = "github.com"
    def gitAddress = "${gitHubBaseAddress}/tmax-cloud/install-hypercloud.git"
    def homeDir = "/var/lib/jenkins/workspace/hypercloud5-integrated"
    def buildDir = "${homeDir}/install-hypercloud"
    def version = "${params.majorVersion}.${params.minorVersion}.${params.tinyVersion}.${params.hotfixVersion}"
    def userName = "aldlfkahs"
    def userEmail = "seungwon_lee@tmax.co.kr"


    dir(buildDir){
        stage('Install-hypercloud (git pull)') {
            git branch: "${params.installHypercloudBranch}",
            credentialsId: '${userName}',
            url: "http://${gitAddress}"

            // git pull
            sh "git checkout ${params.installHypercloudBranch}"
            sh "git config --global user.name ${userName}"
            sh "git config --global user.email ${userEmail}"
            sh "git config --global credential.helper store"

            sh "git fetch --all"
            sh "git reset --hard origin/${params.installHypercloudBranch}"
            sh "git pull origin ${params.installHypercloudBranch}"
        }

        stage('Install-hypercloud (upload CRD yaml)'){
            if (fileExists("${homeDir}/hypercloud-single-operator-v${version}.yaml")){
                sh "cp ${homeDir}/hypercloud-single-operator-v${version}.yaml hypercloud-single-operator/"
            }
            if (fileExists("${homeDir}/hypercloud-multi-operator-v${version}.yaml")){
                sh "cp ${homeDir}/hypercloud-multi-operator-v${version}.yaml hypercloud-multi-operator/"
            }
            if (fileExists("${homeDir}/capi-aws-template-v${version}.yaml")){
                sh "cp ${homeDir}/capi-aws-template-v${version}.yaml hypercloud-multi-operator/"
            }
            if (fileExists("${homeDir}/capi-vsphere-template-v${version}.yaml")){
                sh "cp ${homeDir}/capi-vsphere-template-v${version}.yaml hypercloud-multi-operator/"
            }

            sh "rm -f ${homeDir}/hypercloud-single-operator-v${version}.yaml ${homeDir}/hypercloud-multi-operator-v${version}.yaml"
        }

        stage('Install-hypercloud (git push)'){
            sh "git checkout ${params.installHypercloudBranch}"

            sh "git config --global user.name ${userName}"
            sh "git config --global user.email ${userEmail}"
            sh "git config --global credential.helper store"
            sh "git add -A"

            def commitMsg = "[Distribution] Upload Operator CRD yaml - v${version}"
            sh (script: "git commit -m \"${commitMsg}\" || true")            

            sh "git remote set-url origin https://${githubUserToken}@github.com/tmax-cloud/install-hypercloud.git"
            sh "sudo git push -u origin +${params.installHypercloudBranch}"

            sh "git fetch --all"
            sh "git reset --hard origin/${params.installHypercloudBranch}"
            sh "git pull origin ${params.installHypercloudBranch}"
        }
    }   
}

void TestDisSingleOperator() {
    def gitHubBaseAddress = "github.com"
    def gitAddress = "${gitHubBaseAddress}/tmax-cloud/hypercloud-single-operator.git"
    def homeDir = "/var/lib/jenkins/workspace/hypercloud5-integrated"
    def buildDir = "${homeDir}/single-operator"
    def scriptHome = "${buildDir}/scripts"
    def version = "${params.majorVersion}.${params.minorVersion}.${params.tinyVersion}.${params.hotfixVersion}"
    def imageTag = "b${version}"
    def userName = "aldlfkahs"
    def userEmail = "seungwon_lee@tmax.co.kr"


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
            sh "sudo kubectl kustomize ./config/crd/ > bin/hypercloud-single-operator-crd-v${version}.yaml"
            sh "sudo sed -i 's#\$(CERTIFICATE_NAMESPACE)#hypercloud5-system#g' bin/hypercloud-single-operator*v${version}.yaml"
            sh "sudo sed -i 's#\$(CERTIFICATE_NAME)#hypercloud-single-operator-serving-cert#g' bin/hypercloud-single-operator*v${version}.yaml"
            sh "sudo tar -zvcf bin/hypercloud-single-operator-manifests-v${version}.tar.gz bin/hypercloud-single-operator-v${version}.yaml bin/hypercloud-single-operator*v${version}.yaml"

            sh "sudo mkdir -p build/manifests/v${version}"
            sh "sudo cp bin/*v${version}.yaml build/manifests/v${version}/"
            // make directory if not exists
            if(!fileExists("${homeDir}/convert/")){
                sh "mkdir $homeDir/convert"
            }
            sh "sudo cp bin/hypercloud-single-operator-crd-v${version}.yaml ${homeDir}/convert/" // 폴더가 없으면 만들어주나?
            sh "sudo cp bin/hypercloud-single-operator-v${version}.yaml ${homeDir}/"
        }
    }
}

void MakeKeyMappingFile() {
    def gitHubBaseAddress = "github.com"
    def gitAddress = "${gitHubBaseAddress}/tmax-cloud/schema-converter.git"
    def homeDir = "/var/lib/jenkins/workspace/hypercloud5-integrated"
    def buildDir = "${homeDir}/schema-converter"
    def userName = "aldlfkahs"
    def userEmail = "seungwon_lee@tmax.co.kr"
    //def GOOGLE_APPLICATION_CREDENTIALS = "/var/lib/jenkins/workspace/hypercloud5-integrated/credential/gcp-credential.json"

    dir(buildDir){
        stage('schema-converter (git pull)') {
            git branch: "main",
            credentialsId: '${userName}',
            url: "http://${gitAddress}"

            // git pull
            sh "git checkout main"
            sh "git config --global user.name ${userName}"
            sh "git config --global user.email ${userEmail}"
            sh "git config --global credential.helper store"

            sh "git fetch --all"
            sh "git reset --hard origin/main"
            sh "git pull origin main"
        }

        stage('schema-converter (sed file)') {
            sh "sudo sed -i 's#C:\\\\\\\\cicd-crd\\\\\\\\#'$homeDir'/convert#g' ./schema-converter/src/main/java/com/tmax/ck/main/Main.java"
            sh "sudo sed -i 's#String outputDir = rootDir + System.currentTimeMillis() + \"\\\\\\\\\"#String outputDir = \"'$homeDir'/convert/result/\"#g' ./schema-converter/src/main/java/com/tmax/ck/main/Main.java"

            if ("${params.translateCRD}" == 'true') {
                sh "sudo sed -i 's#autoTranslation = false#autoTranslation = true#g' ./schema-converter/src/main/java/com/tmax/ck/main/Main.java"
            }
        }

        stage('schema-converter (convert CRD yaml)') {
            dir("${buildDir}/schema-converter"){
                sh "export GOOGLE_APPLICATION_CREDENTIALS=/var/lib/jenkins/workspace/hypercloud5-integrated/credential/gcp-credential.json"
                sh "chmod +x gradlew"
                sh "./gradlew"
                sh "./gradlew run"
            }
        }
    }
}

void TestUploadCRD() {
    def gitHubBaseAddress = "github.com"
    def gitAddress = "${gitHubBaseAddress}/tmax-cloud/install-hypercloud.git"
    def homeDir = "/var/lib/jenkins/workspace/hypercloud5-integrated"
    def buildDir = "${homeDir}/install-hypercloud"
    def version = "${params.majorVersion}.${params.minorVersion}.${params.tinyVersion}.${params.hotfixVersion}"
    def userName = "aldlfkahs"
    def userEmail = "seungwon_lee@tmax.co.kr"


    dir(buildDir){
        stage('Install-hypercloud (git pull)') {
            git branch: "jenkins-test", // 브랜치 바꿔야함
            credentialsId: '${userName}',
            url: "http://${gitAddress}"

            // git pull
            sh "git checkout jenkins-test"
            sh "git config --global user.name ${userName}"
            sh "git config --global user.email ${userEmail}"
            sh "git config --global credential.helper store"

            sh "git fetch --all"
            sh "git reset --hard origin/jenkins-test"
            sh "git pull origin jenkins-test"
        }

        stage('Install-hypercloud (upload CRD yaml)'){

            if (fileExists("${homeDir}/hypercloud-single-operator-v${version}.yaml")){
                sh "cp ${homeDir}/hypercloud-single-operator-v${version}.yaml hypercloud-single-operator/"
            }
            if (fileExists("${homeDir}/hypercloud-multi-operator-v${version}.yaml")){
                sh "cp ${homeDir}/hypercloud-multi-operator-v${version}.yaml hypercloud-multi-operator/"
            }
            if (fileExists("${homeDir}/capi-aws-template-v${version}.yaml")){
                sh "cp ${homeDir}/capi-aws-template-v${version}.yaml hypercloud-multi-operator/"
            }
            if (fileExists("${homeDir}/capi-vsphere-template-v${version}.yaml")){
                sh "cp ${homeDir}/capi-vsphere-template-v${version}.yaml hypercloud-multi-operator/"
            }

            if (fileExists("${homeDir}/convert/result/hypercloud-single-operator-crd-v${version}.yaml")){
                sh "cp ${homeDir}/convert/result/hypercloud-single-operator-crd-v${version}.yaml hypercloud-single-operator/crd/"
            }
            if (fileExists("${homeDir}/convert/result/hypercloud-multi-operator-crd-v${version}.yaml")){
                sh "cp ${homeDir}/convert/result/hypercloud-multi-operator-crd-v${version}.yaml hypercloud-multi-operator/crd/"
            }

            sh "sudo rm -f ${homeDir}/hypercloud-single-operator-v${version}.yaml ${homeDir}/hypercloud-multi-operator-v${version}.yaml"
            sh "sudo rm -f ${homeDir}/convert/*.yaml"
            sh "sudo rm -f ${homeDir}/convert/result/*.yaml"
        }

        stage('Install-hypercloud (git push)'){
            sh "git checkout jenkins-test"

            sh "git config --global user.name ${userName}"
            sh "git config --global user.email ${userEmail}"
            sh "git config --global credential.helper store"
            sh "git add -A"

            def commitMsg = "[Distribution] Upload Operator CRD yaml - v${version}"
            sh (script: "git commit -m \"${commitMsg}\" || true")            

            sh "git remote set-url origin https://${githubUserToken}@github.com/tmax-cloud/install-hypercloud.git"
            sh "sudo git push -u origin +jenkins-test"

            sh "git fetch --all"
            sh "git reset --hard origin/jenkins-test"
            sh "git pull origin jenkins-test"
        }
    }   
}