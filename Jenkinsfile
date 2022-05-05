pipeline {
  agent any 
  tools {
    maven 'Maven'
  }
  stages {
    stage ('Initialize') {
      steps {
        sh '''
                    echo "PATH = ${PATH}"
                    echo "M2_HOME = ${M2_HOME}"
            ''' 
      }
    }
    
    stage ('Check-Git-Secrets') {
      steps {
        sh 'rm trufflehog || true'
        sh 'docker run gesellix/trufflehog --json https://github.com/cehkunal/webapp.git > trufflehog'
        sh 'cat trufflehog'
      }
    }
    
      stage('Source Composition Analysis') {
       environment {
          WS_APIKEY = credentials('whitesource-apikey')
          WS_WSS_URL = "https://saas-eu.whitesourcesoftware.com/agent"
          WS_USERKEY = credentials('whitesource-serviceaccount-userkey')
          WS_PRODUCTNAME = "webapp_test"
          WS_PROJECTNAME = "webapp_test"
    } 
    steps {
   	sh 'curl -LJO https://unified-agent.s3.amazonaws.com/wss-unified-agent.jar'
    	sh 'java -jar wss-unified-agent.jar' 
  }
}
    
    stage ('SAST') {
      steps {
        withSonarQubeEnv('sonar') {
          sh 'mvn sonar:sonar'
          sh 'cat target/sonar/report-task.txt'
        }
      }
    }
    
    stage ('Build') {
      steps {
      sh 'mvn clean package'
       }
    }
    
      stage ('DAST') {
      steps {
        sshagent(['zap']) {
         sh 'ssh -o  StrictHostKeyChecking=no ubuntu@54.226.24.142 "docker run -t owasp/zap2docker-stable zap-baseline.py -t http://18.232.140.134:8080/webapp/" || true'
        }
      }
    }
    
    stage ('Deploy-To-Tomcat') {
            steps {
           sshagent(['tomcat']) {
                sh 'scp -o StrictHostKeyChecking=no target/*.war ubuntu@18.232.140.134:/home/ubuntu/prod/apache-tomcat-9.0.62/webapps/webapp.war'
              }      
           }       
    }
    
  }
}
