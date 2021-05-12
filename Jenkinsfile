node {
        stage('Build') {
         git 'https://github.com/qamatters/KarateDemo.git'
          try {

           parameters {
                           choice(name: 'Tags', choices: ['@Smoke', '@Regression', '@@Test123'], description: 'Run with different Tags')
                           choice(name: 'Environment', choices: ['@Stage', 'e2e'], description: 'Run with different Environments')
                       }

          if(isUnix()) {
                   def mvnHome = tool name: 'maven', type: 'maven'
                      tool name: 'JDK 1.8', type: 'jdk'
                      sh '''
                                          echo "PATH = ${PATH}"
                                          echo "M2_HOME = ${M2_HOME}"
                         '''
            /* ... existing build steps ... */
            println "-------------------------------------------------------------------------------------"
            sh 'env'
            println "-------------------------------------------------------------------------------------"
            sh "${mvnHome}/bin/mvn clean test -DargLine=\'-Dkarate.env=${params.Tags}\' -Dkarate.options=\"--tags ${params.Environment}\" -Dtest=CucumberReport -DfailIfNoTests=false"
            } else {
            println "-------------------------------------------------------------------------------------"
            bat 'set'
            println "-------------------------------------------------------------------------------------"
             bat "mvn clean test -DargLine=\'-Dkarate.env=e2e\' -Dkarate.options=\"--tags @Smoke\" -Dtest=CucumberReport -DfailIfNoTests=false"
            }

          } catch (e) {
            // If there was an exception thrown, the build failed
            currentBuild.result = "FAILED"
            notifyBuild(currentBuild.result)
            throw e
          } finally {
            cucumber '**/target/karate-reports/*.json'
            notifyBuild(currentBuild.result)
          }
        }
       }
         def notifyBuild(String buildStatus = 'STARTED') {
           // build status of null means successful
           buildStatus = buildStatus ?: 'SUCCESS'
           // Default values
           def colorName = 'RED'
           def colorCode = '#FF0000'
           def subject = "${buildStatus}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'"
           def summary = "${subject} (${env.BUILD_URL})"
           def details = """<p>STARTED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p> <br> <p>Check console output at:<a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>;</p>"""


           // Override default values based on build status
           if (buildStatus == 'STARTED') {
             color = 'YELLOW'
             colorCode = '#FFFF00'
           } else if (buildStatus == 'SUCCESS') {
             color = 'GREEN'
             colorCode = '#00FF00'
           } else {
             color = 'RED'
             colorCode = '#FF0000'
           }
           emailext (
               subject: subject,
               body: details,
               to: 'testqamatters@gmail.com',
                mimeType: 'text/html'
             )
         }
