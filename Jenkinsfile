pipeline {
  agent any

  parameters {
    activeChoice(
      name: 'File_Category',
      choiceType: 'PT_RADIO',
      description: 'Choose a file category (folder)',
      script: [
        $class: 'GroovyScript',
        script: [
          script: 'return ["javaapp-pipeline", "javaapp-standalone", "javaapp-tomcat"]'
        ]
      ]
    )

    reactiveChoice(
      name: 'SelectedTests',
      choiceType: 'PT_CHECKBOX',
      description: 'Select specific file types for this category',
      script: [
        $class: 'GroovyScript',
        script: [
          script: '''
            if (File_Category.equals('javaapp-pipeline')) {
              return [ 'war' ]
            } else if (File_Category.equals('javaapp-standalone')) {
              return ['jar']
            } else if (File_Category.equals('javaapp-tomcat')) {
              return [ 'war']
            } else {
              return ['No tests available']
            }
          '''
        ]
      ],
      referencedParameters: 'File_Category'
    )

    activeChoiceHtml(
      name: 'SummaryMessage',
      choiceType: 'ET_FORMATTED_HTML',
      description: 'Summary of your selections',
      script: [
        $class: 'GroovyScript',
        script: [
          script: '''
            return "<div style='padding:10px; background:#e8f5e9; border:2px solid #4caf50;'><b>File Category (Folder):</b> " + File_Category + "<br><b>Selected File Types:</b> " + SelectedTests + "</div>"
          '''
        ]
      ],
      referencedParameters: 'File_Category,SelectedTests'
    )
  }

  stages {
    stage('Checkout') {
      steps {
        git 'https://github.com/prashanth0996/jenkins.git'
      }
    }

    stage('Show Parameters') {
      steps {
        echo "Folder selected: ${params.File_Category}"
        echo "File types selected: ${params.SelectedTests}"
        echo "Summary: ${params.SummaryMessage}"
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
        withSonarQubeEnv('Sonar') {
          sh """
            cd ${params.File_Category}
            mvn clean verify sonar:sonar -Dsonar.projectKey='JOB2' -Dsonar.projectName='JOB2' 
          """
        }
      }
    }


stage ('Manual Approval' ) {

when {
        branch 'master'
          }

     options{ 
        timeout( time: 1 , unit: 'MINUTES') 
      }
	steps{
       input 'Approval for the Deploy'
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
      expression { params.File_Category == 'javaapp-tomcat' }
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
