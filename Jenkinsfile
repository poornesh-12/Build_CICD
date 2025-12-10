pipeline {
    agent any

    triggers {
        githubPush()   // or gitlab(...) or GenericTrigger(...)
    }

    environment {
        SONARQUBE_ENV = 'sonarqube-server'
        TOMCAT_DEV = '/opt/tomcat_dev/webapps'
        TOMCAT_QA  = '/opt/tomcat_qa/webapps'
        TOMCAT_PROD = '/opt/tomcat_prod/webapps'
        BACKUP_DIR = "/opt/war_backup"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Compile') {
            steps {
                sh 'mvn clean compile'
            }
        }

        stage('Unit Test') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }

        stage('SonarQube') {
            steps {
                withSonarQubeEnv("${SONARQUBE_ENV}") {
                    sh 'mvn sonar:sonar'
                }
            }
        }

        stage('Build WAR') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Backup WAR') {
            steps {
                sh "mkdir -p ${BACKUP_DIR}"
                sh "cp target/*.war ${BACKUP_DIR}/app-\$(date +%F-%H%M%S).war"
            }
        }

        stage('Deploy to Dev') {
            steps {
                echo "Deploying to Dev..."
                sh "rm -rf ${TOMCAT_DEV}/yourapp"
                sh "cp target/*.war ${TOMCAT_DEV}/yourapp.war"
            }
        }

        stage('Deploy to QA') {
            when {
                expression { return input message: "Deploy to QA?" }
            }
            steps {
                echo "Deploying to QA..."
                sh "rm -rf ${TOMCAT_QA}/yourapp"
                sh "cp target/*.war ${TOMCAT_QA}/yourapp.war"
            }
        }

        stage('Deploy to Prod') {
            when {
                expression { return input message: "Deploy to PROD?" }
            }
            steps {
                echo "Deploying to Prod..."
                sh "rm -rf ${TOMCAT_PROD}/yourapp"
                sh "cp target/*.war ${TOMCAT_PROD}/yourapp.war"
            }
        }
    }

    post {
        success {
            echo "PIPELINE COMPLETED SUCCESSFULLY!"
        }
        failure {
            echo "PIPELINE FAILED!"
        }
    }
}
