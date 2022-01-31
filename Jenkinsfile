pipeline {
    agent any
    stages {
        stage ('Build backend') {
            steps {
                bat 'mvn clean package -DskipTests=true'
            }   
        }
        stage ('Unit Tests') {
            steps {
                bat 'mvn test'
            }   
        }
        stage ('Sonar Analysis') {
            environment {
                scannerHome = tool 'SONAR_SCANNER'
            }
            steps {
                withSonarQubeEnv('SONAR_LOCAL') {
                    bat "${scannerHome}/bin/sonar-scanner -e -Dsonar.projectKey=DeployBack -Dsonar.host.url=http://localhost:9000 -Dsonar.login=97c1b8fbbcb13fb2e394e4209ba681bf323bbf43 -Dsonar.java.binaries=target -Dsonar.coverage.exclusions=**/.mvn/**,**/src/test/**,**/model/**,**Application.java"
                }
            }   
        }
        stage ('Quality Gate') {
            steps {
                //sleep(10)
                timeout(time: 1, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }    
            }
        }
        stage ('Deploy Backend') {
            steps {
                deploy adapters: [tomcat8(credentialsId: 'ToncatUser', path: '', url: 'http://192.168.0.9:8001/')], contextPath: 'tasks-backend', war: 'target/tasks-backend.war'
            }
        }
        stage ('API Test') {
            steps {
				dir('api-test') {
					git credentialsId: 'github_login', url: 'https://github.com/Ivanramosuu/tasks-api-test.git' 
					bat 'mvn test'
				}	
            }
        }
    }
}
