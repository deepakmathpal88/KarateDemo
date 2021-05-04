node {

        stage('Build') {
         git 'https://github.com/qamatters/KarateDemo.git'

          try {

                     def mvnHome = tool name: 'maven', type: 'maven'
                      tool name: 'JDK 1.8', type: 'jdk'
                      sh '''
                                          echo "PATH = ${PATH}"
                                          echo "M2_HOME = ${M2_HOME}"
                         '''

            /* ... existing build steps ... */
            sh "${mvnHome}/bin/mvn clean test"
            sh "${mvnHome}/bin/mvn clean test -DargLine=\'-Dkarate.env=e2e\' -Dkarate.options=\"--tags @Smoke\" -Dtest=CucumberReport -DfailIfNoTests=false"

          } catch (e) {
            // If there was an exception thrown, the build failed
            currentBuild.result = "FAILED"
            notifyBuild(currentBuild.result)
            throw e
          } finally {
            cucumber '**/target/karate-reports/*.json'
            notifyBuild(currentBuild.result)
           fileExists 'Summary.txt'
           def file = readFile 'Summary.txt'
           sh '''
                 set +x

             def totalPass = awk 'NR==1{print $1}' /file
             echo "TotalPass = ${totalPass}"

           '''

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
