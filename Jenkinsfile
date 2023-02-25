pipeline {

    agent any

    stages{

        stage('Git Checkout'){

            steps{
                git branch: 'main', url: 'https://github.com/kennymath/demo-counter-app.git'
            }
        }
        stage('UNIT Testing'){

            steps{
                sh 'mvn test'
            }
        }
        stage('Integration testing'){

            steps{
                sh 'mvn verify -DskipUnitTests'
            }
        }
        stage('Maven Build'){

            steps{
                sh 'mvn clean install'
            }
        }
        stage('Static code analysis'){
            steps {
                script{
                    withSonarQubeEnv(credentialsId: 'SONAR'){
                        sh 'mvn clean package sonar:sonar'
                    }
                }           

            }
        }
        stage('upload war file to nexus'){

            steps{

                script{
                    def readPomVersion = readMavenPom file: 'pom.xml'

                    def nexusRepo = readMavenPom.version.endsWith("SNAPSHOT") ? "demoapp-snapshot" : "demoapp-release"
                    
                    nexusArtifactUploader artifacts:
                    [
                        [artifactId: 'springboot', 
                        classifier: '', file: 'target/Uber.jar', 
                        type: 'jar'
                        ]
                    ], 
                    credentialsId: 'Nexus-id', 
                    groupId: 'com.example',
                    nexusUrl: '10.0.0.99:8081', 
                    nexusVersion: 'nexus3', 
                    protocol: 'http', 
                    repository: nexusRepo, 
                    version: "${readPomVersion.version}"
                }
            }
        }
     //  stage('Quality Gate status'){

     //       steps{

     //           script{
     //              waitForQualityGate abortPipeline: false, credentialsId: 'SONAR'
     //           }
     //       }
     //   }
    }
}