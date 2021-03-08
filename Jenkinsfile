node {
    def gitHubBaseAddress = "github.com"
	def gitHcIntegratedAddress = "github.com/tmax-cloud/hypercloud-api-server.git"
	def hcIntegratedBuildDir = "/var/lib/jenkins/workspace/hypercloud5-integrated"

	def version = "${params.majorVersion}.${params.minorVersion}.${params.tinyVersion}.${params.hotfixVersion}"
	def preVersion = "${params.preVersion}"
	def imageTag = "b${version}"

	def userName = "dnxorjs1"
	def userEmail = "taegeon_woo@tmax.co.kr"

	switch(Distribution-Type){
	    case "integrated":
	        DisApiServer()
	        DisSingleOperator()
	        DisMultiOperator()
	        break
        case "only-api-server":
            DisApiServer()
            break
        case "only-single-operator":
            DisApiServer()
            break
        case "only-multi-operator":
            echo "test!!!"
            DisApiServer()
            break
        default:
            break
	}
}
void DisApiServer() {
    def gitHubBaseAddress = "github.com"
    def gitAddress = "${gitHubBaseAddress}/tmax-cloud/hypercloud-api-server.git"
    def buildDir = "/var/lib/jenkins/workspace/hypercloud5-integrated/api-server"
    def imageBuildHome = "${buildDir}"
    def scriptHome = "${buildDir}/scripts"
    def gitHcAddress = "${gitHubBaseAddress}/tmax-cloud/hypercloud-api-server.git"
    def version = "${params.majorVersion}.${params.minorVersion}.${params.tinyVersion}.${params.hotfixVersion}"
    def preVersion = "${params.preVersion}"
    def imageTag = "b${version}"
    def userName = "dnxorjs1"
    def userEmail = "taegeon_woo@tmax.co.kr"

    dir(BuildDir){
        stage('git pull & Go build') {
            dir(imageBuildHome){
                echo "test"
            }
        }
    }
}



