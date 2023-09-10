pipeline {
    agent any
    tools{
        jdk  'jdk11'
        maven  'maven3.6'
    }
    
    environment{
        SCANNER_HOME= tool 'sonar-scanner'
    }
    
    stages {
        // stage('Git Checkout') {
        //     steps {
        //         git branch: 'main', changelog: false, credentialsId: '15fb69c3-3460-4d51-bd07-2b0545fa5151', poll: false, url: 'https://github.com/jaiswaladi246/Shopping-Cart.git'
        //     }
        // }
        
        stage('Code Compile') {
            steps {
                sh "mvn clean compile -DskipTests=true"
            }
        }
        
        stage('Owasp Dependency Check') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ ', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        
        stage('Sonarqube Analysis') {
            steps {
                withSonarQubeEnv('sonar'){
                   sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=EKART \
                   -Dsonar.java.binaries=. \
                   -Dsonar.projectKey=EKART '''
               }
            }
        }

        stage("Quality Gate") {
          steps {
              waitForQualityGate abortPipeline: true
            //   timeout(time: 1, unit: 'HOURS') {
                
            //   }
            }
        }
        
        stage('Code Build') {
            steps {
                sh "mvn clean package -DskipTests=true"
            }
        }
        
        stage('Docker Build & Push') {
            steps {
                script{
                    withDockerRegistry(credentialsId: '17105be0-af72-4ebf-b41f-d8c672b0a0fc', toolName: 'docker') {
                        
                        sh "docker build -t shopping-cart -f docker/Dockerfile ."
                        sh "docker tag  shopping-cart xpertdocker254/shopping-cart:latest"
                        sh "docker push xpertdocker254/shopping-cart:latest"
                    }
                }
            }
        }

        stage("Trivy Scan"){
            steps{
                sh " trivy image --scanners vuln xpertdocker254/shopping-cart:latest"
            }
        }
        
        stage("Deploy Product"){
            steps{
                sh "docker run -d --name ekart -p 8070:8070 xxpertdocker254/shopping-cart:latest"
            }
        }


        
        
    }
}
