pipeline {

    agent any

    stages{

        stage('Git Checkout'){

            steps{
                git branch: 'main', url: 'https://github.com/kennymath/demo-counter-app.git'
            }
        }
        stage ('Check-Git-Secrets') {
		    steps {
                script{
                    sh 'rm trufflesecurity/trufflehog:latest || true'
                    sh 'docker run --rm -t -v "$PWD:/pwd" trufflesecurity/trufflehog:latest github --repo https://github.com/kennymath/demo-counter-app.git --debug > trufflehog'
                    sh 'cat trufflehog'

                }
	         
	        }
        }
        stage ('Source-Composition-Analysis (SCA)') {
		    steps {
		     sh 'rm owasp-* || true'
		     sh 'wget https://raw.githubusercontent.com/devopssecure/webapp/master/owasp-dependency-check.sh'	
		     sh 'chmod +x owasp-dependency-check.sh'
		     sh 'bash owasp-dependency-check.sh'
		     sh 'cat /var/lib/jenkins/OWASP-Dependency-Check/reports/dependency-check-report.xml'
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
        stage ('Network Mapper (Port Scan)') {
		    steps {
			sh 'rm nmap* || true'
			sh 'docker run --rm -v "$(pwd)":/data uzyexe/nmap -sS -sV -oX nmap http://10.0.0.143:8080/'
			sh 'cat nmap'
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

                    def nexusRepo = readPomVersion.version.endsWith("SNAPSHOT") ? "demoapp-snapshot" : "demoapp-release"

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
        stage('Docker image Build'){

            steps{

                script {
                    sh 'docker image build -t $JOB_NAME:v1.$BUILD_ID .'
                    sh 'docker image tag $JOB_NAME:v1.$BUILD_ID kentable/$JOB_NAME:v1.$BUILD_ID'
                    sh 'docker image tag $JOB_NAME:v1.$BUILD_ID kentable/$JOB_NAME:latest'
                }
            }
        }
        stage('push image to the dockerHUb'){
            steps{
                script{
                    withCredentials([string(credentialsId: 'dockerhub1', variable: 'docker_hub_cred')]) {
                      sh 'docker login -u kentable -p ${docker_hub_cred}'
                      sh 'docker image push kentable/$JOB_NAME:v1.$BUILD_ID'
                      sh 'docker image push kentable/$JOB_NAME:latest'
                    }                  
                }
            }
        }
    }
}

    