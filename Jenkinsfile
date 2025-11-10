pipeline {
 agent any
 //triggers { pollSCM ('H/5 * * * *) }

stages{
	stage('Checkout') {
	 steps {
          git "https://github.com/prashanth0996/jenkins.git"
         }
        }
     
      stage('Test') {
       steps {
       sh """
          echo "The Test Started"
          cd /var/lib/jenkins/workspace/pipeline_tomcat/javaapp-tomcat/
         mvn clean test
       """
      }
     }

	stage('Build') {
       steps {
       sh """
          echo "The Build Started"
        cd /var/lib/jenkins/workspace/pipeline_tomcat/javaapp-tomcat/
         mvn clean package
       """
      }
     }
        

	stage ('Deploy') {
	steps {
        sh """
        echo "The Deploy Started"
        cd /var/lib/jenkins/workspace/pipeline_tomcat/javaapp-tomcat/target
	sudo cp *.war /opt/apache-tomcat-10.1.48/webapps
	echo "Deployed completed"
          """
	}
	}	

   }
}
