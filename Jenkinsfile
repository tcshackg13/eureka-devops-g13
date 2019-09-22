pipeline {
  agent any
  stages {
    stage('SCM  Checkout') {
      steps {
        git(url: 'https://github.com/tcshackg13/eureka-devops-g13.git', branch: 'master', credentialsId: 'github')
            }
        }
    stage("Build Code") {
            steps {
               sh 'mvn install -DskipTests'
            }
        }
        stage("Integration Testing") {
            steps {
                sh 'mvn clean verify -P integration-test'

            }
        }
    stage("SonarQube analysis") {
            steps {
                sh '/opt/sonar_scanner/sonar-scanner-4.0.0.1744-linux/bin/sonar-scanner -Dsonar.host.url=http://10.138.0.3:9000 -Dsonar.projectKey=devops-practice-lab -Dsonar.projectName=devops -Dsonar.projectVersion=1.0 -Dsonar.sources=/var/lib/jenkins/workspace/$JOB_NAME/eureka-server/src -Dsonar.java.binaries=/var/lib/jenkins/workspace/$JOB_NAME/eureka-server/target/classes/org/exampledriven/eureka/customer/server'
            }
        }
    stage("Publish Compiled Artifact to Nexus") {
        steps {
            nexusArtifactUploader artifacts: [[artifactId: 'eureka-server', classifier: '', file: 'eureka-server/target/eureka-server-0.0.1-SNAPSHOT.jar', type: 'jar']], credentialsId: 'nexus-credential', groupId: 'release', nexusUrl: '34.82.249.171:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'g13project', version: "${BUILD_NUMBER}"
        }
    }
    stage('Dockerizing Application & QA env Deployment') {
        steps {
        	sh label: 'Build Docker Image & Push to Private Repo & create container ', 
        	script: '''
        
        	echo "*******Initiate Docker Build*******"
        	echo ""
        
        	sudo docker login -u jenkin -p jenkins 10.138.0.2:8082
        	sudo docker login -u jenkin -p jenkins 10.138.0.2:8083
        
        	cd eureka-server
        
        	sudo docker build -t 10.138.0.2:8082/eureka-server-svc:latest .
        	sudo docker tag  10.138.0.2:8082/eureka-server-svc:latest  10.138.0.2:8083/eureka-server-svc:latest
        	sudo docker push 10.138.0.2:8083/eureka-server-svc:latest
        
        	CONTAINER=$(sudo docker ps -a | grep eureka | awk '{print $1}')
        	echo "Found container ID" $CONTAINER 
        	sudo docker rm -f $CONTAINER || true
        
        	echo "*******Run Container*******"
        	echo ""
        
        	sudo docker run -itd -p 8761:8761 10.138.0.2:8083/eureka-server-svc:latest'''
        	}
        }
    stage('App Deployment to Kubernetes Cluster') {
        steps {
            sh label: 'Deploy Application to Kubernetes Cluster', 
            script: '''
            
            cd eureka-server
            kubectl delete deployment eureka-micro-svc || true
            kubectl apply -f k8s-deployment.yaml
            kubectl apply -f k8s-service.yaml
            sleep 200
            kubectl get services
            
            '''
        }
    }
  }
}
