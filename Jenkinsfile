pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/rajashekar736j/jenkins.git'
            }
        }
        
        stage('Test') {
            steps {
                sh """
                    echo "The Test Started"
                    cd javaapp-pipeline/
                    mvn clean test
                """
            }
        }
    
        stage('Build') {
            steps {
                sh """
                    echo "The Build Started"
                    cd javaapp-pipeline/
                    mvn clean package
                """
            }
        }
        
        stage('Manual Approval') {
            when {
                branch 'master'
            }
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    input message: 'Approve deployment to production?',
                          ok: 'Deploy',
                          submitter: 'admin,deployer'
                }
            }
        }
        
        stage('Deploy') {
            when {
                branch 'master'
            }
            steps {
                sh """
                    echo "The Deploy Started"
                    cd javaapp-pipeline/target
                    if pgrep -f 'java -jar java-sample-21-1.0.0.jar' > /dev/null; then
                    	pkill -f 'java -jar java-sample-21-1.0.0.jar'
                        echo 'Old build was still running, killed and started new build'
                    else
                    	echo 'App was not running'
                    fi
                    BUILD_ID=dontKillMe nohup java -jar java-sample-21-1.0.0.jar > app.log 2>&1 &
                    echo "Deployed completed"
                """
            }
        }
        
        stage('Deploy to Dev') {
            when {
                branch 'dev'
            }
            steps {
                echo 'Dev branch - skipping deployment'
                echo 'Build and tests completed successfully'
            }
        }
    }
  
    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check logs and SonarQube quality gate.'
        }
    }
}
