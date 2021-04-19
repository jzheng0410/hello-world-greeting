pipeline {
    agent none
    stages {
        stage('Compilation et tests') {
            
            agent {
                label 'agent_java'
            }
            
            stages {
                stage('Tests unitaires') {
                    steps {
                        sh 'mvn test'
                    }
                }
                
                stage('Compilation') {
                    steps {
                        sh 'mvn -B -DskipTests clean package'
                    }
                }
                
                stage('Publication du binaire') {
                    steps {
                        sh "curl -u admin:admin --upload-file target/*.war 'http://10.10.20.31/repository/depot_test/app${BUILD_NUMBER}.war'"
                    }
                }
            }
            
            stage('Test de déploiement') {
                agent {
                    label 'agent_tomcat'
                }
                
                stages {
                    stage('Telechargement du binaire') {
                        steps {
                            sh "wget -P /home/jenkins/tomcat/webapps http://10.10.20.31:8081/repository/depot_test/app${BUILD_NUMBER}.war /home/jenkins/tomcat/webapps/app.war"
                            }
                     }
                    stage('Test de performance') {
                        steps {
                            sh '/home/jenkins/apache-jmeter/bin/jmeter.sh -n -t ./jmeter.jmx -l /home/jenkins/test_report.jtl'
                            }
                    }
                    stage('Validation application') {
                        steps {
                            sh "curl -u admin:admin --upload-file /home/jenkins/tomcat/webapps/app.war 'http://10.10.20.31:8081/repository/hello_fiable/app_fiable${BUILD_NUMBER}.war'"
                        }
                    }
                }
            }
            
            stage('Creation image') {
                agent {
                    label 'agent_docker'
                }
                
                stages {
                    stage('Telechargement du binaire') {
                        steps {
                            sh 'wget -P /home/jenkins/docker/tomcat_app http://10.10.20.31:8081/repository/hello_fiable/app_fiable${BUILD_NUMBER}.war'
                            sh 'mv /home/jenkins/docker/tomcat_app/app_fiable${BUILD_NUMBER}.war /home/jenkins/docker/tomcat_app/app.war'
                        }
                    }
                    
                    stage('Compilation image') {
                        steps {
                            sh 'docker build -t tomcat_app /home/jenkins/docker/tomcat_app'
                        }
                    }
                    
                    stage('Stockage image') {
                        steps {
                            sh "docker tag tomcat_app jzheng0410/tomcat_app:${BUILD_NUMBER}"
                            sh 'docker tag tomcat_app jzheng0410/tomcat_app'
                            sh 'docker login -u jzheng0410 -p Jonathan1989@'
                            sh "docker push jzheng0410/tomcat_app:${BUILD_NUMBER}"
                            sh 'docker push jzheng0410/tomcat_app
                        }
                    }
                }
            }'
        }
    }
}
