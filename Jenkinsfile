pipeline {
    agent any
    stages {
        stage ('Build Backend') {
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
                    bat "${scannerHome}/bin/sonar-scanner -e  -Dsonar.projectKey=DeployBack -Dsonar.host.url=http://localhost:9000 -Dsonar.login=33529402b16082b9906cb3bddd94343c130684aa -Dsonar.java.binaries=target -Dsonar.coverage.exclusions=**/.mvn/**,**/src/test/**,**/model/**,**Application.java"
                }   
            }
        }
        stage ('Quality Gate') {
            steps {
                sleep(5)
                timeout(time: 1, unit: 'MINUTES') { 
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage ('Deploy Backend'){
            steps {
                deploy adapters: [tomcat8(credentialsId: 'tomcatlogin', path: '', url: 'http://localhost:8001')], contextPath: 'tasks-backend', war: 'target/tasks-backend.war'
            }
        }
        stage ('API Test'){
            steps {
                dir('api-test') {  
                    git credentialsId: 'Github_Login', url: 'https://github.com/lecosta10/Tasks-API-test'
                    bat 'mvn test' 
                }
            }
        }
        stage ('Deploy Frontend'){
            steps {
                dir ('Frontend') {  
                    git credentialsId: 'Github_Login', url: 'https://github.com/lecosta10/tasks-frontend'    
                    bat 'mvn clean package'
                    deploy adapters: [tomcat8(credentialsId: 'tomcatlogin', path: '', url: 'http://localhost:8001')], contextPath: 'tasks', war: 'target/tasks.war'

                }
            }
        }
        stage ('Functional Test'){
            steps {
                dir('functional-test') {  
                    git credentialsId: 'Github_Login', url: 'https://github.com/lecosta10/Tasks-functional-tests'
                    bat 'mvn test' 
                }
            }
        }
    }
}



 