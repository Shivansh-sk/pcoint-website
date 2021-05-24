def COLOR_MAP = [
    'SUCCESS': 'good', 
    'FAILURE': 'danger',
    ]
def getBuildUser(){
    return currentBuild.rawBuild.getCause(Cause.UserIdCause).getUserId()
}


pipeline {
    // Set up local variables for your pipeline
    environment {
        // test variable: 0=success, 1=fail; must be string
        doError = '0'
        BUILD_USER = ''
    }

    agent any

    stages {
        stage('Error') {
            // when doError is equal to 1, return an error
            when {
                expression { doError == '1' }
            }
            steps {
                echo "Failure :("
                error "Test failed on purpose, doError == str(1)"
            }
        }
        stage('Success') {
            // when doError is equal to 0, just print a simple message
            when {
                expression { doError == '0' }
            }
            steps {
                
                sh( returnStdout: true, script: '''
                sudo rm -rf /pcoint/
                sudo git clone https://github.com/Shivansh-sk/pcoint-website.git /pcoint

                if sudo docker ps | grep pcoint
                then
                sudo docker rm -f pcoint
                sudo docker run -d -p 8081:80 -v /pcoint:/usr/local/apache2/htdocs --name pcoint httpd
                else
                sudo docker run -d -p 8081:80 -v /pcoint:/usr/local/apache2/htdocs --name pcoint httpd
                fi
                '''.stripIndent())
                echo "Success :)"
            }
        }
    }

    // Post-build actions
    post {
        always {
            script {
                BUILD_USER = getBuildUser()
            }
            echo "Sending output to slack"
            slackSend channel: '#jenkins-slack',
                color: COLOR_MAP[currentBuild.currentResult],
                message: "${currentBuild.currentResult}\n Job ' ${env.JOB_NAME} ' build ${env.BUILD_NUMBER} \nUSER: ${BUILD_USER}"
            
        }
    }
}
