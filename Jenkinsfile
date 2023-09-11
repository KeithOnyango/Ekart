pipeline {
    agent any 
    
    tools{
        jdk 'jdk17'
        maven 'maven3.9'
    }
    
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    
    stages{
        
        // stage('Git Checkout') {
        //     steps {
        //         git branch: 'main', url: 'https://github.com/KeithOnyango/Ekart.git'
        //     }
        // }
        
        stage("Code Compile"){
            steps{
                sh "mvn clean compile"
            }
        }
        
         stage("Unit Test"){
            steps{
                sh "mvn clean compile -DskipTests=true"
            }
        }
        
        stage("Sonarqube  Analysis"){
            steps{
                  withSonarQubeEnv('sonar') {
                      
                  sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=EKART \
                   -Dsonar.java.binaries=. \
                   -Dsonar.projectKey=EKART '''             
                   }
            }
        }

        stage("Quality Gate") {
            steps {
            waitForQualityGate abortPipeline: true
            }
        }
        
        
        stage("OWASP Dependency Check"){
            steps{
                dependencyCheck additionalArguments: '--scan ./ ', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        
         stage("Artisan Build"){
            steps{
                sh "mvn clean package -DskipTests=true"
            }
        }
        
        stage("Build & Tag Docker Image"){
            steps{
                script{
                  withDockerRegistry(credentialsId: '17105be0-af72-4ebf-b41f-d8c672b0a0fc', toolName: 'docker') {
                        sh "docker build -t shopping-cart -f docker/Dockerfile ."
                        sh "docker tag shopping-cart xpertdocker254/shopping-cart:latest "
                    }
                }
            }
        }
        
        stage("Push Docker Image"){
            steps{
                script{
                  withDockerRegistry(credentialsId: '17105be0-af72-4ebf-b41f-d8c672b0a0fc', toolName: 'docker') {
                        sh "docker push xpertdocker254/shopping-cart:latest "
                    }
                }
            }
        }
        
        
        stage("Trivy Scan"){
            steps{
                sh " trivy image --scanners vuln xpertdocker254/shopping-cart:latest"
            }
        }
        
        stage("Deploy Application"){
            steps{
                sh "docker run -d --name ekart1 -p 8070:8070 xpertdocker254/shopping-cart:latest"
            }
        }
    }
}
