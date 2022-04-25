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
    
    node {
  environment {
       WS_APIKEY = credentials('whitesource-apikey')
       WS_WSS_URL = "https://saas-eu.whitesourcesoftware.com/agent"
       WS_USERKEY = credentials('whitesource-serviceaccount-userkey')
       WS_PRODUCTNAME = "Your Product Name"
       WS_PROJECTNAME = "${JOB_NAME}"
   }
  stage('Download Unified Agent') {
   sh 'curl -LJO https://unified-agent.s3.amazonaws.com/wss-unified-agent.jar'
  }
  stage('Run Unified Agent') {
    sh 'java -jar wss-unified-agent.jar' 
  }
}
    
    stage ('Build') {
      steps {
      sh 'mvn clean package'
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
   
