import groovy.json.*
pipeline {
    agent any
    environment {
          def scannerHome = tool 'sonar_scanner'
          def prNo = "${CHANGE_ID}"
          def repo_url = "${env.GIT_URL}"
          def repo_name = repo_url.replace("https://github.com/","").replace(".git","")
          def jenkins_credentials_ID = 'Carter-Admin'
    }
    stages {
        stage('Build') {
            steps {
                  sh './gradlew'
            }

        }
        /*
        stage('Test') {
            steps {
                  sh './gradlew test'
            }
        }
        */
        /*
        stage('Sonarqube'){
            when {
                expression {
                    "${prNo}" != "null"
                }
            }
            steps{
                withCredentials([[$class: 'StringBinding', credentialsId: "${jenkins_credentials_ID}", variable: 'GITHUB_TOKEN']]) {
                    withSonarQubeEnv('sonar') {
                        sh "${scannerHome}/bin/sonar-scanner -Dsonar.analysis.mode=preview "+
                            "-Dsonar.github.repository=${repo_name} "+
                            "-Dsonar.github.pullRequest=${prNo} "+
                            "-Dsonar.github.oauth=${GITHUB_TOKEN} "+
                            "-Dsonar.github.endpoint=https://api.github.com/"
                    }
                }
            }
        }
        */
        stage('Check Security Risk'){
            when {
                expression {
                    "${prNo}" != "null"
                }
            }
            steps{
                script{
                    def changed_lines = getChangedLines()
                    echo String.valueOf(changed_lines.additions)
                    def changed_files = getChangedFiles()
                    if(isLowsecrisk(getLowsecriskConditions(), changed_lines.additions, changed_files.filename)){
                       echo 'Low security risk, approving PR...'
                       postReview(getLowsecriskComment())
                    }
                }
            }
        }
    }
}

def parseJson(jsonText) {
    json_map = readJSON text: jsonText
    return json_map
}

def getChangedLines(){
    script{
        withCredentials([[$class: 'StringBinding', credentialsId: "${jenkins_credentials_ID}", variable: 'GITHUB_TOKEN']]) {
            def linesResponse = sh (script: "curl -s -H \"Authorization: token ${GITHUB_TOKEN}\" \"https://api.github.com/repos/${repo_name}/pulls/${prNo}\"", returnStdout: true).trim()
            echo linesResponse
            return parseJson(linesResponse)
        }
    }    
}

def getChangedFiles(){
    script{
        withCredentials([[$class: 'StringBinding', credentialsId: "${jenkins_credentials_ID}", variable: 'GITHUB_TOKEN']]) {
            def getFilesUrl = "https://api.github.com/repos/${getRepoName()}/pulls/${env.CHANGE_ID}/files"
            /*
            def githubv4call = 'query { \
                                  repository(owner:"JenkinsSonarQubeTesting", name:"fitnesse_CI_DEMO") { \
                                    pullRequests(states:[OPEN,MERGED],last:100) { \
                                      edges { \
                                        node { \
                                          reviews(first: 100, states:[COMMENTED,APPROVED]) { \
                                            nodes { \
                                              createdAt \
                                              bodyText \
                                              author { \
                                                login \
                                              } \
                                            } \
                                          } \
                                          url \
                                        } \
                                      } \
                                      pageInfo { \
                                        startCursor hasPreviousPage \
                                      } \
                                    } \
                                  } \
                                }'
            def response = sh (script: "curl -X \"POST\" \"https://api.github.com/graphql\" -H \"Authorization: Bearer ${GITHUB_TOKEN}\" -d \"\"query\": \"${githubv4call}\"\"", returnStdout: true).trim()
            */
            def response = sh (script: "curl -s -H \"Authorization: token ${GITHUB_TOKEN}\" \"https://api.github.com/repos/${repo_name}/pulls/${prNo}/files\"", returnStdout: true).trim()
            //def response = httpRequest authentication: 'Carter-DM-UserPass', httpMode: 'GET', url: "https://api.github.com/repos/${getRepoName()}/pulls/${env.CHANGE_ID}/files"

            echo response
            return parseJson(response)
        }
    }
}
                                             
def getRepoName(){
    def repo_url = "${env.GIT_URL}"
    def repo_name = repo_url.replace("https://github.com/","").replace(".git","")
    return repo_name
}
                                             
def postReview(message){
    script{
        withCredentials([[$class: 'StringBinding', credentialsId: "${jenkins_credentials_ID}", variable: 'GITHUB_TOKEN']]) {
            sh "curl -s -H \"Authorization: token ${GITHUB_TOKEN}\" -X POST -d '{\"body\": \"${message}\", \"event\": \"APPROVE\"}' \"https://api.github.com/repos/${repo_name}/pulls/${prNo}/reviews\""
        }
    }
}

def getLowsecriskConditions(){
    return ['test', '.xml', 'jenkins']
}

def getLowsecriskComment(){
    return '@lowsecrisk (Automated review)'
}

def isLowsecrisk(conditions, changed_lines, edited_files){
    for(file in edited_files){
        if (!(conditions.any{file.toLowerCase().contains(it)}) && (changed_lines > 1000)){
           return false
       }
    }
    return true
}
