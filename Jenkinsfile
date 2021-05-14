node {

 properties(
    [
        parameters(
            [
             string(defaultValue: '@Smoke', name: 'Tags'),
             string(defaultValue: 'e2e', name: 'Environment'),
             choice(name: 'Environment', choices:['e2e', 'stage', 'dev'] ,defaultValue: 'e2e')
             ]
            )
    ]
  )
        stage('Build') {
         git 'https://github.com/qamatters/KarateDemo.git'
          try {

          if(isUnix()) {
                   def mvnHome = tool name: 'maven', type: 'maven'
                      tool name: 'JDK 1.8', type: 'jdk'
                      sh '''
                                          echo "PATH = ${PATH}"
                                          echo "M2_HOME = ${M2_HOME}"
                         '''
            /* ... existing build steps ... */

            println "-------------------------------------------------------------------------------------"
            println "Tag is ${params.Tags}"
            println "Environment is ${params.Environment}"

            println "----------------------------Print Environment variables in Linux machine---------------------------------------------------------"
            sh 'env'
            println "-------------------------------------------------------------------------------------"
            sh "${mvnHome}/bin/mvn clean test -DargLine=\'-Dkarate.env=${params.Environment}\' -Dkarate.options=\"--tags ${params.Tags}\" -Dtest=CucumberReport -DfailIfNoTests=false"

            } else {

            println "------------------------------Print Environment variables in Windows machine-------------------------------------------------------"
            bat 'set'
            println "-------------------------------------------------------------------------------------"
            bat "mvn clean test -DargLine=\'-Dkarate.env=${params.Environment}\' -Dkarate.options=\"--tags ${params.Tags}\" -Dtest=CucumberReport -DfailIfNoTests=false"
          }

        } catch (e) {
            // If there was an exception thrown, the build failed
            currentBuild.result = "FAILED"
            notifyBuild(currentBuild.result)
            throw e
          } finally {
          def fileexist = fileExists 'Summary.txt'
          if(fileexist)
          {
           String summaryFileContent =  readFile 'Summary.txt'
           println "${summaryFileContent}"
           Total_Scenarios   = summaryFileContent[0]
           print "Total_Scenarios: ${Total_Scenarios}"
           Total_Pass_Scenarios = summaryFileContent[1]
           print "Total_Pass_Scenarios: ${Total_Pass_Scenarios}"
           Total_Fail_Scenarios = summaryFileContent[2]
           print "Total_Fail_Scenarios= ${Total_Fail_Scenarios}"
          }
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
           def details =

           """
           <p>
           BuildNmae:${env.JOB_NAME}
           <br>
           BUILD_URL:${env.BUILD_URL}
           <br>
           Build_Status:${buildStatus}
           Check console output at:<a href='${env.BUILD_URL}'>${env.JOB_NAME}[${env.BUILD_NUMBER}]</a>
           </p>
           """

           def SummaryTable =
           """
           <!DOCTYPE html>
           <html>
           <head>
           <style>
           table, th, td {
             border: 1px solid black;
           }
           td,th {
                      text-align: center;
            }
           </style>
           </head>
           <body>
           <h2>$details}</h2>
           <br>

           <table>
             <tr>
               <th>Total_Scenarios</th>
               <th>Total_Pass_Scenarios</th>
               <th>Total_Fail_Scenarios</th>
             </tr>
             <tr>
               <td>${Total_Scenarios}</td>
               <td>${Total_Pass_Scenarios}</td>
               <td>${Total_Fail_Scenarios}</td>
             </tr>
           </table>
           </body>
           </html>
           """
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
               body: SummaryTable,
               to: 'testqamatters@gmail.com',
                mimeType: 'text/html'
             )
         }
