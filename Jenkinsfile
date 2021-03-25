pipeline{
    agent any
    stages{
        stage('Build Backend'){
            steps{
                sh 'mvn clean package -DskipTests=true'
            }
        }
        stage('Unit Tests'){
            steps{
                sh 'mvn test'
            }
        }
        stage('Sonar Analysis'){
            environment{
                scannerHome = tool 'SONAR_SCANNER'
            }
            steps{
                withSonarQubeEnv('SONAR_LOCAL'){
                  sh "${scannerHome}/bin/sonar-scanner -e \
                  -Dsonar.projectKey=DeployBack \
                  -Dsonar.host.url=http://localhost:9000 \
                  -Dsonar.login=b3607219cf86bc0f470c456e5ed0068ad4753e74 \
                  -Dsonar.java.binaries=target \
                  -Dsonar.coverage.exclusions=**/.mvm/**,**/src/test/**,**/model/**,**Application.java"
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
        stage ('Deploy Backend') {
            steps {
                deploy adapters: [tomcat8(credentialsId: 'tomcat_login', path: '', url: 'http://localhost:8001/')], contextPath: 'tasks', war: 'target/tasks-backend.war'
            }
        }
        stage ('API Test') {
            steps {
                dir('api-test'){
                    git credentialsId: 'github_login', url: 'https://github.com/ladis29/tasks-api-test'
                    sh 'mvn test'
                }
            }
        }
        stage ('Deploy Frontend') {
            steps {
                dir('frontend'){
                    git credentialsId: 'github_login', url: 'https://github.com/ladis29/tasks-frontend'
                    sh 'mvn clean package'
                    deploy adapters: [tomcat8(credentialsId: 'tomcat_login', path: '', url: 'http://localhost:8001/')], contextPath: 'tasks', war: 'target/tasks.war'
                }
            }
        }
        stage ('Functional Test') {
            steps {
                dir('functional-test'){
                    git credentialsId: 'github_login', url: 'https://github.com/ladis29/tasks-functional-tests'
                    sh 'mvn test'
                }
            }
        }
        stage('Deploy Prod'){
            steps{
                sh 'docker-compose up -d'
            }
        }
        stage ('Health Check') {
            steps {
                sleep(5)
                dir('functional-test'){
                    sh 'mvn verify -Dskip.surefile.tests'
                }
            }
        }
    }
}