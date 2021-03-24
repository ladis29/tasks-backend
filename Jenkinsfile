pipeline{
    agent any
    stages{
        stage('Build Backend'){
            steps{
                mvn clean package -DskipTests=true
            }
        }
    }
}