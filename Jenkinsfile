import groovy.json.JsonSlurper
import java.text.SimpleDateFormat

def targetEnv = "dev2"
def jobName = "IndiaStudio"

def agentLabel_master = "master"
def agentLabel_slave01 = "master"
def workPath_slave01 = "C:\\Work\\project\\git-pc\\gentoolgroup\\indiastudiodeploy\\tmp\\IndiaStudioDist_dev2_20201109-171316\\Slave02_local01\\"

def agentLabel_slave02local01 = ""master"
def workPath_slave02local01 = "C:\\Work\\project\\git-pc\\gentoolgroup\\indiastudiodeploy\\tmp\\IndiaStudioDist_dev2_20201109-171316\\Slave02_local01\\"

def agentLabel_slave02local02 = "master"
def workPath_slave02local02 = "C:\\Work\\project\\git-pc\\gentoolgroup\\indiastudiodeploy\\tmp\\IndiaStudioDist_dev2_20201109-171316\\Slave02_local01\\"

def buildId = ""
def currentPath = workPath_slave01
def currentAgentLabel = agentLabel_slave01
boolean getJobResult = false
baseUrl = ""

def getStudioUrl(targetEnv)
{
    echo "getStudioUrl"
	def studioReqUrl = 'http://api-' + targetEnv + '.onstove.com/ssg/default/clients/pc-client/lastest/india_studio_url'
	echo "studioReqUrl = ${studioReqUrl}"
	
	def urlResp = httpRequest url: "${studioReqUrl}", validResponseContent: 'OK'
    echo "urlResp = ${urlResp.content}"
    
	studioUrl = ""
    def jsonUrlResp = new JsonSlurper().parseText(urlResp.content)
	if(jsonUrlResp.value != null && jsonUrlResp.value.categories != null && jsonUrlResp.value.categories.urls != null) {
		studioUrl = jsonUrlResp.value.categories.urls.india_studio_url_internal
		echo "getStudioUrl : studioUrl = ${studioUrl}"
	}
	baseUrl = studioUrl
    return studioUrl
}

def getJob(studioUrl) {
    echo "getJob"
    def reqUrl = studioUrl + '/open/bvt/builds/poll?size=1'
    echo "reqUrl = ${reqUrl}"

    def response = httpRequest url: "${reqUrl}", validResponseContent: 'OK'
    echo "response = ${response.content}"
    return response.content
}

