pipeline{
    agent any
        stages{
            stage("checkout"){
                steps{
                    checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/sravanreddie/Registration-app.git']])
                }
            }
            stage("build"){
                steps{
                    sh 'mvn package'
                }
            }
            stage("Test"){
                steps{
                    sh 'mvn test'
                }
            }
            stage("quality testing sonarqube"){
                steps{
                    sh "mvn clean verify sonar:sonar -Dsonar.projectKey=registration-app -Dsonar.host.url=http://ec2-52-87-20-228.compute-1.amazonaws.com:9000 -Dsonar.login=sqp_c65d76e95c643568639a09977c61640ec437b336"
                }
            }
            stage("nexus"){
                steps{
                    nexusArtifactUploader artifacts: [[artifactId: 'maven-project', classifier: '', file: '/var/lib/jenkins/workspace/project/webapp/target/webapp.war', type: 'war']], credentialsId: 'Nexus_cred', groupId: 'com.example.maven-project', nexusUrl: '54.90.15.213:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'maven-snapshots', version: '1.0-SNAPSHOT'
                }
            }
            stage("docker image build &push"){
                steps{
                    script{
                        withDockerRegistry(credentialsId: 'dockerhub') {
                            sh ' docker build -t sravan2605/registrationapp:lst .'
                            sh ' docker push sravan2605/registrationapp:lst'
                        }
                    }
                }
            }
            stage("deploy to k8s"){
                steps{
                    script{
                        withCredentials([string(credentialsId: 'docker-secrettext', variable: '')]){
                            sh 'kubectl apply -f deployment.yaml'
                            sh 'kubectl get pods'
                        }
                    
                    }
                }
            }
	    }
	    post {
		always {
			mail bcc: '', body: "<br>Project: ${env.JOB_NAME} <br>Build Number: ${env.BUILD_NUMBER} <br> URL de build: ${env.BUILD_URL}", cc: '', charset: 'UTF-8', from: '', mimeType: 'text/html', replyTo: '', subject: "${currentBuild.result} CI: Project name -> ${env.JOB_NAME}", to: "sravankumarreddysingam@gmail.com";  
		 }
	   }
}
