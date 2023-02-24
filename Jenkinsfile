	def COLOR_MAP = [
	    'SUCCESS': 'good',  // green color
	    'FAILURE': 'danger',  // red color
	]

pipeline {
    agent any
  
    
    environment {
        
        NEXUSPASS = credentials('nexuspass')

    }

    stages {

        stage('Setup Parameter'){

            steps{
                script {
                    properties ([
                        parameters([
                            string(
                                defaultValue: '',
                                name: 'BUILD',
                            ),
                            string(
                                defaultValue: '',
                                name: 'TIME',
                            )
                        ])
                    ])
                }
            }
        }
       
        stage('Ansible Deploy to PROD'){
            steps{
                ansiblePlaybook([
                inventory   : 'ansible/prod.inventory',
                playbook    : 'ansible/site.yml',
                installation: 'ansible',
                colorized   : true,
                credentialsId: 'appserver-prod', //app server creds on jenkins
                disableHostKeyChecking: true,
                extraVars   : [
                    USER: "admin",
                    PASS: "${NEXUSPASS}",
                    nexusip: "172.31.13.216",
                    reponame: "vprofile-release",
                    groupid: "QA",
                    time: "${env.TIME}",
                    build: "${env.BUILD}",
                    artifactid: "vproapp",
                    vprofile_version: "vproapp-${env.BUILD}-${env.TIME}.war"

                ]

                ])
            }
        }
    }
        post {
	        always {
	            echo 'Slack Notifications.'
	            slackSend channel: '#jenkinscicd',
	                color: COLOR_MAP[currentBuild.currentResult],
	                message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n More info at: ${env.BUILD_URL}"
	        }
       } 
}


