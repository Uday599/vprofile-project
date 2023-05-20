pipeline {

    agent any
/*
	tools {
        maven "maven3"
    }
*/
    environment {
        registry = "uday1011/vproappdock"
        registryCredential = 'dockerhub'
    }

    stages{

        stage('BUILD'){
            steps {
                sh 'mvn clean install -DskipTests'
            }
            post {
                success {
                    echo 'Now Archiving...'
                    archiveArtifacts artifacts: '**/target/*.war'
                }
            }
        }

        stage('UNIT TEST'){
            steps {
                sh 'mvn test'
            }
        }

        stage('INTEGRATION TEST'){
            steps {
                sh 'mvn verify -DskipUnitTests'
            }
        }

        stage ('CODE ANALYSIS WITH CHECKSTYLE'){
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
            post {
                success {
                    echo 'Generated Analysis Result'
                }
            }
        }


        stage('CODE ANALYSIS with SONARQUBE') {

            environment {
                scannerHome = tool 'mysonarscanner4'
            }

            steps {
                withSonarQubeEnv('sonar-pro') {
                    sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                   -Dsonar.projectName=vprofile-repo \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
                }

                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        
        stage('Building App image') {
            steps{
              script {
                dockerImage = docker.build registry + ":$BUILD_NUMBER"   // Docker file should present in source code
              }
            }
        }
        
        stage('Upload/Deploy Image') {
          steps{
            script {
              docker.withRegistry( '', registryCredential ) {  // passing docker hub creds
                dockerImage.push("V$BUILD_NUMBER")
                dockerImage.push('latest')
              }
            }
          }
        }

         stage('Remove Unused docker image') {   // parallely deleteing images in our local system to free up space
          steps{
            sh "docker rmi $registry:V$BUILD_NUMBER"
          }
        }

        stage('Kubernetes Deploy') {
	  agent { label 'k8-node' }
            steps {
                    sh "helm upgrade --install --force vproifle-stack helm/vprofilecharts --set appimage=${registry}:V${BUILD_NUMBER} --namespace prod"
		    
		    // helm upgrade -- install-- force name_of_stack Path/to/chart  --set appimages= Variable_need_to_pass --namespace prod
            }
        }

    }


}
