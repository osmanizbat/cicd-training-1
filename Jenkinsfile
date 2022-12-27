pipeline {
    agent any

    environment {
        PROJECT_NAME = 'spring-petclinic'
        REPO_URL = 'https://github.com/spring-projects/spring-petclinic'
    }    

    stages {

        stage('git clone') {
            steps {
                sh 'rm -rf $PROJECT_NAME'
                sh 'git clone $REPO_URL'
                sh 'pwd && ls -al'
            }
        }

        stage('Build') {
            steps {
                echo 'Building..'
                sh 'pwd && ls -al'
                sh 'cd $PROJECT_NAME'
                sh 'pwd && ls -al'
                sh 'mvn clean package'
            }
        }

        stage('Test') {
            steps {
                echo 'Testing..'
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying....'
            }
        }
    }
}