def COLOR_MAP = [
    'SUCCESS': 'good', 
    'FAILURE': 'danger',
]

pipeline {
    
	agent any
	
	tools {
        maven "Maven3"
        jdk "OracleJDK11"
    }
	
    environment {
        NEXUS_VERSION = "nexus3"
        NEXUS_USER = 'admin'
        NEXUS_PASS = 'seun'
        NEXUS_PROTOCOL = "http"
        NEXUSIP = "172.31.46.214"
        NEXUSPORT = '8081'
        RELEASE_REPO = "thedev-repo-1"
        CENTRAL_REPO = "thedevcloud-proxy-repo"
	    NEXUS_GRP_REPO    = "devcloud-grp-repo"
        NEXUS_LOGIN = "nexuslogin"
        ARTVERSION = "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}"
    }
	
    stages{
        
        stage('BUILD'){
            steps {
                sh 'mvn -s settings.xml clean install -DskipTests'
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
                sh 'mvn -s settings.xml test'
            }
        }

	    stage('INTEGRATION TEST'){
            steps {
                sh 'mvn -s settings.xml verify -DskipUnitTests'
            }
        }
		
        stage ('CODE ANALYSIS WITH CHECKSTYLE'){
            steps {
                sh 'mvn -s settings.xml checkstyle:checkstyle'
            }
            post {
                success {
                    echo 'Generated Analysis Result'
                }
            }
        }

        stage('CODE ANALYSIS with SONARQUBE') {
          
		  environment {
             scannerHome = tool 'sonar-scan'
          }

          steps {
            withSonarQubeEnv('jenkins_sonar') {
               sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=thedevcloud \
                   -Dsonar.projectName=thedevcloud-app \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
            }
            /*
            timeout(time: 10, unit: 'MINUTES') {
               waitForQualityGate abortPipeline: true
            }*/
          }
        }

        stage("Publish to Nexus Repository Manager") {
            steps {
                script {
                    nexusArtifactUploader(
                            nexusVersion: "${NEXUS_VERSION}",
                            protocol: "${NEXUS_PROTOCOL}",
                            nexusUrl: "${NEXUSIP}:${NEXUSPORT}",
                            groupId: 'QA',
                            version: "${ARTVERSION}",
                            repository: "${RELEASE_REPO}",
                            credentialsId: "${NEXUS_LOGIN}",
                            artifacts: [
                                [artifactId: 'thedevcloudapp',
                                 classifier: '',
                                 file: 'target/vprofile-v2.war',
                                 type: 'war']
                            ]
                        )
                }
            }
        }
    }

    post {
        always {
            echo 'Slack Notifications.'
            slackSend channel: '#build_jenkins',
                color: COLOR_MAP[currentBuild.currentResult],
                message: "*${currentBuild.currentResult}:* Job: ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n More info at: ${env.BUILD_URL}"
        }
    }
}