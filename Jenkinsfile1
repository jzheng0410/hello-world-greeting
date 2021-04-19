pipeline {
  
  agent none
  
  stages {
    
    stage('Compilation et tests') {

      agent {
        label 'agent_java'
      }

      stages {
  
        stage('Test unitaire & publication') {
    
          steps {
            sh 'mvn test'
          }
      
          post {
      
            always {        
              junit 'target/surefire-reports/*.xml'
            }
        
          }
      
        }

        stage('Analyse statique') {
      
          steps {
        
            withSonarQubeEnv('SonarQube') {
              sh 'mvn org.sonarsource.scanner.maven:sonar-maven-plugin:3.2:sonar'
            }

          }
      
        }

        stage('Compilation') {
    
          steps {  
            sh 'mvn -B -DskipTests clean package'
          }
      
        }
            
        stage('Publication du binaire') {

          steps {
            sh "curl -u admin:Shaymin122 --upload-file target/*.war 'http://84.39.43.46:8081/repository/depot_test/rondoudou${BUILD_NUMBER}.war'"        
          }

        }
    
      }
  
    }
    
    stage('Tests de déploiement') {
      
      agent {
        label 'agent_tomcat'
      }
      
      stages {
        
        stage('Téléchargement du binaire') {
          
          steps {
            sh "wget -P /home/jenkins/tomcat/webapps http://84.39.43.46:8081/repository/depot_test/rondoudou${BUILD_NUMBER}.war"
            sh "mv /home/jenkins/tomcat/webapps/rondoudou${BUILD_NUMBER}.war /home/jenkins/tomcat/webapps/rondoudou.war"
          }
 
        }
        
        stage('Test de performance') {
          
          steps {
            sh '/home/jenkins/apache-jmeter/bin/jmeter.sh -n -t ./jmeter.jmx -l /home/jenkins/test_report.jtl'
          }
         
        }
        
        stage ('Validation de l\'application') {
  
          steps {
    
            sh "curl -u admin:Shaymin122 --upload-file /home/jenkins/tomcat/webapps/rondoudou.war 'http://84.39.43.46:8081/repository/hello_fiable/rondoudou_fiable${BUILD_NUMBER}.war'"
            sh "curl -u admin:Shaymin122 --upload-file /home/jenkins/tomcat/webapps/rondoudou.war 'http://84.39.43.46:8081/repository/hello_livrable/dernier_rondoudou_fiable.war'"
  
          }
  
        }
        
      }
      
    }
    
    stage('Creation de l\'image') {
      
      agent {
        label 'agent_docker'
      }
      
      stages {
        
        stage('Téléchargement du binaire') {
          
          steps {
            sh 'wget -P /home/jenkins/docker/tomcat_app http://84.39.43.46:8081/repository/hello_livrable/dernier_rondoudou_fiable.war'
            sh 'mv /home/jenkins/docker/tomcat_app/dernier_rondoudou_fiable.war /home/jenkins/docker/tomcat_app/rondoudou.war'
          }
          
        }
          
        stage('Compilation de l\'image') {
      
          steps {
            sh 'docker build -t tomcat_app /home/jenkins/docker/tomcat_app'
          }
          
        }
        
        stage('Stockage de l\'image') {
              
          steps {
            sh "docker tag tomcat_app reeban/tomcat_app:${BUILD_NUMBER}"
            sh 'docker tag tomcat_app reeban/tomcat_app'
            sh 'docker login -u reeban -p Shaymin122'
            sh "docker push reeban/tomcat_app:${BUILD_NUMBER}"
            sh 'docker push reeban/tomcat_app'
          }
         
        }
      }
    }
  }
}
