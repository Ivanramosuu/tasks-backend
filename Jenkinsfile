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
                sleep(10)
                timeout(time: 2, unit: 'MINUTES') {
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
        stage ('Deploy Frontend') {
            steps {
                dir('frontend') {
                    git credentialsId: 'github_login', url: 'https://github.com/Ivanramosuu/tasks-frontend.git'
                    bat 'mvn clean package' 
                    deploy adapters: [tomcat8(credentialsId: 'ToncatUser', path: '', url: 'http://192.168.0.9:8001/')], contextPath: 'tasks', war: 'target/tasks.war'   
                }
            }
        }
        stage ('Functional Test') {
            steps {
                dir('functional-test') {
		            git credentialsId: 'github_login', url: 'https://github.com/Ivanramosuu/tasks-functional-test.git'
                    bat 'mvn test' 
                }
            }
        }
        stage ('Deploy Prod') {
            steps {
                bat 'docker-compose-v1 build'
                bat 'docker-compose-v1 up -d'
            }
        }
    }
}
