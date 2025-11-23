pipeline {
    agent any

    parameters {
        activeChoice(
            name: 'Region',
            choiceType: 'PT_RADIO',
            script: [
                $class: 'GroovyScript',
                script: [
                    script: 'return ["Prod", "Dev"]'
                ]
            ]
        )

        reactiveChoice(
            name: 'Stages',
            choiceType: 'PT_CHECKBOX',
            script: [
                $class: 'GroovyScript',
                script: [
                    script: '''
                        if (Region.equals('Prod')) {
                            return ['SonarQube Analysis', 'Store Artifacts', 'Approval for deployment']
                        } else if (Region.equals('Dev')) {
                            return ['SonarQube Analysis']
                        } else {
                            return ['No tests available']
                        }
                    '''
                ]
            ],
            referencedParameters: 'Region'
        )
        
        activeChoiceHtml(
            name: 'Validation_msg',
            choiceType: 'ET_FORMATTED_HTML',
            script: [
                $class: 'GroovyScript',
                script: [
                    script: '''
                        return "<div style='padding:10px; background:#e3f2fd; border:2px solid #2196F3;'><b>Type:</b> " + Region + "<br><b>Tests:</b> " + Stages + "</div>"
                    '''
                ]
            ],
            referencedParameters: 'Region,Stages'
        )
    }
    
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
                    cd javaapp-tomcat/
                    mvn clean test
                """
            }
        }

        stage('Build') {
            steps {
                sh """
                    echo "The Build Started"
                    cd javaapp-tomcat/
                    mvn clean package
                """
            }
        }

        stage('Code Analysis') {
            when {
                expression { params.Stages?.contains('SonarQube Analysis') }
            }
            steps {
                withSonarQubeEnv('sonar') {
                    sh '''
                        cd javaapp-tomcat && pwd
                        mvn clean verify sonar:sonar \
                        -Dsonar.projectKey=java \
                        -Dsonar.projectName=java 
                    '''
                }
            }
        }

        stage('Deploy to Dev') {
            when {
                anyOf {
                    branch 'dev'
                    expression { params.Region == 'Dev' }
                }
            }
            steps {
                echo 'Dev branch - skipping deployment'
                echo 'Build and tests completed successfully'
            }
        }

        stage('Push to Artifactory') {
            when {
                allOf {
                    branch 'master'
                    expression { params.Stages?.contains('Store Artifacts') }
                }
            }
            steps {
               rtUpload(
                    serverId: 'Jfrog',
                    spec: '''{
                        "files": [{
                            "pattern": "javaapp-tomcat/target/*.*ar",
                            "target": "myorg-local/2.${BUILD_NUMBER}/"
                        }]
                    }'''
                )
            }
        }
        
        stage('Approvals') {
            when {
                allOf {
                    branch 'master'
                    expression { params.Stages?.contains('Approval for deployment') }
                }
            }
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    input message: 'Approve deployment to production?',
                          ok: 'Deploy',
                          submitter: 'admin,deployer'
                }
            }
        }
        
        stage('Deploy to Prod') {
            agent { label 'Agent1' }
            when {
                allOf {
                    branch 'master'
                    expression { params.Region == 'Prod' }
                }
            }
            steps {
                sh """
                    echo "The Deployment Started" && pwd && ls
                    cd javaapp-tomcat/target
            	    sudo scp -o StrictHostKeyChecking=no artisantek-app.war root@13.203.231.229:/opt/tomcat/latest/webapps/
            	    echo "Deployment completed"
                  """
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
