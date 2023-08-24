pipeline{
    agent any
        stages{
            stage("checkout"){
                steps{
                    checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/sravanreddie/registration-app.git']])
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
                    sh "mvn clean verify sonar:sonar -Dsonar.projectKey=monitoring-app -Dsonar.host.url=http://ec2-54-157-170-174.compute-1.amazonaws.com:9000 -Dsonar.login=sqp_d992934984c5a07b909b85c3dfb8eaaf164e908a"
                }
            }
            stage("nexus"){
                steps{
                    nexusArtifactUploader artifacts: [[artifactId: 'maven-project', classifier: '', file: '/var/lib/jenkins/workspace/project/webapp/target/webapp.war', type: 'war']], credentialsId: 'nexus', groupId: 'com.example.maven-project', nexusUrl: '54.167.178.18:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'maven-snapshots', version: '1.0-SNAPSHOT'
                }
            }
            stage("docker image build &push"){
                steps{
                    script{
                        withDockerRegistry(credentialsId: 'dockerhub') {
                            sh ' docker build -t sravan2605/registration:lst .'
                            sh ' docker push sravan2605/registration:lst'
                        }
                    }
                }
            }
            stage("deploy to k8s"){
                steps{
                    script{
                        withCredentials([string(credentialsId: 'docker-secrettext', variable: '')]){
                            sh 'kubectl apply -f regapp-deploy.yml'
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
