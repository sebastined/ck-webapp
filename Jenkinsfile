pipeline {
  agent any 
  tools {
    maven 'Maven3'
    }
  triggers {
      pollSCM '*/5 * * * *'
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
        sh 'docker run gesellix/trufflehog --json https://github.com/sebastined/ck-webapp.git > trufflehog'
        sh 'cat trufflehog'
      }
    }
    
    stage ('Source Composition Analysis') {
      steps {
         sh 'rm owasp* || true'
         sh 'wget "https://raw.githubusercontent.com/sebastined/ck-webapp/master/owasp-dependency-check.sh" '
         sh 'chmod +x owasp-dependency-check.sh'
         sh 'bash owasp-dependency-check.sh'
         sh 'cat /var/lib/jenkins/OWASP-Dependency-Check/reports/dependency-check-report.xml'
         sh 'mv /var/lib/jenkins/OWASP-Dependency-Check/reports/dependency-check-report.xml /var/lib/jenkins/dumps'
        
      }
    }
    
    stage ('SAST') {
      steps {
        withSonarQubeEnv('sonar') {
          sh 'mvn sonar:sonar'
          sh 'cat /var/lib/jenkins/workspace/webapp00/target/sonar/report-task.txt'
        }
      }
    }
    
    stage ('Build') {
      steps {
        sh 'mvn clean package'
      }
    }
    
    stage ('Deploy-To-Docker') {
      steps {
        script {
          // Build Docker image
          sh 'docker build -t sebastined/webapp .'
          
          // Tag Docker image
          sh 'docker tag webapp -name sebastined/webapp'
          
          // Push Docker image to Docker registry
          sh 'docker push sebastined/webapp'
        }
      }
    }
  }
}