pipeline {
    options {
      timeout(time: 20, unit: 'MINUTES') //timeout(time: 1, unit: 'HOURS') 
    }
    agent {
        label 'master'
    } 
    stages {
        stage('01_GetJob') {     
            agent {
                label currentAgentLabel
            } 
            steps {
                script {
					echo "${jobName}"
					def url = getStudioUrl(targetEnv);
					if(url == "") {
						echo('error - get studio url fail')
						error('get studio url fail')
					}
					def content = getJob(url)
					def object = new JsonSlurper().parseText(content)
					if(object.keySet().containsAll(['code','message','value'])) 
					{
						def keyList = object.value
						if(keyList.size()>0) {
							buildId = keyList.get(0).build_id
							echo "${buildId}"
							getJobResult = true
						}
						else
						{
							echo 'no job'
							//error('no job')
							getJobResult = false
						}
					}
					else
					{
						echo 'no job(no value)'
						//error('no job')
						getJobResult = false
					}
                }
            }
        }
        stage('02-FileDownload'){
			when {
				expression { getJobResult == true }
			}
            agent {
                label currentAgentLabel
            } 
            steps(){
				script {
					echo "currentAgentLabel: ${currentAgentLabel}"
					currentPath = workPath_slave01
					def runWithParam = "D:\\python373\\python " + currentPath + "Unify_test.py UPLOAD_UPLOAD " + buildId + " " + targetEnv + " " + env.BUILD_NUMBER + " " + currentPath + " " + baseUrl
					echo "runWithParam= ${runWithParam}"
					bat """
					${runWithParam}
					"""
					
					currentAgentLabel = agentLabel_slave02local01
					echo "currentAgentLabel: ${currentAgentLabel}"
				}
            }
        }
        stage('03-PreVerifyTool'){
			when {
				expression { getJobResult == true }
			}
            agent {
                label currentAgentLabel
            } 
            steps(){
				script {
					echo "currentAgentLabel: ${currentAgentLabel}"
					currentPath = workPath_slave02local01
					def runWithParam = "C:\\python373\\python " + currentPath + "Unify_test.py DEPLOY_FILESCAN " + buildId + " " + targetEnv + " " + env.BUILD_NUMBER + " " + currentPath + " " + baseUrl
					echo "runWithParam= ${runWithParam}"
					bat """
					${runWithParam}
					"""						
				}
            }
        }
        stage('04-GenTool(build)'){
			when {
				expression { getJobResult == true }
			}
            agent {
                label currentAgentLabel
            } 
            steps(){
                script {
					currentPath = workPath_slave02local01
					def runWithParam = "C:\\python373\\python " + currentPath + "Unify_test.py DEPLOY_PRE_BUILD " + buildId + " " + targetEnv + " " + env.BUILD_NUMBER + " " + currentPath + " " + baseUrl
					echo "runWithParam= ${runWithParam}"
					bat """
					${runWithParam}
					"""
                }
            }
        }
        stage('05-GenTool(upload)'){
			when {
				expression { getJobResult == true }
			}
            agent {
                label currentAgentLabel
            } 
            steps(){
                script {
					currentPath = workPath_slave02local01
					def runWithParam = "C:\\python373\\python " + currentPath + "Unify_test.py DEPLOY_BUILD " + buildId + " " + targetEnv + " " + env.BUILD_NUMBER + " " + currentPath + " " + baseUrl
					echo "runWithParam= ${runWithParam}"
					bat """
					${runWithParam}
					"""
                }
            }
        }
        stage('06-VerifyTool'){
			when {
				expression { getJobResult == true }
			}
            agent {
                label currentAgentLabel
            } 
            steps(){
                script {
					currentPath = workPath_slave02local01
					def runWithParam = "C:\\python373\\python " + currentPath +  "Unify_test.py DEPLOY_UPLOAD " + buildId + " " + targetEnv + " " + env.BUILD_NUMBER + " " + currentPath + " " + baseUrl
					echo "runWithParam= ${runWithParam}"
					bat """
					${runWithParam}
					"""
                }
            }
        }
    }
    post { 
        failure {  
            node(agentLabel_master) {
                script{
					//FileCopy.bat dev2 20201022 5f8fe2234fec938e2d989a4d668965aed8dfa52a 91 IndiaStudio \\\\10.250.214.67\\workspace \\\\10.250.214.86\\workspace \\\\10.250.214.86\\workspace2
					def dateFormat = new SimpleDateFormat("yyyyMMdd")
					def date = new Date()
					today = dateFormat.format(date)      
					
					//buildId="5f8fe2234fec938e2d989a4d668965aed8dfa52a"
					buildNumber = env.BUILD_NUMBER
					if(buildId != "") {
						def param = targetEnv + ' ' + today + ' ' + buildId + ' ' + buildNumber + ' ' + jobName + ' ' + '\\\\10.250.214.67\\workspace \\\\10.250.214.86\\workspace \\\\10.250.214.86\\workspace2'
						echo "param : ${param}"
						bat 'C:\\Jenkins_indiastudio\\LogUtil\\FileCopy.bat ' + param
					}
					
					def logPath = "C:\\Program Files (x86)\\Jenkins\\jobs\\" + env.JOB_NAME + "\\builds\\" + buildNumber +"\\log"
					bat 'C:\\Python373\\python c:\\Jenkins_indiastudio\\LogUtil\\prettier.py ' + '"' + logPath + '"'
					
					def subject = 'Build failed in Jenkins : ' + env.JOB_NAME +  '#' + buildNumber
					echo "subject : ${subject}"
				 
					def command = 'C:\\Python373\\python c:\\Jenkins_indiastudio\\LogUtil\\send.py ' +   '"' + subject + '" "'  + env.JOB_NAME +  '" "'  + buildId  + '" "' + buildNumber + '" "'+ targetEnv + '" "'
					echo "command : ${command}"
					bat "${command}"
                }
            }
        }
    }
}

