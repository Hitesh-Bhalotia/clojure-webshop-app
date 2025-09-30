pipeline {
    agent any
    
    tools{
        jdk "jdk"
    }
     environment{
        SCANNER_HOME=tool "sonar-scanner"
    }

    stages {
        stage('Git Checkout') {
            steps {
                git 'https://github.com/Hitesh-Bhalotia/clojure-webshop-app.git'
            }
        }
        
        stage('CLJ-HOLMES-SCAN') {
            steps {
                sh "clj-holmes fetch-rules"
                sh "clj-holmes scan -p ./"
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=WEBSHOP \
                    -Dsonar.java.binaries=. \
                    -Dsonar.projectKey=WEBSHOP '''
                }   
            }
        }
        
        stage('OWASP Dependency Check') {
            steps {
                 dependencyCheck additionalArguments: ' --scan ./ ', odcInstallation: 'DC'
                    dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        
        stage('Build & Deploy') {
            steps {
                 sh "docker-compose up -d"
            }
        }
    }
}
