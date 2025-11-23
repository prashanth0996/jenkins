pipeline {
  agent any

  parameters {
    choice(
      name: 'File_Category',
      choices: ['javaapp-pipeline', 'javaapp-standalone', 'javaapp-tomcat'],
      description: 'Choose a file category (folder)'
    )
    choice(
      name: 'SelectedTests',
      choices: ['jar', 'war'],
      description: 'Select file types for this category'
    )
    string(
      name: 'SummaryMessage',
      defaultValue: '',
      description: 'Summary of your selections'
    )
  }

  stages {
    stage('Checkout') {
      steps {
        git 'https://github.com/prashanth0996/jenkins.git'
      }
    }

    stage('Test') {
      steps {
        sh """
          echo "The test is started"
          cd ${params.File_Category}
          mvn clean test
        """
      }
    }

    stage('Build') {
      steps {
        sh """
          echo "The build is started"
          cd ${params.File_Category}
          mvn clean package -DskipTests
        """
      }
    }

    stage('Code Analysis') {
      when {
        allOf {
          branch 'master'
          expression { params.File_Category == 'javaapp-pipeline' }
        }
      }
      steps {
        withSonarQubeEnv('sonar') {
          sh '''
            cd ${params.File_Category} && pwd
            mvn clean verify sonar:sonar \
              -Dsonar.projectKey=java \
              -Dsonar.projectName=java
          '''
        }
      }
    }

    stage('Manual Approval') {
      when {
        branch 'master'
      }
      options {
        timeout(time: 1, unit: 'MINUTES')
      }
      steps {
        input 'Approval for the Deploy'
      }
    }

    stage('Upload to Artifactory') {
      when {
        branch 'master'
      }
      steps {
        rtUpload(
          serverId: 'Jfrog',
          spec: '''
            {
              "files": [{
                "pattern": "${params.File_Category}/target/*.*ar",
                "target": "Maven/2.${env.BUILD_NUMBER}/"
              }]
            }
          '''
        )
      }
    }

    stage('Deploy Standalone') {
      when {
        allOf {
          branch 'master'
          expression { params.File_Category == 'javaapp-standalone' }
        }
      }
      steps {
        sh """
          echo "The Deploy is started"
          pwd
          cd ${params.File_Category}/target
          JENKINS_NODE_COOKIE=dontKillMe nohup java -jar java-sample-21-1.0.0.jar > out.log 2>&1 &
          sudo netstat -tulnp | grep 5000 || echo "Deployment check completed"
        """
      }
    }

    stage('Deploy Tomcat') {
      when {
        allOf {
          branch 'master'
          expression { params.File_Category == 'javaapp-tomcat' || params.File_Category == 'javaapp-pipeline' }
        }
      }
      steps {
        sh """
          echo "The Deploy Started"
          cd ${params.File_Category}/target
          sudo cp *.*ar /opt/tomcat/webapps/
          echo "Deployed completed"
        """
      }
    }
  }

  post {
    always {
      echo "Clean the work space after post build"
      cleanWs()
    }
  }
}
