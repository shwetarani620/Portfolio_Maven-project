pipeline {
   agent any
   tools{
      // jdk 'java11'
      maven 'maven3'
      // sonarqube scanner 'SonarQube-Scanner'
    }
    // environment {
    //   // defining sonarqube enviroment
    //   SONAR_SCANNER = 'SonarQube-Scanner'
    //   SONAR_SERVER = 'sonarqube'
    //     // This can be nexus3 or nexus2
    //   NEXUS_VERSION = "nexus3"
    //   //This can be http or https
    //   NEXUS_PROTOCOL = "http"
    //   //Where your nexus is running
    //   NEXUS_URL = "192.168.80.26:8081"
    //   //Repository where we will upload artifact
    //   NEXUS_REPOSITORY = "Devopsproject"
    //   //jenkins credential id to authenticate to Nexus OSS
    //   NEXUS_CREDENTIAL_ID = "nexus_token"
    // }
    // stages {
    // //   stage('SCM') {
    // //     steps {
    // //       svn 'http://192.168.80.24/svn/devops_project/'
    // //       credentialsId: 'svn_key'
    // //     }
    // //   }
       stages {
    stage ('Initialize') {
      steps {
        sh '''
                echo "PATH = ${PATH}"
                echo "M2_HOME = ${M2_HOME}"
            ''' 
      }
    }
      stage ('Check Secrets') {
       steps {
       sh 'https://github.com/shwetarani620/Portfolio_Maven-project.git -f json -o truffelhog_output.json || true'
       }
     }
      stage ('Software Composition Analysis') {
             steps {
                 dependencyCheck additionalArguments: ''' 
                     -o "./" 
                     -s "./"
                     -f "ALL" 
                     --prettyPrint''', odcInstallation: 'owasp-dc'

                 dependencyCheckPublisher pattern: 'dependency-check-report.xml'
             }
        }
      
   //      steps {
		 // echo '============================== Static Analysis =============================='
   //        withSonarQubeEnv('sonar') {
   //        // sh 'mvn clean sonar:sonar -Dsonar.javabinaries=src -Dsonar.projectName=sonarkey -Dsonar.jacoco.reportsPath=target/jacoco.exec' 
   //        sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=shweta\
   //        -Dsonar.projectName=key \
   //        -Dsonar.projectVersion=1.0 \
   //        -Dsonar.sources=webapp/ \
   //        -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
   //        -Dsonar.junit.reportsPath=targetsurefire-reports/ \
   //        -Dsonar.jacoco.reportsPath=target/site/jacoco/jacoco.xml/ \
   //        -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
   //        }
   //      }
   //    }
  stage ('Static Analysis') {
            steps {
               withSonarQubeEnv('sonar') {
                 // sh 'mvn sonar:sonar'
                sh 'mvn clean sonar:sonar -Dsonar.java.binaries=src'
             }
	  }
	}
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
     // stage ('Fetch Application server') {
      //   steps {
      //     echo '============================== FETCH APPLICATION SERVER =============================='
      //     sshagent(['application_server']) {
      //       sh 'scp -o StrictHostKeyChecking=no /var/lib/jenkins/workspace/hello_world_pipeline/webapp/target/webapp.war root@192.168.80.10:/opt/tomcat/webapps/'
      //     }
      //   }
      // }
      // stage ('Deploy on tomcat') {
      //   steps {
      //     echo '============================== DEPLOY ON TOMCAT =============================='
      //     sshagent(['application_server']) {
      //       // sh 'ssh -o  StrictHostKeyChecking=no root@192.168.80.10 "bash /opt/tomcat/bin/shutdown.sh"'
      //       sh 'ssh -o  StrictHostKeyChecking=no root@192.168.80.10 "bash /opt/tomcat/bin/startup.sh"'
      //     }
      //   }
      // }
      // stage ('Dynamic analysis') {
      //   steps {
      //     echo '============================== RUNNING SOFTWARE TESTING  =============================='
      //     sshagent(['application_server']) {
      //       sh 'ssh -o  StrictHostKeyChecking=no root@192.168.80.10 "sudo docker run --rm -v /root:/zap/wrk/:rw -t owasp/zap2docker-stable zap-full-scan.py -t http://192.168.80.10:8080/webapp/ -x zap_report || true"'
	     //    }
      //   }
      // }
    }
}
