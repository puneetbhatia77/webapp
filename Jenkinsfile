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
        sh 'docker run gesellix/trufflehog --json https://github.com/puneetbhatia77/webapp.git > trufflehog'
        sh 'cat trufflehog'
      }
    }
    
    stage ('Source Composition Analysis') {
          steps {
             sh 'rm owasp* || true'
             sh 'wget "https://raw.githubusercontent.com/puneetbhatia77/webapp/master/owasp-dependency-check.sh" '
             sh 'chmod +x owasp-dependency-check.sh'
             sh 'bash owasp-dependency-check.sh'
             sh 'cat /var/lib/jenkins/OWASP-Dependency-Check/reports/dependency-check-report.xml'
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
    
       stage ('Deploy-To-Tomcat') {
            steps {
           sshagent(['tomcat']) {                
                sh 'sudo scp -o StrictHostKeyChecking=no target/*.war /root/apache-tomcat-8.5.57/webapps/webapp.war'     
           }
           }       
        }
         
    stage ('DAST') {
      steps {
        sshagent(['zap']) {
         sh 'docker run -t owasp/zap2docker-stable zap-baseline.py -t http://192.168.56.101:8081/webapp/ || true'
        }
      }
    }
    
    
       }
    }
