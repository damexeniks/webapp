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
    
     stage ('Source Composition Analysis') {
      steps {
         sh 'rm owasp* || true'
         sh 'wget "https://raw.githubusercontent.com/damexeniks/webapp/master/owasp-dependency-check.sh" '
         sh 'chmod 777 owasp-dependency-check.sh'
         sh 'mkdir -p "$(pwd)/odc-reports" '
         sh 'bash owasp-dependency-check.sh'
        
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
   
