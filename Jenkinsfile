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
                return ['jar', 'war', 'ear']
            } else if (File_Category.equals('javaapp-standalone')) {
                return ['jar', 'war', 'ear']
            } else if (File_Category.equals('javaapp-tomcat')) {
                return ['jar', 'war', 'ear']
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

                        steps {
                withSonarQubeEnv('Sonar') {
                    sh '''
                        cd ${folder} && pwd
                        mvn clean verify sonar:sonar \
                        -Dsonar.projectKey=java \
                        -Dsonar.projectName=java 
                    '''
                }
            }
        }

  stage ('Manual Approval' ) {

when {
        branch 'main'
          }

     options{ 
        timeout( time: 1 , unit: 'MINUTES') 
      }
	steps{
       input 'Approval for the Deploy'
	}
	}


stage ('Deploy') {
 when {
        branch 'main'
          }
	steps {
        sh """
        echo "The Deploy Started"
        cd ${folder}/target
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
