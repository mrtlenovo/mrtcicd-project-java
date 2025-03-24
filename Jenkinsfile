def COLOR_MAP = [
    'SUCCESS': 'good',
    'FAILURE': 'danger',
]

pipeline {
    agent any
    tools {
        maven "maven3"
        jdk "jdk11"
    }

    environment {
        SNAP_REPO = 'vprofile-snapshot'
        NEXUS_USER = 'admin'
        NEXUS_PASS = 'test@123'
        RELEASE_REPO = 'vprofile-release'
        CENTRAL_REPO = 'vpro-maven-central'
        NEXUS_IP = '10.129.2.195'
        NEXUS_PORT = '8081'
        NEXUS_GRP_REPO = 'vpro-maven-group'
        NEXUS_LOGIN = 'nexus-creds'
        SONARSERVER = 'SonarQube'
        SONARSCANNER = 'sonarscanner'
    }

    stages {
        stage('Build'){
            steps {
                sh 'mvn -s settings.xml -DskipTests install'
            }
            post {
                success {
                    echo 'Archiving'
                    archiveArtifacts artifacts: '**/*.war'
                }
            }
            
        }

        stage('Test') {
            steps {
                sh 'mvn -s settings.xml test'
            }
        }

        stage('Checkstyle Analysis') {
            steps {
                sh 'mvn -s settings.xml checkstyle:checkstyle'
            }
        }

        stage ('Sonar Analysis') {
            environment {
                scannerHome = tool "${SONARSCANNER}" 
            }
            steps {
              withSonarQubeEnv("${SONARSERVER}") {
                 sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                     -Dsonar.projectName=vprofile-repo \
                     -Dsonar.projectVersion=1.0 \
                     -Dsonar.sources=/var/lib/jenkins/workspace/mrtcicd-pipeline-java/ \
                     -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                     -Dsonar.exclusions=**/*.js,**/*.ts,**/*.css,**/*.jps \
                     -Dsonar.junit.reportsPath=target/surefire-reports/ \
                     -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                     -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
                }
            }
        }

        stage ("Quality Gate") {
            steps {
                timeout(time:1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage ("Upload Artifact") {
            steps {
                nexusArtifactUploader(
                  nexusVersion: 'nexus3',
                  protocol: 'http',
                  nexusUrl: "${NEXUS_IP}:${NEXUS_PORT}",  
                  groupId: 'QA',
                  version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}",
                  repository: "${RELEASE_REPO}",
                  credentialsId: "${NEXUS_LOGIN}", 
                  artifacts: [
                    [artifactId: 'vproapp',
                     classifier: '',
                     file: 'target/vprofile-v2.war',
                     type: 'war']
                  ]
                )
            }
        }
    }
}
