pipeline {
 agent any

	triggers{
		githubPush()
	}

stages{
	stage('Checkout') {
	 steps {
          git "https://github.com/rajashekar736j/jenkins"
         }
        }
     
      stage('Test') {
       steps {
       sh """
          echo "The Test Started"
          cd /var/lib/jenkins/workspace/tomcat-pipeline/javaapp-tomcat/
         mvn clean test
       """
      }
     }

	stage('Build') {
       steps {
       sh """
          echo "The Build Started"
          cd /var/lib/jenkins/workspace/tomcat-pipeline/javaapp-tomcat/
         mvn clean package
       """
      }
     }
        

	stage ('Deploy') {
	steps {
        sh """
        echo "The Deploy Started"
        cd /var/lib/jenkins/workspace/tomcat-pipeline/javaapp-tomcat/target
	    sudo cp *.war /opt/tomcat/webapps
	    echo "Deployed completed"
          """
	}
	}	

   }
}
