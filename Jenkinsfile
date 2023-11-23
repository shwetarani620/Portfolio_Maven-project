pipeline {
	agent any
	  tools{
      jdk 'java11'
      maven 'maven3'
      // sonarqube scanner 'SonarQube-Scanner'
    }
    environment {
      // defining sonarqube enviroment
      SONAR_SCANNER = 'SonarQube-Scanner'
      SONAR_SERVER = 'sonarqube'
        // This can be nexus3 or nexus2
      NEXUS_VERSION = "nexus3"
      //This can be http or https
      NEXUS_PROTOCOL = "http"
      //Where your nexus is running
      NEXUS_URL = "192.168.80.26:8081"
      //Repository where we will upload artifact
      NEXUS_REPOSITORY = "Devopsproject"
      //jenkins credential id to authenticate to Nexus OSS
      NEXUS_CREDENTIAL_ID = "nexus_token"
    }
    stages {
    //   stage('SCM') {
    //     steps {
    //       svn 'http://192.168.80.24/svn/devops_project/'
    //       credentialsId: 'svn_key'
    //     }
    //   }
      stage ('Initialize') {
        steps {
          echo '============================== PATH INITIALIZED =============================='
          sh '''
                  echo "PATH = ${PATH}"
                  echo "M2_HOME = ${M2_HOME}"
              ''' 
        }
      }
      stage ('Check secrets') {
        steps {
          echo '================================ CHECK SECRET ================================='
          sh 'trufflehog3 http://192.168.80.24/svn/devops_project/ -f json -o truffelhog_output.json || true'
        }
       }
      stage ('Software Composition Analysis') {
        steps {
          echo '============================== DEPENDENCY CHECK =============================='
          dependencyCheck additionalArguments: ''' 
              -o "./" 
              -s "./"
              -f "ALL" 
              --prettyPrint''', odcInstallation: 'Owasp-DC'
          dependencyCheckPublisher pattern: 'dependency-check-report.xml'
        }
      }
      stage('SonarQube analysis') {
        environment {
          scannerHome = tool "${SONAR_SCANNER}"
        }
        steps {
          echo '============================== SONARQUBE SCANNER =============================='
          withSonarQubeEnv('sonarqube') {
          // sh 'mvn clean sonar:sonar -Dsonar.javabinaries=src -Dsonar.projectName=sonarkey -Dsonar.jacoco.reportsPath=target/jacoco.exec' 
          sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=hello_world_pipeline \
          -Dsonar.projectName=sonarkey \
          -Dsonar.projectVersion=1.0 \
          -Dsonar.sources=webapp/ \
          -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
          -Dsonar.junit.reportsPath=targetsurefire-reports/ \
          -Dsonar.jacoco.reportsPath=target/site/jacoco/jacoco.xml/ \
          -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
          }
        }
      }
   //    stage ('Static analysis') {
   //      steps {
   //        echo '=========== SonarQube analysis ============'
   //        withSonarQubeEnv('sonarqube') {
	  // sh 'mvn sonar:sonar'
   //        // sh '''$SCANNER_HOME/bin/sonarqube -Dsonar.projectName=hello_world_pipeline \
   //        // -Dsonar.java.binaries=. \
   //        // -Dsonar.projectKey=sonarkey '''
   //        }
   //      }
   //    }
      // stage('SonarQube Analysis') {
      //   steps{  // def mvn = tool 'Default Maven';
      //     withSonarQubeEnv('SonarQube-Scanner') {
      //       sh "${mvn}/bin/mvn clean verify sonar:sonar -Dsonar.projectKey=hello_world_pipeline -Dsonar.projectName='hello_world_pipeline'"
      //     }
      //   }
      // }
      stage('Generate and compile') {
        steps {
          echo '============================== SOFTWARE COMPILE =============================='
          sh "mvn compile"
        }
      }
      stage('Test Application'){
        steps{
          echo '============================== SOFTWARE TEST =============================='
          sh 'mvn test'
        }
      }
      stage('Delete old Application'){
        steps{
          echo '============================== DELETE OLD BUILD SOFTWARE =============================='
          sh 'mvn clean package'
        }
      }
      stage('Build Application'){
        steps{
          echo '============================== BUILD SOFTWARE =============================='
          sh 'mvn clean install'
        }
      }
      stage("publish to nexus") {
        steps {
          echo '============================== UPLOADING ARTIFACTS TO NEXUS =============================='
          nexusArtifactUploader artifacts: [
            [
              artifactId: 'maven-project', 
              classifier: '', 
              file: 'webapp/target/webapp.war', 
              type: 'war'
            ]
          ], 
          credentialsId: 'nexuscredential', 
          groupId: 'com.example.maven-project', 
          nexusUrl: '192.168.80.26:8081', 
          nexusVersion: 'nexus3', 
          protocol: 'http', 
          repository: 'Devopsproject', 
          version: '1.0-SNAPSHOT'
        }
      }
      stage ('Fetch Application server') {
        steps {
          echo '============================== FETCH APPLICATION SERVER =============================='
          sshagent(['application_server']) {
            sh 'scp -o StrictHostKeyChecking=no /var/lib/jenkins/workspace/hello_world_pipeline/webapp/target/webapp.war root@192.168.80.10:/opt/tomcat/webapps/'
          }
        }
      }
      stage ('Deploy on tomcat') {
        steps {
          echo '============================== DEPLOY ON TOMCAT =============================='
          sshagent(['application_server']) {
            // sh 'ssh -o  StrictHostKeyChecking=no root@192.168.80.10 "bash /opt/tomcat/bin/shutdown.sh"'
            sh 'ssh -o  StrictHostKeyChecking=no root@192.168.80.10 "bash /opt/tomcat/bin/startup.sh"'
          }
        }
      }
      stage ('Dynamic analysis') {
        steps {
          echo '============================== RUNNING SOFTWARE TESTING  =============================='
          sshagent(['application_server']) {
            sh 'ssh -o  StrictHostKeyChecking=no root@192.168.80.10 "sudo docker run --rm -v /root:/zap/wrk/:rw -t owasp/zap2docker-stable zap-full-scan.py -t http://192.168.80.10:8080/webapp/ -x zap_report || true"'
	        }
        }
      }
    }
}
