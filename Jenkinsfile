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

        stage('SonarQube Analysis') {
            when {
                expression { params.Stages?.contains('SonarQube Analysis') }
            }
            steps {
                withSonarQubeEnv('sonar') {
                    sh '''
                        cd javaapp-pipeline && pwd
                        mvn clean verify sonar:sonar \
                        -Dsonar.projectKey=java \
                        -Dsonar.projectName=java 
                    '''
                }
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
                            "pattern": "javaapp-pipeline/target/*.*ar",
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
